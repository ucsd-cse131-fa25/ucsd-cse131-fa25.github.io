![caduceus](./caduceus.png)

## Week 4: Caduceus, Due Wednesday, May 3, 10pm

In this assignment, you'll spend time reflecting on, and learning from, the
designs you and others chose for PA4, Cobra.

We will make PA4 compilers available to the class. We'll make the
review assignments by the end of the day on Thursday, April 27. These are based on the Cobra implementatins uploaded to Gradescope:

Assignment Sheet:
<https://docs.google.com/spreadsheets/d/1JVrxUOXTJKvv6fQFgpC4W10ut7GpB0Sk3NdkCoXUO7o/edit?usp=sharing>

You will be assigned 2 other compilers to review.

Your feedback will be **shared with the class (including the author of the
compiler)**, so make sure to keep what you write professional and
constructive.

Note: You should be hopefully assigned a repo that isn't your own, but if I made a mistake, choose a different compiler to review and note this on the google form!

### Tracing the Compiler

For **each** of the compilers you are reviewing, choose an interesting program that runs
successfully on the compiler under review (e.g. they match the correct
behavior). Make sure that between both programs you chose, it altogether uses at least two of:

  - A loop that runs several times and terminates
  - At least 2 different binary operators
  - `input`
  - `set!` assignment to variables

For each program you chose, show at least _two_ relevant code snippets from the
compiler that are critical to its compilation. For example, you might
show the data structures used in the type checker, the code generation,
and the parsing for a particular expression. Only choose the same snippet
of code for both programs if it behaves in an interestingly different way
across the two.

For each code snippet, write a sentence of how it relates to different parts
of the program you're testing, or a cool way it might be implemented that you haven't thought of.

You can use the same two programs on both compilers if you think they
illustrate the behavior well.

This means you should have a total of **four** code snippets (two per
compiler, per two examples).

### Bugs, Missing Features, and Design Decisions

For **each** compiler you are reviewing, choose a program that has different
behavior than it should, which can be for either AOT, JIT, or REPL.

- If a key feature in the program isn't implemented, describe how you would
add it to the compiler (see below for how to do this).
- If it is implemented but produces an error, describe how you could fix the
error and make it produce the correct answer (see below for how to do
this)
- If it is implemented but produces the wrong answer, decide if you
think this was a reasonable design decision. Describe as appropriate:

  1. If you think producing this answer instead made certain parts of the
  compiler design simpler or easier than matching the spec, and identify how.
  2. If you think it's just a bug, and if so, how to fix it.
  3. If you think it's a better design decision than what was chosen for Cobra.
- If you think the compiler perfectly implements Cobra,
explain what you tested to reach this conclusion and why you are confident
that it does.

This means you should have a total of **two** examples that might create unexpected behavior from the specification. We recommend you choose `-g` or the REPL to catch bugs/errors/or differences between all aspects of the compiler, not just only AOT or JIT.

### Lessons and Advice

Answer the following questions:

1. Identify a decision made in this compiler that's different from yours.
Describe one way in which it's a **better** design decision than you made.
1. Identify a decision made in this compiler that's different from yours.
Describe one way in which it's a **worse** design decision than you made.
1. What's one improvement you'll make to your compiler based on seeing this
one?
1. What's one improvement you recommend this author makes to their compiler
based on reviewing it?

### Handin

You will submit a PDF, first with the pages for your first compiler review and then followed by pages for your second compiler review. Label the submission number of the compiler you reviewed before your review. Please start the review of the second compiler on a *new page*.
(We wish you could submit and label 2 pdfs but Gradescope doesn't allow that). 


