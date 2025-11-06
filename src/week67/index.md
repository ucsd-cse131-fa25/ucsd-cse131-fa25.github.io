![egg-eater](./eastern-diamondback.jpg)

# Week 6: Eastern Diamondback

In this assignment you'll implement a _static type checker_ for Diamondback
(with a few extensions).

## Language

Eastern Diamondback has two additions atop regular Diamondback. The language is
extended with _annotated functions_ and _casts_:

```
<type> := Num | Bool | Nothing | Anything
<defn> := (fun (<name> (<name> : <type>)*) <expr>)
       | ... as before ...
<expr> := (cast <type> <expr>)
       | ... as before ...
<prog> := ... as before ...
<repl> := ... as before ...
```

### (Dynamic) Semantics

The dynamic semantics of `(fun (<name> (<name> : <type>)*) <expr>)` are
_exactly_ the same as the dynamic semantics of `(fun (<name> <name>*) <expr>)`.
That is, the dynamic semantics are the same as ignoring the types and compiling
the corresponding un-annotated function.

The dynamic semantics of `(cast <type> <expr>)` are:

- Evaluate `<expr>` to a value `v` then:
  - If `<type>` is `Num` and `v` is a number, evaluate to `v`
  - If `<type>` is `Num` and `v` is a boolean, error with a string containing `"bad cast"`
  - If `<type>` is `Bool` and `v` is a number, error with t a string containing `"bad cast"`
  - If `<type>` is `Bool` and `v` is a boolean, evaluate to `v`
  - If `<type>` is `Nothing`, error with a string containing `"bad cast"`
  - If `<type>` is `Anything`, evaluate to `v`

### (Static) Semantics

The static semantics—that is, the potential compiler errors—of Eastern
Diamondback are where most of its interesting behavior is found. This language
has a type-checked mode where many errors that would have been previously
reported dynamically are reported at compile time.

The following are the _type rules_ for the expressions in the language. There
are a few notational conventions:

- `Γ e : T` means “expression `e` has type `T` in environment Γ”
- `Γ e ! "S"` means “expression `e` has type error with message `S` in environment Γ”
- `Γ e ≤ T` means `Γ e : T'` and `T' ≤ T`
- `T1 ≤ T2` means “T1 is a subtype of T2”
- `≮` means “is not a subtype of”
- `Γ` is a type environment of the shape `[x1 : T1][x2 : T2]...`, and `Γ(x)` means “look up x in Γ”

```
Γ <number> : Num

Γ true : Bool

Γ false : Bool

Γ input : Anything

Γ x : T
  when Γ(x) = T
  
Γ (let ((x1 e1) (x2 e2) ...) e) : T
  when Γ e1 : T1, Γ[x1 : T1] e2 : T2, ...
  and Γ[x1 : T1][x2 : T2]... e : T

Γ (add1 e) : Num
  when Γ e ≤ Num

Γ (add1 e) ! "Expected number"
  when Γ e ≮ Num

Γ (op e1 e2) : Num
  when Γ e1 ≤ Num and Γ e2 ≤ Num
  and op is +, -, *

Γ (op e1 e2) !! "Expected number"
  when Γ e1 ≮ Num or Γ e2 ≮ Num
  and op is +, -, *

Γ (op e1 e2) : Bool
  when Γ e1 ≤ Num and Γ e2 ≤ Num
  and op is <, >, <=, >=

Γ (op e1 e2) ! "Expected number"
  when Γ e1 ≮ Num or Γ e2 ≮ Num
  and op is <, >, <=, >=

Γ (= e1 e2) : Bool
  when Γ e1 ≤ Num and Γ e2 ≤ Num

Γ (= e1 e2) : Bool
  when Γ e1 ≤ Bool and Γ e2 ≤ Bool

Γ (set! x e) : T
  when e : T
   and Γ(x) ≤ T
   
Γ (set! x e) ! "Invalid set!"
  when e : T1
   and Γ(x) = T2
   and T1 ≮ T2
   
Γ (if e1 e2 e3) : T1 ∪ T2
  when Γ e2 : T1
   and Γ e3 : T2
   and Γ e1 : Bool
 
Γ (block e1 e2 ... en) : Tn
  when Γ e1 : T1, Γ e2 : T2, ..., Γ en : Tn
 
Γ (loop e) : T1 ∪ T2 ∪ ... ∪ Tn
  when Γ1 e1 : T1, Γ2 e2 : T2, ... Γn en : Tn
   and e1, e2, ... en are (break e) subexpressions of e not nested in another break
   and Γ1, Γ2, ..., Γn are the environments for the corresponding expressions

Γ (break e) : Nothing
  when Γ e : T

Γ (f e1 e2 ...) : T
  when (fun (f x1 x2 ...) e) is defined (an unannotated function)
   and e1 : T1, e2 : T2, ...

Γ (f e1 e2 ...) : T
  when (fun (f (x1 : T1) (x2 : T2) ...) e) is defined
   and e1 ≤ T1, e2 ≤ T2, ...

Γ (cast T e) : T
  when Γ e : T'
```

These rules refer to union `∪` and subtyping `≤`, which are defined as:

```
∀ T . T ∪ Anything = Anything
∀ T . Anything ∪ T = Anything
∀ T . T ∪ T = T
∀ T . T ∪ Nothing = T
∀ T . Nothing ∪ T = T
Num ∪ Bool = Anything
Bool ∪ Num = Anything
```

```
∀ T . T ≤ T = true
∀ T . T ≤ Anything = true
∀ T . Nothing ≤ T = true
```

### Eastern Diamondback

Your task is to add a new _mode_ to your compiler for type checking.

Each of the existing options can have `t` added:

- `-tc <prog>.snek <prog>.s` should run the type checker, report any type errors as compile errors,
  and generate the compiled output if there were no compile errors
- `-tg <prog>.snek <prog>.s <input>` and `-te <prog>.snek <input>` should behave
  like `-e` and `-g`, except they should use the
  _provided value of `input`_ to calculate `input`'s type. That is they should
  use an updated rule for `input` that `Γ input : Num` and/or `Γ input : Bool`
  depending on what was provided
- `-ti` should behave like `-i`, but all REPL entries should be type-checked and
  only evaluated if they have no type errors
- Finally, `-t <prog>.snek` should _just_ typecheck the program and report
  errors (if any)
