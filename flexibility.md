# Software Flexibility


## Flexibility in general

Let's see two function signatures below:

```cpp
void setAutopilot(bool);
```

```cpp
void setAutopilot(const string&);
```

Which one is better? Actually it depends on the domain requirements or, so-called specification. Without that we can
only say the latter is more flexible than the first one, because any strings can be accepted there, while the first can
have only two possible values.

So there is a flexibility expressed in the code and there is a flexibility required by the specification. In this case
the possible inputs can be represented as a set. Similarly the specification also declares a set of values which must be
accepted as an input.

Now we have two sets, so math help as to define their relationship:

  1. required flexibility = actual flexibility: this is the optimal implementation
  2. required > actual: this means that possible states represented in the code is insufficient to properly handle the
problem
  3. required < actual: the difference is the set of redundant/irrelevant/illegal states (which one? see later). And
these states must be handled somehow
  4. intersected: it can be resolved as 2 and 3, becase there are values that should be covered by code and there are
other values that actually covered but they shouldn't be. This shows that not only the cardinality(?) of the sets
matters 
  5. disjoint set: it is the corner case of the previous one. Practically this case is totally irrelevant, this shows
that your program solves a completely different problem than expected by the specification

### required > actual

The typical example for this case is the division method:

```cpp
double divide(double a, double b);
```

What should it return if `b` equals to `0`?

One could choose a value as a *null* value. The `0` is a typical *null* value for numbers, while empty is for strings or
containers and `nullptr` for pointers. Please note that by this "solution" information can be easily lost, because if we
choose `0` as result of a "zero-division" error, we will be unable to distinguish it from the case when `a` equals to
`0`.

Another solution can come into our mind: return a `pair<bool, double>`, where the `bool` is `false` if division is
impossible. By this solution we covered the specification, but more than required.


### required < actual

So let examine the `pair<bool, double> divide(double a, double b);` function. The main problem with that is the caller
is not forced to check the result, he is able to use the returned `double`, but what should be its value if the division
has been failed? (This question is out of the domain specification.)

But too much possible input can also cause problems. If we want to turn autopilot on and off, then taking `string` as a
parameter is a bad idea. Although it provides flexibility ("on"/"off", "true"/"false", "enabled"/"disabled" can be
provided), but what should be happen if the input is not ane of these expected values? Turn it on? Off? Leave unchanged?
But actually these values must be handled somehow, which requires extra validation, corresponding and exhaustive unit
tests. And finally we will have solved a problem, that should not exist at all. It just created by us because of
choosing a wrong representation

TODO setAutopilot


TODO domain-requirements can be scaled up/down: your customer might not explicitly asked a divide method, but your
solution might requires it. At one layer down, this becomes a domain-requirement. This is another, but so important
question, that choosing this solution at one layer up is the best solution for that problem? 


Sofware Flexibility has (minimum) three aspects:
 * number of states
 * flexibility of the control flow
 * compile-time vs runtime flexibility


[Value/State flexibility](flexibility_value.md)
[Control flow flexibility](flexibility_control_flow.md)
[Compile-time vs runtime flexibility /TODO](TODO.md)






