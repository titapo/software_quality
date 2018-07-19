# Expressiveness

## Levels of documentation

### 1. Out-of-code documentation

This is the case when there is a documentation managed separately from the code. This is the worst case, because it
is very easy to ignore both for a user to read, and for the maintainer to update. When a separate documentation needed,
its better to generate it from the code (See the next point)

### 2. Inline documentation (comments)

Writing explanation comments to the code can be useful in some cases, but you should feel bad if you don't have any
other tool to describe what your program does (or intents to do). These comments are very fragile in the meaning that it
can diverge from the code, especially if it is an unclear comment: if one does not understand it, he scarcely will
update or remove it when it becomes outdated.

Please see the following function. Imagine that this is a monolithic function with more than 100 lines that takes all
responsibility of processing a message.

```cpp
int processMessage(const Message& message)
{
  // Check message integrity
  if (/* ... */)
    /* ... */

  // Authorize the sender
  if (/* ... */)
    /* ... */

  // Process the message
  switch (message.type)
  {
    case /* ... */:
    /* Several more cases ... */
  }
}
```

These comments might help to understand the code, but it is still true that easy to diverge from the actual content of
the code. These comment are undoubtedly useful for refactoring: they lead your hand to part into smaller functions and
recommend names for these functions.

In real life the refactoring is not such simple thing, because tipically there are more coupling within the function.
For example there is a try catch block, that contains one or more of these blocks, and here the investigation can be
started: which exception can be thrown from where? That's why [exceptions are obfuscate](flexibility_control_flow.md)

### 3. Weak notations

There are elements that are parts of the language, for example *typedef aliases*.

```cpp
using UserId = unsigned;
```

This expresses the intention better than a comment, like: `/* user id */`. It is useful for refactorings, too, because
ideally the programmer has to change the code at fewer places. But unfortunately it still can be ignored at any point.
Do not forget that alias does not change anything just makes your code more reasonable and closer to the domain.

Even worse that in this particular case that `UserId` can be converted to `double` back-and-forth and it is true for a
lot more primitive types.

/// TODO other weak notations

### 4. Tests

Unit tests are generally not considered as documentation, but actually these are the primary keepers of the actual
behaviour of the units. Please note that the content of the unit tests cannot be ignored, they must be updated if the
effective behaviour of the program changed.

### 5. Strong notations

/// TODO strong typedefs




//TODO signal-to-noise ratio

