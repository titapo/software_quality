## Control flow flexibility

Programs written in imperative style operates with statements which are called in a sequence. 

There are just a few rules applied to control flow.

1. A function can be called after its parameters have been instantiated.
    Before C++17 the evaluation order of parameters is not specified.
2. A member function can be called after its object has been instantiated. (Considering `this` as the 0. parameter it is
   the same as the previous)
3. The destructor is called when the object goes out of scope.
    This is the base of the RAII technique, used by smart pointers as well.

And there are some other notes:

`if`/`switch` controls are fragile:
Condition can be accidentally negated or branches can be interchanged. The only thing that can help you to notice the
bug is the properly written test for that branch.

Using error code requires `if`:
See the previous point.

Exceptions can obfuscate:
Since exceptions are not visible at interface level in the function signature, you have to make investigation whether the
current function, other functions called from this (at any depth), involving virtual functions and callbacks does not
throw exception. The only thing that can help is the `noexcept` specifier.


We are talking to flexibility and the problem here also is that too much flexibility can be harmful. // TODO fix this
sentence

Let's see the common examples. Consider the following example from an imaginary library

```cpp
void init();
```

This can be a free function or a method of a class, it does not really matter. The point here this interface provides great
flexibility for us to call it when we want. It is very nice, but we need to ask whether is it intentional or not?

Because with this power of flexibility, we can
- forget to call,
- call more than once,
- call concurrently from different threads.

Will the program work properly under these circumstances? Are all of these cases handled by the library implementor? Or
at least "Don't do this!" warnings has been added to the documentation even it is easy to ignore?

And we are not ready yet, because further questions emerge:

- What happens if initialization fails? It throws an exception or goes to an invalid state?
- What happens if other function is called which requires initialization? All of these functions should check
  `is_initialized()` defensively? Or it should be supposed to be in an initialized state even if it possibly causes crash?

Some other smells similar to this kind of `init()`:

- `void connect()`: same as `init()`
- `bool is_valid()`: if validity has to be checked to prevent some 'sub-interface' to be used reveals the smell that
  call flow provided more flexibility than the intention.
- `void open(Read|Write)`: typically file handlers suffers from this antipattern. One side it has the same illness as
  `init()`, but there is more here. Depending the open mode, some functions should not be called. For example, if a file
opened in read-only mode, `write()` is not supposed to be called. Similarly as `if`-s we need exhaustive test to ensure it is used properly.

We should examine pointers at that point. They are very flexible because they can be dereferenced even for `nullptr`.
This is not just redundancy, but a critical danger. So we need an `if` to prevent the program from crashing. So if we
do not want to encounter this problem, we can use reference instead of pointer, if reassign is not a requirement.

We can go further to ask a more radical question: Every setters are affected from this danger? So mutability is
problematic in general?

Functional programming responds clear 'yes' to this. Here I do not go this far, but it is worth to think about it a bit.


There is a silly but very concrete example:

```cpp
Client client(some_data);
client.close();
client.sendMessage();
client.connect(addr)
```

Did you notice the mistake? You are right, methods are called in a wrong order. Compiler does not complain about this,
we need to run the program to admit that it does not working, like script languages. It is the worst combination to have
a compiler, but relying runtime only... One can say, we have to reorder the *statements* to fix the bug, but please not
that this is just an accidental fix.

So it's time to take out the control flow constraining rules. Based on them `connect()`, `sendMessage()`, `close()` can
be called only `Client` instantiation, but this is the only rule which is used in the example above:

```
Client => connect
       => sendMessage
       => close
```

But we need more constraint, like:

```
Client => connect => sendMessage => close
```

It can be achieved by introducing a new type which represents the connection:

```cpp
Client::connect(Address) -> Connection
```

And after we moved the `sendMessage()` into this `Connection`, the only thing remaining to place the `close`
functionality into `~Connection()`.

The possible call-graph now has the following shape:

```
Client => connect -> Connection                (=> close)
                                => sendMessage (=> close)
```

And the actual code is below.

```cpp
Client client(some_data);
Connection connection = client.connect(addr);
connection.sendMessage(“Hello”);
// connection is closed by its destructor
```

None of the lines can be interchanged with an other one. And at this point we do not need for the silly corner-case unit
tests, like `close_without_connected`. And the nice thing here that we do not have to hunt these cases, the compiler
will point to them, and we have to decide whether is it still a relevant test or not (hopefully a good name was
given to the test previously)


### Type transformations

To understand it better we can see this solution from an other point of view. For that we have to go back to the
original idea of the function: from an input it produces an output.

```
B f(A);
```

Constrain control flow (now function call ordering) is possible by defining functions that transform types one from other, so if we want `g()` to be callable only after `f()` two functions needed:

```
B f(A);
C g(B);
```

Supposing that these types are producible only by these functions, our constraints are ready:

```
A -> f() -> B -> g() -> C
```

/// TODO should *composablility* take place here?

### Back to divide

Previously when we discussed value-flexiblity we had the following solution:

```cpp
std::optional<double> divide(double a, double b)
{
  if (b == 0.0)
    return std::nullopt;

  return a / b;
}
```

We can say that `b = 0` is an undesired input value here, which should be excluded, so we need a function that makes the
divide from `double` and a `not zero double`, and consequently the output type can be `double` as well.

```cpp
double divide(double a, NotZero b)
{
  return a / b.value;
}
```

We can remove the `if` which is good, only two questions arise: How to implement `NotZero`? And why we need the `.value`
accessor?


```cpp
struct NotZero // (1)
{
  public:
    static optional<NotZero> create(double v) // (2)
    {
      if (v == 0.0) return {};
      return NotZero{v};
    }

    const double value; // (3)

  private: // (4)
    NotZero(double v) : value(v) {}
};
```
ŕ


1. The underlying type can be accessible as a template parameter, it does not matter in the current implementation.
2. a `static` function is available to instantiate an object. This is the only way to establish this, because it
   accesses the private constructor that is not available from outside the class. It is required because the
   construction can fail. (In pre-C++17 version throwing an exception can be an option, but otherwise it is not
   encouraged). The main point to not leave any possiblity to instantiate an invalid object. The `create` handles the logic
   that required to instantiate the object, while the real constructor remains stupidly simple. Instead of the static
   `create` you can use a friend function as a *type constructor* [TODO](haskell ref?).
3. `const` is mandatory here, if it was mutable the, `NotZero` class could not be usable. However it cannot be mutated,
   so public access is not a problem here.
4. private constructor is the another mandatory requrement here in this particular case. If the construction cannot be
   fail (for example. escaping against sql injection), it can be public, because it always producible, but we want to
   distinguish it from a raw object


/// TODO url example or similar: ip address is not a valid URL: protocol, port, path, query? needed. We should not
represent the first one as an Url, and pass it to `Url f(Url)`. It is easy to forget. The previous Url is more (value-)flexible, than it should be.
/// TODO we could reach this solution from value-flexiblity aspect as well by restricting the possible input values as well
