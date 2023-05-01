![diamondback](./diamondback.jpeg)

# Week 4: Diamondback, Due Tuesday, May 2 (Open Collaboration)

_(Yes, that's 3 open collaboration assignments in a row 🙂)_

In this assignment you'll implement a compiler for a language called Diamondback,
which has top-level function definitions.

## Setup

Get the assignment at FILL This will make
a private-to-you copy of the repository hosted within the course's
organization.  You can also access the public starter code
FILL if you don't have or
prefer not to use a Github account.

## The Diamondback Language

### Concrete Syntax

The concrete syntax of Diamondback has a significant change from past
languages. It distinguishes _top-level declarations_ from expressions.  The new
parts are function definitions, function calls, and the `print` unary operator.

```
<prog> := <defn>+ <expr>                (new!)
<defn> := (fun (<name> <name>*) <expr>) (new!)
<expr> :=
  | <number>
  | true
  | false
  | input
  | <identifier>
  | (let (<binding>+) <expr>)
  | (<op1> <expr>)
  | (<op2> <expr> <expr>)
  | (set! <name> <expr>)
  | (if <expr> <expr> <expr>)
  | (block <expr>+)
  | (loop <expr>)
  | (break <expr>)
  | (<name> <expr>*)                    (new!)

<op1> := add1 | sub1 | isnum | isbool | print (new!)
<op2> := + | - | * | < | > | >= | <= | =

<binding> := (<identifier> <expr>)
```


### Abstract Syntax

You can choose the abstract syntax you use for Diamondback.

### Semantics

A Diamondback program always evaluates to a single integer, a single boolean,
or ends with an error. When ending with an error, it should print a message to
_standard error_ (`eprintln!` in Rust works well for this) and a non-zero exit
code (`std::process::exit(N)` for nonzero `N` in Rust works well for this).

A Diamondback program starts by evaluating the `<expr>` at the end of the
`<prog>`. The new expressions have the following semantics:

- `(<name> <expr>*)` is a _function call_. It first evaluates the expressions
  to values. Then it evaluates the body of the corresponding function
  definition (the one with the given `<name>`) with the values bound to each of
  the parameter names in that definition.
- `(print <expr>)` evaluates the expression to a value and prints it to
  standard output followed by a `\n` character. The `print` expression itself should
  evaluate to the given value.

There are several examples further down to make this concrete.

The _compiler_ should stop and report an error if:

* There is a call to a function name that doesn't exist
* Multiple functions are defined with the same name
* A function's parameter list has a duplicate name
* There is a call to a function with the wrong number of arguments
* `input` is used in a function definition (rather than in the expression at
  the end). It's worth thinking of that final expression as the `main` function
  or method

If there are multiple errors, the compiler can report any non-empty subset of
them.

Here are some examples of Diamondback programs.

#### Example 1

```scheme
(fun (fact n)
  (let
    ((i 1) (acc 1))
    (loop
      (if (> i n)
        (break acc)
        (block
          (set! acc (* acc i))
          (set! i (+ i 1))
        )
      )
    )
  )
)
(fact input)
```

#### Example 2

```scheme
(fun (isodd n)
  (if (< n 0)
      (isodd (- 0 n))
      (if (= n 0)
          false
          (iseven (sub1 n))
      )
  )
)

(fun (iseven n)
  (if (= n 0)
      true
      (isodd (sub1 n))
  )
)

(block
  (print input)
  (print (iseven input))
)
```

### Implementing a Compiler for Diamondback

The main new feature in Diamondback is functions. You should choose and
implement a calling convention for these. You're welcome to use the
“standard” x86_64 sysv as a convention, or use some of what we discussed in
class, or choose something else entirely. Remember that when calling _runtime_
functions in Rust, the generated code needs to respect that calling convention.

A compiler for Diamondback does not need guarantee safe-for-space tail calls,
but they are allowed.

### Running and Testing

Running and testing are as for Cobra, there is no new infrastructure.

## Grading

As with the previous assignment, a lot of the credit you get will be based on
us running autograded tests on your submission. You'll be able to see the
result of some of these on while the assignment is out, but we may have more
that we don't show results for until after assignments are all submitted.

We'll combine that with some amount of manual grading involving looking at your
testing and implementation strategy. You should have your own thorough test
suite (it's not unreasonable to write many dozens of tests; you probably don't
need hundreds), and you need to have recognizably implemented a compiler. For
example, you _could_ try to calculate the answer for these programs and
generate a single `mov` instruction: don't do that, it doesn't demonstrate the
learning outcomes we care about.

Any credit you lose will come with instructions for fixing similar mistakes on
future assignments.

## Grading

As with the previous assignment, a lot of the credit you get will be based on
us running autograded tests on your submission. You'll be able to see the
result of some of these on while the assignment is out, but we may have more
that we don't show results for until after assignments are all submitted.

We'll combine that with some amount of manual grading involving looking at your
testing and implementation strategy. You should have your own thorough test
suite (it's not unreasonable to write many dozens of tests; you probably don't
need hundreds), and you need to have recognizably implemented a compiler. For
example, you _could_ try to calculate the answer for these programs and
generate a single `mov` instruction: don't do that, it doesn't demonstrate the
learning outcomes we care about.

Any credit you lose will come with instructions for fixing similar mistakes on
future assignments.

## Extension 1: Proper Tail Calls

Implement safe-for-space tail calls for Diamondback. Test with deeply-nested
recursion. To make sure you've tested _proper tail calls_ and not just _tail
recursion_, test deeply-nested mutual recursion between functions with
different numbers of arguments.

## Extension 2: Compiling Functions with Dynamically-Discovered Types

Consider a function like this one from class:

```
(fun (sumrec num sofar)
  (if (= num 0)
      sofar
      (sumrec (+ num -1) (+ sofar num))))
```

Because this function _could_ be called with booleans for `num` or `sofar`, the
compiler is obligated to insert tag checks for the `=` and `+` operations here.
(Actually, there are some kinds of purely-static optimizations we could do
here; if control-flow reaches the else branch we know that `num` is a number
because otherwise the `=` check would have errored, so could elide the checks
for `(+ num -1)`. However that won't be the focus of this extension.)

We _could_ save some work by doing all the tag checks before entering the
function body, compiling specific versions of the function for each different
combination of arguments, then dispatching to the correct one based on the
observed tags. This would re-use some of the ideas from the previous assignment
on using the observed types of `input` and `define`d variables to specialize
code. In the `sumrec` example, the compiler would generate _4_ different
functions for something like `sumrec`:

```
(fun (sumrec num sofar)
  (case
    [(and (isnum num) (isnum sofar)) (sumrec_assume_num_num num sofar)]
    [(and (isnum num) (isbool sofar)) (sumrec_assume_num_bool num sofar)]
    [(and (isbool num) (isnum sofar)) (sumrec_assume_bool_num num sofar)]
    [(and (isbool num) (isbool sofar)) (sumrec_assume_bool_bool num sofar)]))
```

However, this quickly explodes generated code size as we introduce more types
and more arguments (it's (#types) _to the power of_ (#args), and for a
general-purpose compiler we should avoid creating binaries that are exponential
in the size of the source program!). We need to be a bit more clever.

For this extension, we'll pick a simple model that is surprisingly effective
and uses the dynamic code-generation techniques we've been studying:

> The **first time** a function is called, the types of the arguments are
> likely the types that will be given to that function again in the future, and
> it's worth specializing for that case.

This would allow us to compile just _two_ versions of a function:

- One that handles all possible combinations of arguments with all tag checks,
  making no assumptions
- One that is specialized to the types of the arguments seen for the first call
  to the function

The first version is the one generated by the standard compiler. Generating the
second is where the dynamic work happens.

There are a number of ways to set this up. We recommend making use of some of
the _dynamic assembly editing_ features of `dynasm-rs`.








