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

- `e : T` means “expression `e` has type `T`”
- `e ! "S"` means “expression `e` has type error with message `S`”
- `x : T` means “x has type T in the environment”
- `e ≤ T` means “`e` has a type that is a subtype of `T`”
- `T1 ≤ T2` means “T1 is a subtype of T2”
- `≮` means “is not a subtype of”

```
<number> : Num

true : Bool

false : Bool

input : Anything

x : T
  when x : T is in the environment
  
(let ((x1 e1) (x2 e2) ...) e) : T
  when e1 : T1, e2 : T2 with x1 ← T1, ...
  and e : T with x1 : T1, x2 : T2, ...

(add1 e) : Num
  when e ≤ Num

(add1 e) ! "Expected number"
  when e ≮ Num

(op e1 e2) : Num
  when e1 ≤ Num and e2 ≤ Num
  and op is +, -, *

(op e1 e2) !! "Expected number"
  when e1 ≮ Num or e2 ≮ Num
  and op is +, -, *

(op e1 e2) :: Bool
  when e1 :: Num and e2 :: Num
  and op is <, >, <=, >=

(op e1 e2) !! "Expected number"
  when e1 or e2 :: Bool or Anything
  and op is <, >, <=, >=

(= e1 e2) :: Bool
  when e1 :: Num and e2 :: Num

(= e1 e2) :: Bool
  when e1 :: Bool and e2 :: Bool

(= e1 e2) !! "Mismatched Types"
  when e1 :: T1 and e2 :: T2

(set! x e) :: T
  when e :: T
   and x :: T in the environment
   
(set! x e) !! "Invalid set!"
  when e :: T1
   and x :: T2 in the environment
   and T1 !< T2
   
(if e1 e2 e3) : T1 ∪ T2
  when e2 : T1
   and e3 : T2
   and e1 : Bool
 
(block e1 e2 ... en) : Tn
  when e1 : T1, e2 : T2, en : Tn
 
(loop e) : T1 ∪ T2 ∪ ... ∪ Tn
  when e1 : T1, e2 : T2, ... en : Tn
   and e1, e2, ... en are (break e) subexpressions of e not nested in another break

(break e) : Nothing

(f e1 e2 ...) : T
  when (fun (f x1 x2 ...) e) is defined (an unannotated function)
   and e1 : T1, e2 : T2, ...

(f e1 e2 ...) : T
  when (fun (f (x1 : T1) (x2 : T2) ...) e) is defined
   and e1 ≤ T1, e2 ≤ T2, ...
```




