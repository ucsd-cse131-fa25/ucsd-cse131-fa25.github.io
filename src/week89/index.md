![forest-flame](https://upload.wikimedia.org/wikipedia/commons/4/40/Chrysopelea_taprobanica.jpg)

# Week 8-10: Flying Snake

**Due Thursday, December 4, 11:59pm**

This week, you will _take off_ (get it, flying snake) in your own direction
implementing optimizations for Eastern Diamondback (both traditional syntactic
optimizations, and type- and JIT-based ones).

There are some significant format differences from the other assignments:

- You can work alone, or in groups of 2 or 3. You _must_ pre-register your
  groups by Friday, November 21 via [FILL form].
- There are no new language features – the specification of the language is
  exactly the same as in Eastern Diamondback, with the exception of a special
  type rule shown below.
- There is no autograder. You will submit a PDF report along with your code.

## Type Checking and Dead Code

Consider this function:

```
(fun (asnum n m)
  (let ((n (if (isbool n) (if (= n true) 1 -1) n)))
    (* m n)))
```

A few things are true about this function:
- It never results in a runtime error no matter the argument value
- It always returns a number no matter the argument value
- It will not type-check with any possible set of annotations



## Types-only Optimizations

Update your compiler to generate more efficient code if the type-checker is
successful on a program. This could mean:

- Skipping tag checks on binary operations
- Reducing `(isbool e)`  to `true` or `false` if the type of `e` is known
- Reducing `(cast T e)` to `e` if the type of `e` is `≤ T`
- Other opportunities you see

In your PDF report, include the following:

1. For each of the program descriptions below, write a test and run it with the
`-g` and the `-tg` option.  Show in the report the test program, the assembly
output from both cases, and confirm that (a) the answer is correct in both
cases and (b) the type-checked program output is shorter:
  - A program with a function with two arguments, one `Num` and one `Bool` and
    does some binary operation work with one or both, and a main expression
    that just calls the function with one constant value (like a number or
    `true`) and the other argument `input` cast to the correct type.
  - A program with a loop that does arithmetic and runs at least 1000 times.
  - A program with a function with an `Any`-typed argument that then casts that
    argument _within_ the function to `Num`, so it can type-check and optimize.
2. For the program below, run it with the
   `-g` and `-tg` options. Show in the report the test program, the assembly
   output and answer from the `-g` case, and the type error in the `-tg` case.

   ```
   (fun (sum start stop step)
     (let ((step (if (isbool step) (if (= step true) 1 -1) step))
           (res 1)
           (curr start))
      (loop
        (if (if (> step 0) (>= curr stop) (<= curr stop)) (break res)
            (block
              (set! res (+ res curr))
              (set! curr (+ curr step)))))))
   (block
    (print (sum 3 10 1))
    (print (sum 10 3 false))
    (print (sum true false 1)))

   ```

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


