![cobra](./cobra.jpg)

# Week 3: Cobra, Due Tuesday, April 25 (Open Collaboration)

_(Yes, that's 3 open collaboration assignments in a row 🙂)_

In this assignment you'll implement a compiler for a small language called Cobra,
which extends Boa with booleans, conditionals, variable assignment, and loops.

## Setup

Get the assignment at FILL.github.com/a/1bvTt9dk>. This will make a
private-to-you copy of the repository hosted within the course's organization.
You can also access the public starter code FILL if you don't have or prefer
not to use a Github account.

## The Cobra Language


### Concrete Syntax

The concrete syntax of Cobra is:

```
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

<op1> := add1 | sub1 | isnum | isbool
<op2> := + | - | * | < | > | >= | <= | =

<binding> := (<identifier> <expr>)
```

`true` and `false` are literals. Names used in `let` cannot have the name of
other keywords or operators (like `true` or `false` or `let` or `block`).
Numbers should be representable as a signed 63-bit number (e.g. from
-4611686018427387904 to 4611686018427387903).

### Abstract Syntax

The abstract syntax of Cobra is a Rust `enum` that extends the one used in Boa.

<!--- previous ocaml types
```
type prim1 =
  | Add1
  | Sub1

type prim2 =
  | Plus
  | Minus
  | Times

type expr =
  | Number of int
  | Id of string
  | Let of (string * expr) list * expr
  | Prim1 of prim1 * expr
  | Prim2 of prim2 * expr * expr
```
 --->

```
enum Op1 {
    Add1,
    Sub1
}

enum Op2 {
    Plus,
    Minus,
    Times,
    Equal,
    Greater,
    GreaterEqual,
    Less,
    LessEqual
}

enum Expr {
    Number(i32),
    Boolean(bool),
    Id(String),
    Let(Vec<(String, Expr)>, Box<Expr>),
    UnOp(Op1, Box<Expr>),
    BinOp(Op2, Box<Expr>, Box<Expr>),
    If(Box<Expr>, Box<Expr>, Box<Expr>),
    Loop(Box<Expr>),
    Break(Box<Expr>),
    Set(String, Box<Expr>),
    Block(Vec<Expr>),
}
```

### Semantics

A "semantics" describes the languages' behavior without giving all of the
assembly code for each instruction.

A Cobra program always evaluates to a single integer, a single boolean, or ends
with an error.

- All Boa programs evaluate in [the same way as before](../week2/index.md), with one
  exception: if numeric operations would overflow a 63-bit integer, the program
  should end in error, reporting `"overflow"` as a part of the error.
- If the operators other than `=` are used on booleans, an error should be
  raised *from the compiled program*, and **the error should contain "invalid
  argument"**. Note that this is not a compilation error.
- Boolean expressions (`true` and `false`) evaluate to themselves
- `input` expressions evaluate to the first command-line argument given to the
  program. The command-line argument can be any value: a valid number or `true`
  or `false`. If no command-line argument is provided, the value of `input` is
  `false`.
- `if` expressions evaluate their first expression (the condition) first. If it's
  `false`, they evaluate to the third expression (the “else” block), and to
  the second expression if any other value (including numbers).
- `block` expressions evaluate the subexpressions in order, and evaluate to the
  result of the _last_ expression. Blocks are mainly useful for writing
  sequences that include `set!`
- `set!` expressions evaluate the expression to a value, and change the value
  stored in the given variable to that value (e.g. variable assignment). The
  `set!` expression itself evaluates to the new value.
- `loop` and `break` expressions work together. Loops evaluate their subexpression
  in an infinite loop until `break` is used. Break expressions evaluate their
  subexpression and the resulting value becomes the result of the entire loop.
  Typically the body of a loop is written with `block` to get a sequence of
  expressions in the loop body.

There are several examples further down to make this concrete.

The _compiler_ should stop and report an error if:

* There is a binding list containing two or more bindings with the same name.
  **The error should contain the string `"Duplicate binding"`**
* An identifier is unbound (there is no surrounding let binding for it) **The
  error should contain the string `"Unbound variable identifier {identifier}"`
  (where the actual name of the variable is substituted for `{identifier}`)**
* A `break` appears outside of any surrounding `loop`. **The error should
  contain "break"**
* An invalid identifier is used (it matches one of the keywords). **The error
  should contain "keyword"**

If there are multiple errors, the compiler can report any non-empty subset of
them.

Here are some examples of Cobra programs.

#### Example 1

**Concrete Syntax**

```scheme
(let ((x 5))
     (block (set! x (+ x 1))))
```

**Abstract Syntax**

```rust
Let(vec![("x".to_string(), Number(5))],
    Box::new(Block(
      vec![Set("x".to_string(),
               Box::new(BinOp(Plus, Id("x".to_string()),
                                    Number(1)))])))
```

**Result**

```
6
```


#### Example 2

- FILL something w nested loops

```scheme
(let ((a 2) (b 3) (c 0) (i 0) (j 0)) 
   (loop
      (if (< i a)
         (block
            (loop
               (if (< j b)
                  (block (set! c (add1 c)) (set! j (sub1 j)))
                  (break (c))
               )
            )
            (set! i (add1 i)))
         (break (c))
      )
   )
)
```

**Abstract Syntax**

```rust

```

**Result**

```
-6
```

#### Example 3

This program calculates the factorial of the input.
```scheme
(let
  ((i 1) (acc 1))
  (loop
    (if (> i input)
      (break acc)
      (block
        (set! acc (* acc i))
        (set! i (+ i 1))
      )
    )
  )
)
```

### Implementing a Compiler for Cobra

FILL

- Give starter code/class code for passing in `input`
- Give starter code/class code for throwing a runtime exception
- Discuss choices in representation
- Emphasize what needs to be dynamic vs. static


### Running

FILL

- How to pass a command-line arg, how to pass it in tests

### Ignoring or Changing the Starter Code

You can change a lot of what we describe above; it's a (relatively strong)
suggestion, not a requirement. You might have different ideas for how to
organize your code or represent things. That's a good thing! What we've shown
in class and this writeup is far from the only way to implement a compiler.

To ease the burden of grading, we ask that you keep the following in mind: we
will grade your submission (in part) by copying our own `tests/` directory in
place of the one you submit and running `cargo test -- --test-threads 1`. This relies on the
interface provided by the `Makefile` of producing `.s` files and `.run` files.
It _doesn't_ rely on any of the data definitions or function signatures in
`src/main.rs`. So with that constraint in mind, feel free to make new
architectural decisions yourself.

## Strategies, and FAQ

## Grading

A lot of the credit you get will be based on us running autograded tests on
your submission. You'll be able to see the result of some of these on while the
assignment is out, but we may have more that we don't show results for until
after assignments are all submitted.

We'll combine that with some amount of manual grading involving looking at your
testing and implementation strategy. You should have your own thorough test
suite (it's not unreasonable to write many dozens of tests; you probably don't
need hundreds), and you need to have recognizably implemented a compiler. For
example, you _could_ try to calculate the answer for these programs and
generate a single `mov` instruction: don't do that, it doesn't demonstrate the
learning outcomes we care about.

Any credit you lose will come with instructions for fixing similar mistakes on
future assignments.

## Extension: Using Dynamic Information to Optimize

A compiler for Cobra needs to generate extra instructions to check for booleans
being used in binary operators. We could use a static type-checker to avoid
these, but at the same time, the language is fundamentally dynamic because the
compiler cannot know the type of `input` until the program starts running
(which happens after it is compiled). This is the general problem that systems
for languages like JavaScript and Python face.

However, if our compiler can make use of some dynamic information, we can do
better.

There are two instructive optimizations we can make with dynamic information,
one for standalone programs and one at the REPL.

### Eval

Add a new command-line option, `-e`, for “eval”, that evaluates a program
directly after compiling it with knowledge of the command-line argument. The
usage should be:

```
cargo run -- -e file.snek <arg>
```

That is, you provide both the file and the command-line argument. When called
this way, the compiler should skip any instructions used for checking for
errors related to `input`. For example, for this program, if a number is given
as the argument, we could omit all of the tag checking related to the `input`
argument (and since `1` is a literal, we could recover essentially the same
compilation as for Boa).

```
(+ 1 input)
```

For this program, if `input` is a boolean, we should preserve that the program
throws an error as usual.

### Known Variables at the REPL

Similarly, after a `define` statement evaluates at the REPL, we can know that
variable's tag and use that information to compile future entries. For example,
in this REPL sequence, we define a numeric variable and use it in an operator
later. We could avoid tag checks for `x` in the later use:

```
> (define x (+ 3 4))
> (+ x 10)
```

Note a pitfall here – if you allow `set!` on `define`d variables, their types
could change mid-expression, so there are some restrictions on when this should
be applied.

### Discussion

It's worth re-emphasizing that a static type-checker could recover a lot of
this performance, and for Cobra it's pretty straightforward to implement a 
type-checker (especially for expressions that don't involve `input`).

However, we'll soon introduce functions, which add a whole new layer of
potential dynamism and unknown types (because of function arguments), so the
same principles behind these simple cases become much more pronounced. And a
language with functions and a static type system is quite different from a
language with functions and no static type system.

