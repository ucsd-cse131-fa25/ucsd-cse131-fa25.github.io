---
layout: page
title: "Week 1 - A simple compiler in Rust"
doodle: "../doodle.png"
---

# Adder

In this assignment you'll implement a compiler for a small language called
Adder, that supports 32-bit integers and three operations – `add1`, `sub1`,
and `negate`.  There is no starter code for this assignment; you'll do it _from
scratch_ based on the instructions here.

## Setup

The necessary tools are installed on `ieng6.ucsd.edu`, and you should have a
course-specific account set up that you can access [via
ETS](https://sdacs.ucsd.edu/~icc/index.php). You can use standard remote access
tools like `ssh` or Visual Studio Code plugins like
[`sshfs`](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs)
to match your preferred working style.

You may also want to work on your own computer. You will need to install `rust`
and `cargo`:

[https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)

You may also (depending on your system) need to install
[`nasm`](https://www.nasm.us/). On OSX I used `brew install nasm`; on other
systems your package manager of choice likely has a version. On Windows you
should use [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/)

**The assignments assume that your computer can build and run x86-64-bit
binaries.  This is true of most (but not all) mass-market Windows and Linux
laptops. Newer Macs use a different ARM architecture, _but_ can also run legacy
x86-64-bit binaries, so those are fine as well.  You should ensure that
whatever you do to build your compiler also runs on `ieng6`, which is our
standard environment for testing these.**

The first few sections of the [Rust
Book](https://doc.rust-lang.org/book/ch01-00-getting-started.html) walk you
through installing Rust, as well. We'll assume you've gone through the
“Programming a Guessing Game” chapter of that book before you go on, so
writing and running a Rust program isn't too weird.

## The Adder Language

In each of the next several assignments, we'll introduce a language that we'll
implement.  We'll start small, and build up features incrementally.  We're
starting with Adder, which has just a few features –numbers and three
operations.

There are a few pieces that go into defining a language for us to compile:
* A description of the concrete syntax – the text the programmer writes.
* A description of the abstract syntax – how to express what the
  programmer wrote in a data structure our compiler uses.
* A *description of the behavior* of the abstract
  syntax, so our compiler knows what the code it generates should do.

### Concrete Syntax

The concrete syntax of Anaconda is:

```
<expr> :=
  | <number>
  | (add1 <expr>)
  | (sub1 <expr>)
  | (negate <expr>)
```

### Abstract Syntax

The abstract syntax of Anaconda is a Rust datatype, and corresponds nearly
one-to-one with the concrete syntax. We'll show just the parts for `add1` and
`sub1` in this tutorial, and leave it up to you to include `negate` to get
practice.

```
enum Expr {
  Num(i32),
  Add1(Box<Expr>),
  Sub1(Box<Expr>)
}
```

The `Box` type is necessary in Rust to create recursive types like these (see
[Enabling Recursive Types with
Boxes](https://doc.rust-lang.org/book/ch15-01-box.html#enabling-recursive-types-with-boxes)).
If you're familiar with C, it serves roughly the same role as introducing a
pointer type in a struct field to allow recursive fields in structs.

The reason this is necessary is that the Rust compiler calculates a size and
tracks the contents of each field in each variant of the `enum`. Since an
`Expr` could be an `Add1` that contains another `Add1` that contains another
`Add1`... and so on, there's no way to calculate the size of an enum variant like

```
  Add1(Expr)
```

(What error do you get if you try?)

Values of the `Box` type always have the size of a single reference (probably
represented as a 64-bit address on the systems we're using). The address will
refer to an `Expr` that has already been allocated somewhere.  `Box` is one of
several _smart pointer_ types whose memory are carefully, and automatically,
memory-managed by Rust.

### Semantics

A ``semantics'' describes the languages' behavior without giving all of the
assembly code for each instruction.

An Adder program always evaluates to a single i32.

- Numbers evaluate to
themselves (so a program just consisting of `Num(5)` should evaluate to the
integer `5`).
- `add1` and `sub1` expressions perform addition or subtraction by one on
their argument.
- `negate` produces the result of the argument multiplied by `-1`

There are several examples further down to make this concrete.

Here are some examples of Adder programs:

#### Example 1

**Concrete Syntax**

```scheme
(add1 (sub1 5))
```

**Abstract Syntax**

```rust
Add1(Sub1(Number(5)))
```

**Result**

```
5
```

#### Example 2

**Concrete Syntax**

```scheme
4
```

**Abstract Syntax**

```rust
Number(4)
```

**Result**

```
4
```

#### Example 3

**Concrete Syntax**

```scheme
(negate (add1 3))
```

**Abstract Syntax**

```rust
Negate(Add1(Number(3)))
```

**Result**

```
-4
```

## Implementing a Compiler for Adder

We're going to start by _just_ compiling numbers, so we can see how all the
infrastructure works. We *won't* give starter code for this so that you see how
to build this up from scratch.

By _just_ compiling numbers, we mean that we'll write a compiler for a language
where the “program” file just contains a single number, rather than
full-fledged programs with function definitions, loops, and so on. (We'll get
there!)

### Creating a Project

First, make a new project with

```
$ cargo new adder
```

This creates a new directory, called `adder`, set up to be a Rust project.

The main entry point is in `src/main.rs`, which is where we'll develop the
compiler. There's also a file called `Cargo.toml` that we'll use in a little
bit, and a few other directories related to building that we won't be too
concerned with in this assignment.

### The Runner

We'll start by just focusing on numbers.

It's useful to set up the goal of our compiler, which we'll come back to
repeatedly in this course:

> “Compiling” an expression means generating assembly instructions that
> evaluate it and leave the answer in the `rax` register.

Given this, before writing the compiler, it's useful to spend some time
thinking about how we'll run these assembly programs we're going to generate.
That is, what commands do we run at the command line in order to get from our
soon-to-be-generated assembly to a running program?

We're going to use a little Rust program to kick things off. It will look like
this; you can put this into a file called `runtime/start.rs`:

```
#[link(name = "our_code")]
extern {
    fn our_code_starts_here() -> i64;
}

fn main() {
  let i : i64 = unsafe {
    our_code_starts_here()
  };
  println!("{i}");
}
```

This file says:

- We're expecting there to be a precompiled file called `libour_code` that we
  can load and link against (we'll make it in a few steps)
- That file should define a global function called `our_code_starts_here`. It
  takes no arguments and returns a 64-bit integer.
- For the `main` of this Rust program, we will call the `our_code_starts_here`
  function in an `unsafe` block. It has to be in an `unsafe` block because Rust
  doesn't and cannot check that `our_code_starts_here` actually takes no
  arguments and returns an integer; it's trusting us, the programmer, to ensure
  that, which is `unsafe` from its point of view. The `unsafe` block lets us do
  some kinds of operations that would otherwise by compile errors in Rust.
- Then, print the result.

Let's next see how to build a `libour_code` file out of some x86-64 assembly
that will work with this file. Here's a simple assembly program that has a
global label for `our_code_starts_here` that has a “function body” that
returns the value `31`:

```
section .text
global _our_code_starts_here
_our_code_starts_here:
  mov rax, 31
  ret
```

Put this into a file called `test/31.s` if you like, to test things out (you
should now have a `runtime/` and a `test/` directory that you created).

We can create a standalone binary program that combines these with these
commands (substitute `macho64` for `elf64` on OSX):

```
$ nasm -f elf64 test/31.s -o runtime/our_code.o
$ ar rcs runtime/libour_code.a runtime/our_code.o
$ ls runtime
libour_code.a          our_code.o             start.rs
$ rustc -L runtime/ runtime/start.rs -o test/31.run
$ ./test/31.run
31
```

The first command _assembles_ the assembly code to an object file. The basic
work there is generating the machine instructions for each assembly
instruction, and enough information about labels like `our_code_starts_here` to
do later linking. The `ar` command takes this object file and puts it in a
standard format for dynamic library linking used by `#[link` in Rust. Then
`rustc` combines that `.a` file and `start.rs` into a single executable binary
that we named `31.run`.

We haven't written a compiler yet, but we _do_ know how to go from files
containing assembly code to runnable binaries with the help of `nasm` and
`rustc`. Our next task is going to be writing a program that generates assembly
files like these.

### Generating Assembly

Let's revisit our definition of compiling:

> “Compiling” an expression means generating assembly instructions that
> evaluate it and leave the answer in the `rax` register.

Since, for now, our programs are going to be single expressions (in fact just
single numbers), this means that for a program like “5”, we want to generate
assembly instructions that put the constant `5` into `rax`.

Let's write a Rust function that does that, with a simple `main` function that
shows it working on a single hardcoded input; this goes in `src/main.rs` and is
the start of our compiler:

```
/**
  Compile a source program into a string of x86-64 assembly
*/
fn compile(program : String) -> String {
  let num = program.trim().parse::<i32>().unwrap();
  return format!("mov rax, {}", num);
}

fn main() {
  let program = "37";
  let compiled = compile(String::from(program));
  println!("{}", compiled);
}
```

You can compile and run this with `cargo run`:

```
$ cargo run
   Compiling adder v0.1.0 (/Users/joe/src/adder)
mov rax, 37
```

Really all I did here was look up the documentation in Rust about converting a
string to an integer and template the number into a `mov` command. The input
`37` is hardcoded, and to use the output like we did above, we'd need to
copy-paste the `mov` command into a larger assembly file with
`our_code_starts_here`, and so on.

Here's a more sophisticated `main` that takes two command-line arguments: a
source file to read and a target file to write the resulting assembly to. It
also puts the generated command into the template we designed for our generated
assembly:

```rust
fn _main() -> std::io::Result<()> {
  let args: Vec<String> = env::args().collect();

  let in_name = &args[1];
  let out_name = &args[2];

  let mut in_file = File::open(in_name)?;
  let mut in_contents = String::new();
  in_file.read_to_string(&mut in_contents)?;

  let result = compile(in_contents);

  let asm_program = format!("
section .text
global _our_code_starts_here
_our_code_starts_here:
  {}
  ret
", result);

  let mut out_file = File::create(out_name)?;
  out_file.write_all(asm_program.as_bytes())?;

  Ok(())

}
```

Since this now expects _files_ rather than hardcoded input, let's make a test
file in `test/37.snek` that just contains `37` as contents. Then we'll read the
“program” (still just a number) from `37.snek` and store the resulting
assembly in `37.s`. (`snek` is a meme-y spelling of snake, which is a theme of
the languages in this course.)

Then we can run our compiler with these command line arguments:

```
$ cat test/37.snek
37
$ cargo run -- test/37.snek test/37.s
$ cat test.367.s

section .text
global _our_code_starts_here
_our_code_starts_here:
  mov rax, 37
  ret
```

Then we can use the same sequence of commands from before to run the program:

```
$ nasm -f elf64 test/37.s -o runtime/our_code.o
$ ar rcs runtime/libour_code.a runtime/our_code.o
$ rustc -L runtime/ runtime/start.rs -o test/37.run
$ ./test/37.run
37
```

We're close to saying we've credibly built a “compiler”, in that we've taken
some source program and gone all the way to a generated binary.

The next steps will be to clean up the clumsiness of running 3 post-processing
commands (`nasm`, `ar`, and `rustc`), and then adding some nontrivial
functionality.

#### Cleaning up with a Makefile

There are a lot of thing we could do to try and assemble and run the program,
and we'll discuss some later in the course. For now, we'll simply tidy up our
workflow by creating a Makefile that runs through the compile-assemble-link
steps for us. Put these rules into a file called `Makefile` in the root of the
repository:

```
test/%.s: test/%.snek src/main.rs
	cargo run -- $< test/$*.s

test/%.run: test/%.s runtime/start.rs
	nasm -f macho64 test/$*.s -o runtime/our_code.o
	ar rcs runtime/libour_code.a runtime/our_code.o
	rustc -L runtime/ runtime/start.rs -o test/$*.run
```

And then you can run just `make test/<file>.run` to do the build steps:

```
$ make test/37.run
cargo run -- test/37.snek test/37.s
    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
     Running `target/x86_64-apple-darwin/debug/adder test/37.snek test/37.s`
nasm -f macho64 test/37.s -o runtime/our_code.o
ar rcs runtime/libour_code.a runtime/our_code.o
rustc -L runtime/ runtime/start.rs -o test/37.run
```

The `cargo run` command will re-run if the `.snek` file or the compiler
(`src/main.rs`) change, and the assemble-and-link commands will re-run if the
assembly (`.s` file) or the runtime (`runtime/start.rs`) change.

#### Adding Nontrivial Language Features
