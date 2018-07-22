# Good code

This chapter is inspired by the following idea (originated from Herb Sutter):

Good code by default is when ...

> Any bugs are sins of commission, not of omission.
> ([video](https://www.youtube.com/watch?v=JfmTagWcqoE))


// TODO convention enforcement? Compile time vs runtime


// TODO context and explanation

APPROACH         | ABSTRACTION LEVEL | REQUIRED KNOWLEDGE | SAFETY                 | ADDITIONAL COST 
-----------------|-------------------|--------------------|------------------------|------------------- 
Just work        | HIGH              | MINIMAL            | MAXIMUM                | POSSIBLY HIGH
Default          | MIDDLE            | MEDIUM             | APPROPRIATE            | BARELY NOTICEABLE
Override default | LOW               | EXPERT             | (POSSIBLY) ERROR-PRONE | NONE (HOPEFULLY)
