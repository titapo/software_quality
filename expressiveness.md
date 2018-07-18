# Expressiveness

## Levels of documentation

1. Out-of-code documentation
   This is the case when there is a documentation managed separately from the code. This is the worst case, because it
is very easy to ignore both for a user to read, and for the maintainer to update. When a separate documentation needed,
its better to generate it from the code (See the next point)

2. Inline documentation (comments)
   Writing explanation comments to the code can be useful in some cases, but you should feel bad if you don't have any
other tool to describe what your program does (or intents to do). These comments are very fragile in the meaning that it
can diverge from the code, especially if it is an unclear comment: if one does not understand it, he scarcely will
update or remove it when it becomes outdated.

Please see the following function. Imagine that this is a monolithic

```cpp
int processMessage(const Message& message)
{
  // Check message integrity
  if (/* ... */)
    /* ... *

  // Authorize the sender
  if (/* ... */)
    /* ... */

  // Process the message
  switch (message.type)
  {
    /* .. */
  }
}
```

3. weak notations
4. tests
5. strong notations


//TODO signal-to-noise ratio

