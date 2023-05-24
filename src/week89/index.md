![forest-flame](./forest-flame.jpg)

# Week 8-9: Forest Flame, Due Thursday, June 1st (Closed Collaboration)

In this assignment you'll implement garbage collection for a language called
[Forest Flame], which uses our design for heap allocation.

[Forest Flame]: https://en.wikipedia.org/wiki/Oxyrhopus_petolarius

## Setup

For this assignment, you will (as in previous assignments) submit both a
compiler and a runtime.

Since garbage collection is a runtime feature, we provide a working
Forest Flame compiler for you: ([github classroom], [public starter code]).
If you use the starter code, you'll only have to modify the runtime.
However, feel free to instead update your own Egg-Eater compiler to match the Forest
Flame spec.

If you are participating in the Rust error study, please make sure you have the
`build.rs` and `config.txt` files in the new repository and set a new `project`
value in `config.txt` if you are using the starter code.

[github classroom]: FILL
[public starter code]: https://github.com/ucsd-compilers-s23/forest-flame-starter

## The Forest Flame Language

The Forest Flame language extends Diamondback with heap allocation and garbage
collection.

### Concrete Syntax

```
<prog> := <defn>* <expr>
<defn> := (fun (<name> <name>*) <expr>)
<expr> :=
  | <number>
  | true
  | false
  | nil
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
  | (gc)
  | (vec <expr>*)
  | (make-vec <expr> <expr>) count thing
  | (vec-get <expr> <expr>)
  | (vec-set! <expr> <expr> <expr>)
  | (vec-len <expr>)
  | (<name> <expr>*)
optionally:
  | (snek-<name> <expr>*)

<op1> := add1 | sub1 | isnum | isbool | isvec | print
<op2> := + | - | * | / | < | > | >= | <= | =

<binding> := (<identifier> <expr>)
```

The new pieces of syntax are `nil`, `gc`, `vec`, `make-vec`, `vec-get`,
`vec-set!`, `vec-len`, `isvec`, and `/` (division).

Additionally, it allows for any implementation-defined extra operations that
start with `snek-`,
which compilers for Forest Flame may implement or not as they like.
The starter code implements a `(snek-printstack)` operation.

### Semantics

Forest Flame adds the runtime type of *vectors*.
A vector is either `nil` or a heap-allocated list of zero or more elements.

It adds these new syntax constructs:

 - `nil` evaluates to the `nil` vector.

 - `(gc)` forces the garbage collector to run, and returns 0.

 - `(vec arg1 ... argN)` allocates a new vector on the heap of size `N` with
   contents `[arg1, ..., argN]`.

 - `(make-vec count value)` allocates a new vector on the heap of size `count`
   with contents `[value, value, value, ...]`.

   It gives a runtime error if `count` does not evaluate to a number, or if it
   evaluates to a negative number.

 - `(vec-get vec index)` gets the `index`th component of `vec`.

   It gives a runtime error if `vec` does not evaluate to a non-nil vector or if
   `index` does not evaluate to a valid (0-based) index into the vector.

 - `(vec-set! vec index value)` sets the `index`th component of `vec` to
   `value` and returns `vec`.

   It gives a runtime error if `vec` does not evaluate to a non-nil vector or if
   `index` does not evaluate to a valid (0-based) index into the vector.

 - `(vec-len vec)` returns number of items of `vec`.

   It gives a runtime error if `vec` does not evaluate to a non-nil vector.

 - `(isvec value)` returns `true` if `value` is a vector (possibly nil) and
   `false` otherwise.

 - `(/ x y)` implements division and gives a runtime error if the denominator
   `y` is zero.

 - `(snek-<name> args...)`: the specification and behavior of operations
   beginning with `snek-` is implementation-defined.
   This means compilers can do whatever they want with these operations.

   The motivation is to make debugging your GC easier: feel free to add whatever
   built-in debugging operations would be helpful. For example, the starter code
   compiler implements a `(snek-printstack)` operation.

 - `=` should implement reference equality for vectors.

 - Vectors are printed as comma-separated lists surrounded by square brackets,
   `nil` is printed as `nil`, and cyclic references should be printed as
   `[...]`.

   For example, the cyclic linked list containing `true` and `nil` would be
   printed as `[true, [nil, [...]]]`.

In addition, the compiled program now takes *two* arguments instead of just one.

 - The first argument is the input, which may be `true`, `false`, a number, or
   `nil` (new!). If no arguments are provided, the default input is `false`.

 - The second argument is the *heap size* in (8-byte) words, which must be a
   nonnegative number. If no second argument is provided, the default heap size
   is 10000.

During a program's execution, data on the heap is *live* if it reachable from
the program's stack.
If the program tries to allocate heap data but there is not enough space in the
heap, the garbage collector should run to compact the live 

During a program's execution, if heap space runs out, it runs the garbage
collector. If there is still not enough heap space, it exits with the error `out
of memory`.

### Examples

TODO FILL examples of programs and what they do

## Garbage collection

TODO (also all the subsections)

this section is about garbage collection and it says what the heap layout is and
it reminds you how mark-compact works.

blah blah gc

### Heap layout

A Forest Flame heap object has two metadata words, followed by the actual data.

 - First, there is a GC word, used to store the mark bit and the forwarding
   pointer during garbage collection. Outside of garbage collection, the GC word
   is always `0`.
 - Next, there is a word which stores the length of the vector. (Note that a
   vector of length `len` actually uses `len + 2` words, from the metadata.)
 - Next, there is each element of the vector, in order.

For example, the data `(vec false true 17)` stored at heap address `0x100` would
be represented by the value `0x101` and this heap data:

FILL image

As a running example, consider this program:

```scheme
(let ((x (vec false true 17))
      (y (vec 1 2)))
     (block
        (set! x (vec nil y nil))
        (set! y nil)
        (gc)))
```

At the start of collection, the heap looks like this:

FILL image

The stack contains the variables `x` and `y`. `x` has value `0x149` = `C` and
`y` is `nil`, so the root set is {`C`}.

### Marking

The first step of mark-compact is marking. We mark a heap object by setting its
mark bit, the lowest bit of the GC word. Marking does a graph traversal of the
heap, starting from the roots found on the stack.

Here's what the heap looks like after marking:

FILL image

### Compacting

The second step of mark-compact is compacting. Compacting has three parts:

 1. Computing forwarding locations
 2. Updating references
 3. Moving objects

#### Compacting 1: compute forwarding addresses

this section states that it's a linear scan through the heap, and shows the
result

#### Compacting 2: update references

this section states that it's a linear scan through *both* the stack and the heap,
and shows the result

#### Compacting 3: move the objects

this section shows the end result of moving all the objects

## Starter code

TODO

this section talks about Nico's compiler's interface for what the stack looks
like, etc. Will appreciate Nico's help (are we using the frame pointer to walk
the stack?) at the very least to proofread it

## Submission and grading

Submit via Gradescope. (TODO anything else?)

## Extension: simple generational GC

As discussed in lecture, nearly all modern garbage collectors take advantage of
the high infant mortality rate for heap allocations, by segregating the heap
into multiple *generations*, each of which stores data of a particular age, and
processing the older generations less frequently than the younger generations.

In this extension, you'll implement this idea by adding a **nursery** to your
GC.

 - The nursery should be around 10% of the size of the old space (the main
   mark-compact heap).
 - Allocations go into the nursery (i.e., `r15` points into here); when nursery
   space runs out, this triggers a **minor collection**.
   (Large objects which don't fit into the nursery should be allocated directly
   to the old space.)
 - A minor collection *evacuates* the nursery, reallocating all its live objects
   into the old space (using the copying GC algorithm). When main heap space
   runs out, this triggers a **major collection**.
 - A major collection collects both the nursery and the old space, compacting
   both the old space and the nursery into the old space.

**Write barriers and the remembered set.**
What root set do we use for a minor collection? Pointers into the nursery can
come from two sources: the stack, or the old space. The stack is typically
quite small, but we want to avoid traversing the whole old space -- after all,
this is what makes a minor collection fast.

So in order to find roots from the old space, we'll make the program keep a log
of which old-space objects contain pointers into the nursery. This has a runtime
component and a compiler component:

 - At run time, we need a physical datastructure (usually called the *remembered
   set*) to record which old-space objects may contain pointers into the
   nursery. A good option here is an array of pointers to old-space objects,
   together with a header flag in each old-space object saying whether it's
   already in the remembered set. When the array fills up, you could either
   resize it, or reset it by triggering a minor collection.
 - Before writing to the heap in `vec-set!`, we need to check if it's putting a
   pointer into the nursery into the old space, and if so, update the remembered
   set accordingly. This requires compiler support. (In general, any kind of
   GC-related check that happens during memory writes is called a *write
   barrier*.)
