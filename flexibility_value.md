## Value/State flexibility

This is the most obvious case. What are the the possibly representable values of your program at a specific point?
And how this number is related to domain requirements?
Function signatures are tipically indicate this information, both on input and output side. Let see the division problem
again! It can succeded or failed because of zero division error, so the required number of output values:

```
#(double) + 1
```

and let see the possible values of following signatures:

(1) `double divide(double, double)` | `#(double)`

it is insufficient. This falls into a *null*-value problem. Otherwise the number of possible states must be extended.

(2) `pair<bool, double> divide(double, double)` | `#(double) * #(bool)`

which is too much mathematically! And in practice if `false` means *failure*, what should be the result of the `double`?
It is a very serious question because the success flag can be easily ignored. And this shows how the additional,
redundant values lead to invalid states. Based on the problem it possibly can be a security issue, a safety problem, or
any kind of bug.

(3) `std::unique_ptr<double> divide(double, double)` | `#(double) + 1`

Hmm... Seeing the numbers only this seems to be a suitable solution. Even if it is not a cheap one, because it requires heap
allocation, and adds extra indirection to access the value. Later we will see (other aspects of flexibility), that this
solution is not acceptable for several reasons. (So viewing problems from one aspect only is not a good idea.)
Currently I just want to mention that the *null* information is held by the value.

(4) `optional<double> divide(double, double)` | `#(double) + 1`

Actually this is a real solution for the problem. The number of possible states matches with the required states, but
there is no heap allocation [TODO reference to the standard], and we have to pay for what is essentially required to
represent this information. The only problem with that is the interface of the `std::optional` interface. Namely we have
to write an `if` to handle the error state separately... But in general we can say that the *success/failure*
information is expressed by type and not by value.



/// TODO information is expressed by type and not by value. Why is it good?

/// TODO optional<int> divide(int, int) is still insufficient

/// TODO state machine example for variant

