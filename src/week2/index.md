![boa](./boa.jpg)

# Week 2: Boa, Due Wednesday, October 8

In this assignment you'll implement a compiler for a small language called Boa,
that has let bindings and binary operators. The key difference between this
language and what we implemented in class is that there can be _multiple_
variables defined within a single let. This language has two modes, _Ahead-of-Time (AOT)_ and _Just-in-Time (JIT)_ compilation. 
- **Part 1**: Supporting the Boa Language and _AOT_ compilation of Boa files with a generated assembly file. This is with the `-c` compile flag to the main program.
- **Part 2**: _JIT_ compilation of Boa files with an evaluation at runtime, using [dynasm](https://github.com/CensoredUsername/dynasm-rs). This is with the `-e` eval flag to the main program. 
  - The `-g` flag prints the evaluation and writes out AOT assembly for debugging purposes.

## Setup

Get the assignment at <https://classroom.github.com/a/RfezDfWu>. This will make a
private-to-you copy of the repository hosted within the course's
organization. You can also access the public testing infrastructure code [directly from this
public URL](https://github.com/ucsd-cse131-fa25/test-starter).

## Part 1: The Boa Language

In each of the next several assignments, we'll introduce a language that we'll
implement.  We'll start small, and build up features incrementally.  We're
starting with Boa, which has just a few features – defining variables, and
primitive operations on numbers. Part 1 is about making an Ahead-of-Time (AOT) compiler for the Boa Language.

There are a few pieces that go into defining a language for us to compile:
* A description of the concrete syntax – the text the programmer writes.
* A description of the abstract syntax – how to express what the
  programmer wrote in a data structure our compiler uses.
* A *description of the behavior* of the abstract
  syntax, so our compiler knows what the code it generates should do.

### Concrete Syntax

The concrete syntax of Boa is:

```
<expr> :=
  | <number>
  | <identifier>
  | (let (<binding> +) <expr>)
  | (add1 <expr>)
  | (sub1 <expr>)
  | (+ <expr> <expr>)
  | (- <expr> <expr>)
  | (* <expr> <expr>)

<binding> := (<identifier> <expr>)
```

Here, a `let` expression can have one *or more* bindings (that's what the
`<binding> +` notation means). `<number>`s are in base 10 and must be
representable as an `i32`. `<identifier>`s are names and should be limited to
alphanumeric characters, hyphens, and underscores, and should start with a
letter. (The `sexp` library handles more than this, but this keeps things
nicely readable.)

### Abstract Syntax

The abstract syntax of Boa is a Rust `enum`. Note that this
representation is different from what we used in
[Adder](https://ucsd-compilers-s23.github.io/week1/index.html).

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
    Times
}

enum Expr {
    Number(i32),
    Id(String),
    Let(Vec<(String, Expr)>, Box<Expr>),
    UnOp(Op1, Box<Expr>),
    BinOp(Op2, Box<Expr>, Box<Expr>)
}
```

### Semantics

A "semantics" describes the languages' behavior without giving all of the
assembly code for each instruction.

A Boa program always evaluates to a single integer.

- Numbers evaluate to
themselves (so a program just consisting of `Number(5)` should evaluate to the
integer `5`).
- Unary operator expressions perform addition or subtraction by one on
their argument. If the result wouldn't fit in an `i32`, the program can
have any behavior (e.g. overflow with `add1` or underflow with `sub1`).
- Binary operator expressions evaluate their arguments and combine them
based on the operator. If the result wouldn't fit in an `i32`, the program can
have any behavior (e.g. overflow or underflow with `+`/`-`/`*`).
- Let bindings should use lexical scoping: evaluate all the binding expressions to
values one by one, and after each, store a mapping from the given name to the
corresponding value in both (a) the rest of the bindings, and (b) the body of
the let expression. Identifiers evaluate to whatever their current stored
value is.

There are several examples further down to make this concrete.

The _compiler_ should stop and report an error if:

* There is a binding list containing two or more bindings with the same name. **The error should contain the string `"Duplicate binding"`**
* An identifier is unbound (there is no surrounding let binding for it) **The error should contain the string `"Unbound variable identifier {identifier}"` (where the actual name of the variable is substituted for `{identifier}`)**

If there are multiple errors, the compiler can report any non-empty subset of
them.

Here are some examples of Boa programs. In the "Abstract Syntax" parts, we
assume that the program has appropriate `use` statements to avoid the `Expr::`
and other prefixes in order to write the examples compactly.

#### Example 1

**Concrete Syntax**

```scheme
5
```

**Abstract Syntax**

```rust
Number(5)
```

**Result**

```
5
```

#### Example 2

**Concrete Syntax**

```scheme
(sub1 (add1 (sub1 5)))
```

**Abstract Syntax**

```rust
UnOp(Sub1, Box::new(UnOp(Add1, Box::new(UnOp(Sub1, Box::new(Number(5)))))))
```

**Result**

```
4
```

#### Example 3

**Concrete Syntax**

```scheme
(let ((x 5)) (add1 x))
```

**Abstract Syntax**

```rust
Let(vec![("x".to_string(), Number(5))],
  Box::new(UnOp(Add1, Box::new(Id("x".to_string())))))
```

**Result**

```
6
```

#### More examples
```
(sub1 5)
# as an expr
UnOp(Sub1, Box::new(Number(5)))
# evaluates to
4
```

```
(let ((x 10) (y 7)) (* (- x y) 2))
# as an expr
Let(vec![("x".to_string(), Number(10)), ("y".to_string(), Number(7))],
    Box::new(BinOp(Times, Box::new(BinOp(Minus, Box::new(Id("x".to_string())),
                          Box::new(Id("y".to_string())))), Box::new(Number(2)))));
# evaluates to
6
```

### Implementing an AOT Compiler for Boa

You have a starter codebase from [Adder](/week1/) that has several pieces of
infrastructure:

* A main program (`main.rs`) that uses the parser and compiler to produce
  assembly code from an input Boa text file.
* A `Makefile` and a runner (`runtime/start.rs`) that you can modify from [Adder](/week1/). You may modify your `Makefile` to add new commands. 
* You will add your own tests by creating new .snek files in the `tests/` directory and compiling them like we did in Adder, but with an extra argument (`-c`) to specify  `cargo run -c in.snek out.s`. 
* To speed up the testing, you can write your own automated [unit testing](https://doc.rust-lang.org/book/ch11-01-writing-tests.html) infrastructure in the `tests/` directory.

### Writing the Parser

The parser will be given a S-expression representing the whole program, and
must build a AST of the `Expr` data type from this S-expression.

An S-expression in Rust is of the following type:
```
pub enum Sexp {
    Atom(Atom),
    List(Vec<Sexp>),
}
pub enum Atom {
    S(String),
    I(i64),
    F(f64),
}
```

Thus, an example S-expression that could be parsed into a program would be as
follows
```
List(vec![Atom("let"), List(vec![List(vec![Atom("x"), Atom("5")])]), Atom("x")])
```
which corresponds to
```
(let ((x 5)) x)
```
in Boa or
```rust
{
    let x = 5;
    x
}
```
in Rust.

This should then parse to the AST
```rust
Let(vec![("x".to_string(), Number(5))], Id("x".to_string()))
```
which can then be compiled.

Since most S-expressions are lists, you will need to check the first element of
the list to see if the operation to perform is a `let`, `add1`, `*`, and so on.
If an S-expression is of an invalid form, (i.e. a `let` with no body, a `+`
with three arguments, etc.) panic with a message containing the string
`"Invalid"`.

You can assume that an id is a valid string of form `[a-zA-z][a-zA-Z0-9]*`.
You will, however, have to check that the string does not match any of
the language's reserved words, such as `let`, `add1`, and `sub1`.

The parsing should be implemented in
```
fn parse_expr(s: &Sexp) -> Expr {
    todo!("parse_expr");
}
```

You can also implement a helper function `parse_bind`
```
fn parse_bind(s: &Sexp) -> (String, Expr) {
    todo!("parse_bind");
}
```
which may be helpful for handling `let` expressions.

### Writing the AOT Compiler

The primary task of writing the Boa compiler is simple to state: take an
instance of the `Expr` type and turn it into a list of assembly
instructions. Start by defining a function that compiles an expression into a
list of instructions:
```
fn compile_to_instrs(e: &Expr) -> Vec<Instr> {
    todo!("compile_to_instrs");
}
```

which takes an `Expr` value (abstract syntax) and turns it into a list of
assembly instructions, represented by the `Instr` type.  Use only the
provided instruction types for this assignment; we will be gradually expanding
this as the quarter progresses. The compiled `Instr` vector can be translated into both a string representation of the assembly code and dynasm instructions for Part 2's JIT compilation.

**Note**: For variable bindings, we used `im::HashMap<String, i32>` from the
[`im`](https://docs.rs/im/latest/im/) crate.
We use the immutable HashMap here to make nested scopes easier because we
found it annoying to remember to pop variables when you leave a scope.
You're welcome to use any reasonable strategy here.

The other functions you need to implement are:

```rust
fn instr_to_str(i: &Instr) -> String {
    todo!("instr_to_str");
}

fn val_to_str(v: &Val) -> String {
    todo!("val_to_str");
}
```

They render individual instances of the `Instr` type and `Val` type into a string
representation of the instruction. This second step is straightforward, but
forces you to understand the syntax of the assembly code you are generating.
Most of the compiler concepts happen in the first step, that of generating
assembly instructions from abstract syntax. Feel free to ask or refer to
on-line resources if you want more information about a particular assembly
instruction!

After that, put everything together with a `compile` function that compiles an
expression into assembly represented by a string.
```rust
fn compile(e: &Expr) -> String {
    todo!("compile");
}
```

### Assembly instructions

The `Instr` type is defined below. The assembly instructions that
you will have to become familiar with for this assignment are:

* `IMov(Val, Val)` — Copies the right operand (source) into the left operand
  (destination). The source can be an immediate argument, a register or a
  memory location, whereas the destination can be a register or a memory
  location.

  Examples:
  ```
    mov rax, rbx
    mov [rax], 4
  ```

* `IAdd(Val, Val)` — Add the two operands, storing the result in its first
  operand.

  Example: `add rax, 10`

* `ISub(Val, Val)` — Store in the value of its first operand the result of
  subtracting the value of its second operand from the value of its first
  operand.

  Example: `sub rax, 216`

* `IMul(Val, Val)` — Multiply the left argument by the right argument, and
  store in the left argument (typically the left argument is `rax` for us)

  Example: `imul rax, 4`

### Running

With the `-c` flag, run `cargo run -- -c test.snek test.s` to compile your snek file into assembly. You can modify your `Makefile` to add a command to do this for you.

```
$ cargo run -c -- test/add1.snek test/add1.s
$ cat test/add1.s

section .text
global our_code_starts_here
our_code_starts_here:
  mov rax, 131
  ret
```

To actually evaluate your assembly code, we need to link it with `runtime.rs` to
create an executable. This is covered in the `Makefile` from Adder.
```
$ make test/add1.run
nasm -f elf64 test/add1.s -o runtime/our_code.o
ar rcs runtime/libour_code.a runtime/our_code.o
rustc -L runtime/ runtime/start.rs -o test/add1.run
```
Finally you can run the file by executing to see the evaluated output:
```
$ ./test/add1.run
131
```
- Note: Locally on arm macs, to target with Rosetta 2, you need to use:
```
cargo run --target x86_64-apple-darwin -- -c test.snek test.s
```
and
```
nasm -f macho64
ar rcs runtime/libour_code.a runtime/our_code.o
rustc --target x86_64-apple-darwin -L tests/ -lour_code:$* runtime/start.rs -o tests/$*.run
```
OR you can simply add a new directory and file `.cargo/config.toml` with the following contents:
```
[build]
target = "x86_64-apple-darwin"
```
And run all commands normally.

## Part 2: Dynamic Compilation
Now that you have an AOT compiler with all of the Boa language features, you can use it to generate machine code and execute it at runtime. Just like in Adder, you will use the [dynasm](https://github.com/CensoredUsername/dynasm-rs) crate to generate machine code at runtime. 

The dynasm crate provides macros that allow you to write assembly-like code that gets compiled into a vector of bytes representing machine instructions. You can then execute this machine code by casting it to a function pointer using `mem::transmute`. This is exactly how you evaluate the Boa program directly without generating an assembly file or linking it to `runtime.rs`!

### Writing the JIT Compiler
Notice how we used the `Instr` type to represent assembly instructions in Part 1. This abstraction is useful for generating assembly code as a string. But how about for JIT compilation, where we need to generate machine code directly with dynasm's macros? Using a vector of `Instr`, instead of generating strings, is it possible to generate machine code directly? It may be useful to have a function like this:
```rust
fn instr_to_asm(i: &Instr, ops: &mut dynasmrt::x64::Assembler) {
    todo!("instr_to_asm");
}
```
Ideally, it would be nice to generate dynasm from all possible instructions; you may find it useful to restrict it to just instructions that you expect your compiler to produce. This may involve a lot of boilerplate; it's a good place to see if a LLM can do some boring work for you.

Although this is one approach, feel free to use any strategy you would like!

### Running

Now with the `-e` flag, run `cargo run -- -e test.snek` to compile your snek file and directly execute it. You can modify your `Makefile` to also include this command.

```
$ cargo run -e -- test/add1.snek
131
```

Here we have already evaluated our program at runtime! Notice that we did not link it to `runtime.rs` to create an executable. 

We also will include a `-g` with `cargo run -- -g test.snek test.s` flag that will do both AOT and JIT compilation. This is for debugging purposes, since we can evaluate Boa code directly at runtime and print out the (hopefully correct) generated assembly that should produce the same evaluation. In all, your AOT and JIT compilers should be producing the same results.

```
$ cargo run -- -g test.snek test.s
131
$ cat test/add1.s

section .text
global our_code_starts_here
our_code_starts_here:
  mov rax, 131
  ret
```

Notice how it both evaluates our program and writes to `test.s`.

- Note: Locally on arm macs, make sure to target the correct architecture for dynasm also with `--target x86_64-apple-darwin`.


### Ignoring or Changing the Starter Code

You can change a lot of what we describe above; it's a (relatively strong)
suggestion, not a requirement. You might have different ideas for how to
organize your code or represent things. That's a good thing! What we've shown
in class and this writeup is far from the only way to implement a compiler.

To ease the burden of grading, we ask that you keep the following in mind: we
will grade your submission (in part) by copying our own `tests/` directory in
place of the one you submit and running with all 3 flags:
```
cargo test -- -c tests/test1.snek tests/test1.s
cargo test -- -e tests/test1.snek
cargo test -- -g tests/test1.snek tests/test1.s
```
And producing corresponding executable files and outputs from generated assembly.
It _doesn't_ rely on any of the data definitions or function signatures in
`src/main.rs`. So with that constraint in mind, feel free to make new
architectural decisions yourself.

## Strategies, and FAQ

**Working Incrementally**

If you are struggling to get started, here are a few ideas:

- Try to tackle features one at a time. For example, you might completely
ignore let expressions at first, and just work on addition and numbers to
start. Then you can work into subtraction, multiplication, and so on.
- Some features can be broken down further. For example, the let expressions
in this assignment differ from the ones in class by having multiple variables
allowed per let expression. However, you can first implement let for just a
single variable (which will look quite a bit like what we did in class!) and
then extend it for multiple bindings.
- Use git! Whenver you're in a working state with some working tests, make a
commit and leave a message for yourself. That way you can get back to a good
working state later if you end up stuck.
- For AOT compilation, first check if your generated `.s` assembly is equivalent to the `Instr` vector, and for JIT, check if your `Instr` instructions are pattern matched correctly. Make sure you call the dynasm conventions from Adder correctly.

**FAQ**

**What should `(let ((x 5) (z x)) z)` produce?**

From the PA writeup: “Let bindings should evaluate all the binding expressions to values one by one, and after each, store a mapping from the given name to the corresponding value in both (a) the rest of the bindings, and (b) the body of the let expression. Identifiers evaluate to whatever their current stored value is.”

**Do Boa programs always have the extra parentheses around the bindings?**

In Boa, there's always the extra set of parens around the list.

**Can we write additional helper functions?**

Yes.


**Do we care about the text return from panic?**

Absolutely. Any time you write software you should strive to write thoughtful error messages. They will help you while debugging, you when you make a mistake coming back to your code later, and anyone else who uses your code.

As for the autograder, we expect you to catch parsing and compilation errors. For parsing errors you should `panic!` an error message containing the word `Invalid`. For compilation errors, you should catch duplicate binding and unbound variable identifier errors and `panic!` `Duplicate binding` and `Unbound variable identifier {identifier}` respectively. We've also added these instructions to the PA writeup.

**How should we check that identifiers are valid according to the description in the writeup?**

From the PA writeup: “You can **assume** that an id is a valid string of form `[a-zA-z][a-zA-Z0-9]*`. You will, however, ...”

**Assume** means that we're not expecting you to check this for the purposes of the assignment (though you're welcome to if you like).

**What should the program `()` compile to?**

Is `()` a Boa program (does it match the grammar)? What should the compiler do with a program that doesn't match the grammar?

**What's the best way to test? What is test case <some-test-name-from-autograder> testing?**

A few suggestions:

- First, make sure to test all the different expressions as a baseline
- Then, look at the grammar. There are lots of places where `<expr>` appears. In each of those positions, _any other expression_ could appear. So `let` can appear inside `+` and vice versa, and in the binding position of let, and so on. Make sure you've tested enough _nested expressions_ to be confident that each expression works no matter the context
- Names of variables are interesting – the names can appear in different places and have different meanings depending on the structure of let. Make sure that you've tried different combinations of `let` naming and uses of variables.

**My tests non-deterministically fail sometimes**

You are probably running `cargo test` instead of `cargo test -- --test-threads 1`. The testing infrastructure interacts with the file system in a way that can cause data races when
tests run in parallel. Limiting the number of threads to 1 will probably fix the issue.

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

Happy hacking!
