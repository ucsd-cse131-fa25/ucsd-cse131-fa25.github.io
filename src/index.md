![doodle](./doodle.jpg)

# UCSD CSE131 Syllabus and Logistics

- [Joe Gibbs Politz](https://jpolitz.github.io) (Instructor)
  - OH: Tuesday 3pm and Wednesday 1pm, CSE 3206 (Joe's office)
- Shaurya Raswan (TA)
  - OH: Mondays at 12PM, B275 CSE Basement

[Basics](#basics) -
[Schedule](#schedule) -
[Staff &amp; Resources](#staff--resources) -
[Grading](#course-components-and-grading) -
[Policies](#policies)

In this course, we'll explore the implementation of **compilers**: programs
that transform source programs into other useful, executable forms. This will
include understanding syntax and its structure, checking for and representing
errors in programs, writing programs that generate code, and the interaction of
generated code with a runtime system.

We will explore these topics interactively in lecure, you will implement
an increasingly sophisticated series of compilers throughout the course to
learn how different language features are compiled, and you will think
through design challenges based on what you learn from implementation.

This web page serves as the main source of announcements and resources for the
course, as well as the syllabus.

## Basics

- Office Hours:
  - Joe: Tuesday 3pm and Wednesday 1pm, CSE 3206 (Joe's office)
  - Shaurya: Monday 12pm, also discussion Wednesday 12pm (CSE B275)
- Q&A Forum: [Piazza](https://piazza.com/class/mg11c8j2t3d7gi/)
- Textbook/readings: There's no official textbook, but we will link to
  different online resources for you to read to supplement lecture. Versions
  of this course have been taught at several universities, so sometimes I'll
  link to those instructors' materials as well. Some useful resources are:
  - [The Rust Book](https://doc.rust-lang.org/book/) (also [with embedded quizzes](https://rust-book.cs.brown.edu/))
  - [An Incremental Approach to Compiler Construction](http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf)
  - [UMich EECS483](https://maxsnew.com/teaching/eecs-483-fa22/)
  - [Northeastern CS4410](https://courses.ccs.neu.edu/cs4410/)

## Schedule

The schedule below outlines topics, due dates, and links to assignments. The
schedule of lecture topics might change slightly, but I post a general plan so
you can know roughly where we are headed.


### Week 1 - Rust and Source to Assembly Conversion

- [Assignment 1](./week1/index.md)
- [Repository (includes PDF of handout)](https://github.com/ucsd-cse131-fa25/2025-09-26-lecture)
- [Handout Wednesday](https://drive.google.com/file/d/1cRxv84Rcna0X3DVscHE8T_o292v5nM43/view?usp=sharing)
- Reading and resources:
  - [Rust Book Chapters 1-6](https://doc.rust-lang.org/book)
  - [x86-64 quick reference (Stanford)](https://web.stanford.edu/class/archive/cs/cs107/cs107.1196/guide/x86-64.html)
  - [x86-64 quick reference (Brown)](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)
  - [Max New on Adder](https://maxsnew.com/teaching/eecs-483-fa21/lec_let-and-stack_notes.html#%28part._incr._.Growing_the_language__adding__and_subtracting__1%29) _Max New and Ben Lerner have done a nice job writing up notes on some of my original scrawlings around this material. They don't use exactly the same style or make the same decisions I do in this class, so make sure you implement what's in **our** assignments, not what they wrote, but things are close enough to be useful._

### Week 0 – Welcome
  - [Friday In-class Notes](./notes/lecture1.pdf)
  - [Friday repository](https://github.com/ucsd-cse131-fa25/2025-09-26-lecture)
  - [Friday Handout](https://drive.google.com/file/d/1AOZ-MRYc1DYdbBlz6xkMrETaeCfHujZI/view?usp=share_link)
  - Reading and resources:
    - [Rust Book Chapters 1-6](https://doc.rust-lang.org/book)
    - [x86-64 quick reference (Stanford)](https://web.stanford.edu/class/archive/cs/cs107/cs107.1196/guide/x86-64.html)
    - [x86-64 quick reference (Brown)](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)

## Course Components and Grading

Your grade will be calculated from **engagement**, **assignments**, and **code
demos**.

- **Assignments** are given periodically, typically at one or two week intervals.
  On each you'll get a score from 0-4.

  Assignments will have a mix of written work and programming work. The written
  work will include things like:

  - describing and justifying _design decisions_ you made in your compiler
  - giving a description of implementation or tradeoffs in a potential design
    decision one could make
  - doing code review of other students' compilers

  We plan for 6 total assignments. There will be opportunities for later
  assignments to count towards credit for earlier assignments.

- **Code Demos** are the exams for the course. Twice during the quarter you
  will meet in-person with the instructor and/or a TA for a code review of a
  recent assignment. You will demonstrate your code running on examples we
  provide in the code review and walk us through how your code works on those
  examples. We will assign a 0-4 score and will also give feedback on what
  you'd need to fix to raise your score. More details about the promps and
  scheduling will be shared in advance of the reviews.

  In finals week, you will have a chance for a _second round code review_ for
  **one** of your in-quarter code reviews, which can raise your score by up to
  3 points (e.g. 0 to 3, 1 to 4, 2 to 4) on that review. In the second round
  review you will present your response to our feedback from the first review.

  This is also the **only** policy for make-ups for a missed code review during
  the quarter: scheduling a second round review in finals week. A missed review
  has a maxiumum score of 3 for the rescheduled review. Missing both in-quarter
  reviews puts a C ceiling on your grade (if you miss one you can still earn an
  A).

The standards for grade levels are below. You must achieve the thresholds for
*all* course elements to earn that letter grade.

- **A**:
  - Code review point total 7, or 8 (including second round reviews)
  - Assignment point total 20 or higher
- **B**:
  - Exam point total 5 or 6 (including second round reviews)
  - Assignment point total 16 or higher
- **C**
  - Exam point total 3 or 4
  - Assignment point total 12 or higher

+/- modifiers will be assigned around the boundaries of these categories
consistently across students, and will also take into consideration
participation in class and Piazza, as well as exceptional assignment or code
review performance.


## Policies

### Programming, AI, and Collaboration

We encourage you to attempt all programming projects yourself first. They are
meant to provide productive learning and to prepare you to present your
understanding in code reviews.

Of course, there is lots of valuable learning you can get from discussing the
assignments with us and with your peers, and **we encourage it after you've
made attempts yourself**. When you work together with anyone (including course
staff!), you should properly credit them. See the guidelines below for
crediting other students you work with.

Though we discourage you using it as a way to get started, write entire
functions, or set up your project, you're welcome to use AI tools to aid your
programming. In particular, it may be useful for learning new Rust idioms or
autocompleting “routine” parts of functions to save time. However, we both
(a) from experience don't think that AI agents are very good at writing the
core logic of the compilers in this class and (b) will be checking your
understanding of the code you wrote in code reviews. So you should be able to
understand, trace, debug, and update any code that you generate from an AI
tool.

In each assignment:

- Any code that you didn't write must be cited in the CREDITS.txt file that
  goes along with your submission, or in an inline comment next to the code you
  didn't write.
  - **Example:** On an open collaboration assignment, you and another student
    chat online about the solution, you figure out a particular helper method
    together. Put an inline comment next to the FOO method saying “This FOO
    method was developed in collaboration with Firstname Lastname”
  - **Example:** On an open collaboration assignment, a student posts the
    compilation strategy they used to handle a type of expression you were
    struggling with. Your CREDITS.txt could say “I used the code from the
    forum post at [link]”
  - **Example**: If a function or a substantial logical chunk of code was
    generated by ChatGPT, Copilot, or another LLM or AI system you could put an
    inline comment next to the code describing the prompt you used to get it if
    you do.
- Anyone you work with in-person must be noted in your CREDITS.txt
  - **Example:** You and another student sit next to each other in the lab, and
    point out mistakes and errors to one another as you work through the
    assignment. As a result, your solutions are substantially similar.  Your
    CREDITS.txt should say “I collaborated with Firstname Lastname to develop
    my solution.”
- Do not share publicly your entire repository of code or paste an entire
  solution into a message board. Keep snippets to reasonable, descriptive
  chunks of code; think a dozen lines or so to get the point across.
- You _cannot_ use whole solutions that you find online (though it's OK to
  copy-paste from Stack Overflow, tutorials etc, if you need help with Rust
  patterns, etc.) You shouldn't get assistance or code from students outside of
  this offering of the class. All the code that is handed in should be
  developed by you or someone in the class.

### Late Work

Late work is generally not accepted, because often we'll release partial or
full solutions immediately following the deadline for an assignment.
Opportunities for making up missed credit are given in other ways, and will be
described throughout the quarter.

### Regrades

Mistakes occur in grading. Once grades are posted for an assignment, we will
allow a short period for you to request a fix (announced along with grade
release). If you don't make a request in the given period, the grade you were
initially given is final.

### Exams

You should **not** discuss the details of your code reviews with anyone outside
the course staff until the whole class has received their grades for it.

### Laptop/Device Policy in Lecture

There are lots of great reasons to have a laptop, tablet, or phone open
during class. You might be taking notes, getting a photo of an important
moment on the board, trying out a program that we're developing together, and
so on. The main issue with screens and technology in the classroom isn't your
own distraction (which is your responsibility to manage), it's the
distraction of **other students**. Anyone sitting behind you cannot help but
have your screen in their field of view. Having distracting content on your
screen can really harm their learning experience.

With this in mind, the device policy for the course is that if you have a
screen open, you either:

- Have only content onscreen that's directly related to the current lecture.
- Have unrelated content open and **sit in one of the back two rows of the
room** to mitigate the effects on other students. I may remind you of this
policy if I notice you not following it in class. Note that I really don't
mind if you want to sit in the back and try to multi-task in various ways
while participating in lecture (I may not recommend it, but it's your time!)

### Professionalism

This is an upper division course, so everyone here has been in college for a
little bit. Indeed, you are likely closer to your post-college life than the
start of it.

Practicing being a professional should start now for you, if it hasn't already.
Some of the people in this course may be your colleagues in the near future!
In the context of this course, the goal is learning about (and building!)
compilers. So professionalism means making choices that are more likely to
increase your learning and the learning of those around you. Keep course
discussions on topics around compilers, don't demean or talk down to fellow
students, don't do things solely to show off or compare yourself to other
students, make space for others in discussions (and take space if you need to),
and so on.

UCSD has [Principles of Community](https://ucsd.edu/about/principles.html) that
apply across campus and in this class; they are a good baseline for principles
of professional behavior. Feel free to talk to me about things regarding the
class climate that could improve your learning.

