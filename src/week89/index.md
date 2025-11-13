![forest-flame](https://upload.wikimedia.org/wikipedia/commons/4/40/Chrysopelea_taprobanica.jpg)

# Week 8-10: Flying Snake

This week, you will _take off_ in your own direction implementing optimizations
for Eastern Diamondback (both traditional syntactic optimizations, and type-
and JIT-based ones).

There are some significant format differences from the other assignments:

- You can work alone, or in groups of 2 or 3
- There are no new language features – the specification of the language is
  exactly the same as in Eastern Diamondback
- There is no autograder and you will submit a PDF report along with your code

## Types-only Optimizations

Update your compiler to generate more efficient code if the type-checker is
successful on a program. This could mean:

- Skipping tag checks on binary operations
- Reducing `(isbool e)`  to `true` or `false` if the type of `e` is known
- Reducing `(cast T e)` to `e` if the type of `e` is compatible with `T`
- Other opportunities you see

This applies to all the different modes.

... TODO how to submit/which examples ...

## JIT-based Optimizations

The type-based optimizations are ineffective on un-annotated functions,
especially at the REPL when we can't infer anything about functions' types from
their call sites, which we may only find out about later.

(_For the computational model you should have in mind, consider a web page,
where dynamically-loaded scripts may calculate the type of their arguments to a
function only in response to user input, or a computational notebook where
types may only be known once a CSV file is read and its columns parsed. `input`
is our proxy for the unknowns in these situations._)

Your improvements for this assignment should include _compiling or specializing
a function based on the values of its arguments_. In class, we talked a lot
about how to do this for the _first_ time a function is called, which we
recommend if you're not sure where to start, but there are other policies you
could use.

The key idea is that the function should specialize if it is type-checkable
with the types of arguments it is given at runtime. This means that the
generated code for the function needs to call back into the Rust compiler,
generate specialized code if type-checking succeeds, and continue.

... TODO how to submit/which examples ...

_Notes – should show calling the fucntion with different types after
specialization and have the right behavior_

## Standard Optimizations

You should also implement _standard optimizations_. Choose N of:

- Constant folding
- Constant propagation
- Dead code elimination
- Instruction selection on known subexpressions (e.g. `(if (< 4 5) ...)` never needs to materialize a `true` if it can jump correctly)
- Using an explicit target to remove extra moves
- Using registers instead of memory for some operations

... TODO how to show it? ...

