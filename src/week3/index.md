![cobra](./cobra.jpg)

# Week 4: Cobra, Due Wednesday, October 22

In this assignment you'll implement a compiler for a small language called Cobra,
which extends Boa with booleans, conditionals, variable assignment, and loops.

## Setup

Get the assignment at <https://classroom.github.com/a/hQU87gZn> This will make
a private-to-you copy of the repository hosted within the course's organization.  You can also access the public test + 'starter' code <https://github.com/ucsd-cse131-fa25/cobra-test> if you don't have or
prefer not to use a Github account.

Note: the repository has no real code, just a basic project structure. Feel free to add files and modify them, or even replace them entirely and start from your Boa code. Make sure your code can do `cargo build` and `cargo test` on ieng6 (Rust version `1.75`).
- **Part 1**: _AOT_ compilation of Cobra files with a generated assembly file. This is with the `-c` compile flag. The optional argument is only given to the executable `.run` file.
- **Part 2**: _JIT_ compilation of Cobra files with an evaluation at runtime. This is with the `-e` eval flag with an optional argument.
  - The `-g` flag does both (with an optional argument).
- **Part 3**: A REPL for Cobra with the `-i` interactive flag.

```
cargo run -- -c tests/test1.snek tests/test1.s
cargo run -- -e tests/test1.snek <optionalArg>
cargo run -- -g tests/test1.snek tests/test1.s <optionalArg>
cargo run -- -i
```

## Part 1: The Cobra Language


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

You can choose the abstract syntax you use for Cobra. We recommend something like this:

```
enum Op1 { Add1, Sub1, IsNum, IsBool, }

enum Op2 { Plus, Minus, Times, Equal, Greater, GreaterEqual, Less, LessEqual, }

enum Expr {
    Number(i64),
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
with an error. When ending with an error, it should print a message to
_standard error_ (`eprintln!` in Rust works well for this) and a non-zero exit
code (`std::process::exit(N)` for nonzero `N` in Rust works well for this).

- `input` expressions evaluate to the first command-line argument given to the
  program. The command-line argument can be any Cobra value: a valid number or
  `true` or `false`. If no command-line argument is provided, the value of
  `input` is `false`. When running the program the argument should be provided
  as `true`, `false`, or a base-10 number value.
- All Boa programs evaluate in [the same way as before](../week2/index.md), with one
  exception: if numeric operations would overflow a 63-bit integer, the program
  should end in error, **reporting `"overflow"` as a part of the error**.
- If the operators other than `=` are used on booleans, an error should be
  raised from the running program, and **the error should contain "invalid
  argument"**. Note that this is not a compilation error, nor can it be in all
  cases due to `input`'s type being unknown until the program starts.
- The relative comparison operators like `<` and `>` evaluate their arguments
  and then evaluate to `true` or `false` based on the comparison result.
- The equality operator `=` evaluates its arguments and compares them for
  equality. It should raise an error if they are not both numbers or not both
  booleans, and the error should contain **"invalid argument"** if the types
  differ.
- Boolean expressions (`true` and `false`) evaluate to themselves
- `if` expressions evaluate their first expression (the condition) first. If it's
  `false`, they evaluate to the third expression (the “else” block), and to
  the second expression if any other value (including numbers).
- `block` expressions evaluate the subexpressions in order, and evaluate to the
  result of the _last_ expression. Blocks are mainly useful for writing
  sequences that include `set!`, especially in the body of a loop.
- `set!` expressions evaluate the expression to a value, and change the value
  stored in the given variable to that value (e.g. variable assignment). The
  `set!` expression itself evaluates to the new value. If there is no surrounding
  let binding for the variable the identifier is considered unbound and an error
  should be reported.
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

**Abstract Syntax Based on Our Design**

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

```scheme
(let ((a 2) (b 3) (c 0) (i 0) (j 0))
  (loop
    (if (< i a)
      (block
        (set! j 0)
        (loop
          (if (< j b)
            (block (set! c (sub1 c)) (set! j (add1 j)))
            (break c)
          )
        )
        (set! i (add1 i))
      )
      (break c)
    )
  )
)
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

The [starter code](https://github.com/ucsd-compilers-s23/cobra-starter) makes a
few infrastructural suggestions. You can change these as you feel is
appropriate in order to meet the specification.

#### Reporting Dynamic Errors

We've provided some infrastructure for reporting errors via the
[`snek_error`](https://github.com/ucsd-compilers-s23/cobra-starter/blob/main/runtime/start.rs#L13)
function in `start.rs`. This is a function that can be _called from the
generated program_ to report an error. for now we have it take an error code as
an argument; you might find the error code useful for deciding which error
message to print.  This is also listed as an `extern` in [the generated
assembly startup
code](https://github.com/ucsd-compilers-s23/cobra-starter/blob/main/src/main.rs#L17).

#### Calculating Input

We've provided a
[`parse_input`](https://github.com/ucsd-compilers-s23/cobra-starter/blob/main/runtime/start.rs#L27)
stub for you to fill in to turn the command-line argument to `start.rs` into a
value suitable for passing to `our_code_starts_here`. As a reminder/reference,
the first argument in the x86_64 calling convention is stored in `rdi`. This
means that, for example, moving `rdi` into `rax` is a good way to get “the
answer” for the expression `input`.

#### Representations

In class we chose representations with `0` as a tag bit for numbers and `1` for
booleans with the values `3` for `true` and `1` for `false`. You **do not**
have to use those, though it's a great starting point and we recommend it. Your
only obligation is to match the behavior described in the specification, and if
you prefer a different way to distinguish types, you can use it. (Keep in mind,
though, that you still must generate assembly programs that have the specified
behavior!)

### Running and Testing

The test format changed slightly to require a _test name_ along with a _test
file name_. This is to support using the same _test file_ with different
_command line arguments_. You can see several of these in the [sample
tests](https://github.com/ucsd-compilers-s23/cobra-starter/blob/main/tests/all_tests.rs).
Note that providing `input` is optional. These also illustrate how to check for
errors.

If you want to try out a single file from the command line (and perhaps from a
debugger like `gdb` or `lldb`), you can still run them directly from the
command line with:

```
$ cargo run -- -c test/file.snek test/file.s
$ make file.run
$ ./tests/file.run 1234
```

where the `1234` could be any valid command-line argument.

As a note on running all the tests, the best option is to use `make test`,
which ensures that `cargo build` is run first and independently before `cargo
test`.

## Part 2: Dynamic Compilation

### Using Dynamic Information to Optimize

A compiler for Cobra needs to generate extra instructions to check for booleans
being used in binary operators. We could use a static type-checker to avoid
these, but at the same time, the language is fundamentally dynamic because the
compiler cannot know the type of `input` until the program starts running
(which happens after it is compiled). This is the general problem that systems
for languages like JavaScript and Python face; it will get worse when we
introduce functions in the next assignment.

However, if our compiler can make use of some dynamic information, we can do
better.

There are two instructive optimizations we can make with dynamic information,
one for standalone programs and one at the REPL.

### Eval

Add a new command-line option, `-e`, for “eval”, that evaluates a program
directly after compiling it with knowledge of the command-line argument. The
usage should be:

```
cargo run -- -e file.snek <optionalArg>
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
throws an error as usual. If no arg is given, just like for AOT compilation, default the argument to false.

For easier debugging of your assembly (since we can't just view dynasm machine code), use the `-g` flag to generate your _optimized_ assembly and return your JIT output. It might be useful to have a custom `Instr` which simply serves as an assembly comment, which can print something like `; Optimized instructions out here`.

## Part 3: The REPL

### Known Variables at the REPL

Similarly, after a `define` statement evaluates at the REPL, we can know that
variable's tag and use that information to compile future entries. For example,
in this REPL sequence, we define a numeric variable and use it in an operator
later. We could avoid tag checks for `x` in the later use:

```
> (define x (+ 3 4))
> (+ x 10)               // No number checks
17
```

Note a pitfall here – if you allow `set!` on `define` d variables, their values
could change mid-expression. To continue the above interaction:

```
> (block (+ 3 x) (set! x true) (+ 1 x))  // Would be wrong to inline the value of x in the final expression!
error // should be error, but could mistakenly be 18
```


So how do we solve this optimization problem? If the variable is `set!` mid-expression, we don't know what the variable is yet. Therefore, if a prompt has a `set!` on a `define` d variable, we cannot optimize. But after the prompt has run, we can determine what the value and type of `define` d is again, so we optimize!

But now we run into another issue...

Before, in order to get the value of `define` d when matching an `Expr::Id`, we would just inline the immediate value associated with d. But we don't know what this immediate value is yet! We cannot put this varible on the stack, either, because we update our `RBP` each prompt. So what if we use the heap? 

The solution here is to inline a _reference_ to the immediate. This can be done with `Box` of our `define` d value within our Rust runtime. We can inline this raw pointer to a heap allocated value into our assembly. We dereference the pointer into `RAX` to get the `Expr::Id`. Allocation only needs to be done once for each `define` d value if it was ever `set!` d. Then after running our prompt, we use a known immediate and type again.

You should make a helper function that checks which `define` d variables are `set!`. If they are `set!`, allocate a new `Box` for their old value and cast the reference into a raw pointer.

```rust
// For each new define d variable that is also first `set!`
let val: i64 = 777;
let boxed = Box::new(val);                   // Heap allocated i64
let ptr = Box::into_raw(boxed) as i64;       // Raw pointer as i64
```

Then in our actual assembly that is generated, we can dereference this pointer to get the value for `define` d. Notice that we don't need to know what the actual immediate value is. So if we do `(add1 d)`

```
; Get the raw pointer, then dereference
MOV RAX, <a raw ptr as an i64>
MOV RAX, [RAX]
ADD RAX, 2
```

What about `set!`? Unlike `let` variables, we cannot use the stack. So we need to store our `RAX` into the memory location of our raw pointer! So if we do `(set! d 7)`

```
; Store our value in rax into the heap
MOV RAX, 14
MOV RCX, <a raw ptr as an i64>
MOV [RCX], RAX
```

Now, after everything is done, we can map `define` d into a known value (and type) again. We know _where_ the value is (the raw pointer), so how do we dereference it within Rust? 

```rust
// After the prompt has run...
let boxed = unsafe { Box::from_raw(ptr as *mut i64) };
let val = *boxed;       // Dereference a Box
```
Here we let Rust manage our i64 value in our heap by passing our raw pointer to a Rust `Box`, which has all of the nice deconstructing and type invariants with it. However, this code is unsafe because we need to be _very_ sure that our raw pointer is correct and not something we freed before. Rust is really trusting our pointer to not be wrong or malicous here! Finally, we dereference our `Box` as our (hopefully) new known i64 value for `define` d.

It might be a good idea to have a helper function that parses your `Expr` for `set!` on any `define` d variables. Those variables cannot have a known type during the prompt running.

It might also be a good idea to have an _environment_ of sorts for these `define` d variables that should tell you when a variable is of a known or unknown type. If known, you can optimize and inline an immediate. If unknown, so you cannot optimize and you have to dereference a pointer.

Happy hacking!

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

## FAQ

**Some of my tests fail with a `No such file or directory` error**

The initial version of the starter code contained an error in the testing infrastructure. If you
cloned before we fixed it, you'll have to update the code. You can update the code by running:

```console
git remote add upstream https://github.com/ucsd-compilers-s23/cobra-starter
git pull upstream main --allow-unrelated-histories
```

This will merge all commits from the template into your repository. Alternatively, you can also
clone <https://github.com/ucsd-compilers-s23/cobra-starter> and manually replace your `tests/`
directory.

### Discussion

It's worth re-emphasizing that a static type-checker could recover a lot of
this performance, and for Cobra it's pretty straightforward to implement a
type-checker (especially for expressions that don't involve `input`).

However, we'll soon introduce functions, which add a whole new layer of
potential dynamism and unknown types (because of function arguments), so the
same principles behind these simple cases become much more pronounced. And a
language with functions and a static type system is quite different from a
language with functions and no static type system.

