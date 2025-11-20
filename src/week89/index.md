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
- It will not type-check with any possible set of annotations in our type
  system

It's also a nice representation of what “legacy code” might look like in a
language like JavaScript or Python, which often allow this kind of type-based
overloading of functions.

It would be nice if this function could type-check for our JIT. To accommodate
this we will add some type-based optimization, and have four special if rules:

```
Γ (if (isbool e) e2 e3) : T
  when Γ e2 : T
   and Γ e : Bool

Γ (if (isbool e) e2 e3) : T
  when Γ e3 : T
   and Γ e : Num

Γ (if (isnum e) e2 e3) : T
  when Γ e3 : T
   and Γ e : Bool

Γ (if (isnum e) e2 e3) : T
  when Γ e2 : T
   and Γ e : Num
```

These rules _ignore a branch_ of the if expression if the condition will
definitely evaluate to `true` or `false` based on known type information.

So, if we type-check the above function with `(n : Num)` and `(m : Num)`, we
will not consider the `(if (= n true) 1 -1)` expression in the type-checker.
Similarly, with `(n : Bool)` and `(m : Num)` we will not consider the `n`
sub-expression in the else branch. In both cases the relevant branch
type-checks, and the type of the let-bound `n` is `Num`.


## Types-only Optimizations

Update your compiler to generate more efficient code if the type-checker is
successful on a program. This could mean:

- Skipping tag checks on binary operations
- Simplifying `(isbool e)` and `(isnum e)`  to `true` or `false` if the type of
  `e` is known to be `Num` or `Bool`.
- Skipping the condition and unreachable branch in `(if (isbool e) e1 e2)` and `(if
  (isnum e) e1 e2)` when the type of `e` is known
- Reducing `(cast T e)` to `e` if the type of `e` is `≤ T`
- You're free to add other opportunities you see!

In your PDF report, include the following:

1. Write one or more programs that trigger the specific optimizations listed
   above. For each of the programs, run it with `-g` and `-tg` and show:
  - That the output of the program is correct and the same
  - That the assembly is different, and shorter, when run with `-t`

2. Consider the program below.

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
    
   - Run it with the `-g` and `-tg` options. Show in the report the test
     program, the assembly output and answer from the `-g` case, and the type
     error in the `-tg` case.
   - Make it type-check by adding only cast expressions (leave the function
     un-annotated). Show the resulting program and run it with the `-g` and
     `-tg` cases, showing the optimizations and output.

## JIT-based Optimizations

The type-based optimizations are ineffective on un-annotated functions without
casts, especially when we can't infer anything about functions' types from
their call sites, which we may only find out about later.

(_For the computational model you should have in mind, consider a web page,
where dynamically-loaded scripts may calculate the type of their arguments to a
function only in response to user input, or a computational notebook where
types may only be known once a CSV file is read and its columns parsed. `input`
is our proxy for the unknowns in these situations._)

Implement a _just-in-time_ compilation for (un-type-checked) functions that
optimizes them when they are _first called_ based on the types of the arguments
in that call. When run in non-type-checking mode:

- Each function should compile to a “stub” (and potentially a “slow” version)
- The stub should call back into the compiler with (a) an id for the funciton
  itself and (b) the given arguments from the first call.
- The compiler should type-check the function based on the types of these
  arguments. If type-checking is successful, it should generate an optimized
  version of the function (as above), and reconfigure the stub to call the
  optimized version. If type-checking is not successful, it should reconfigure
  the stub to call the slow version always.




