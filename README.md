Note
---

This is an university project which was good enough to score all points.
I have uploaded the repository with original code modifying only this README.

Rust-- (rustmm)
======

**Language interpreter written in Haskell.**

Language
---

In the first place was meant to be subset of the Rust language
(https://www.rust-lang.org/), but eventually it is a little bit different.
It is a mix of imperative language and functional language also including
closure like in JavaScript or Python which remembers their own scopes.
Features: expression based, functions as values (can be arguments and returned
from functions), closures which remember their environment, references,
variables mutability (during static check).

Parser
---

Written in MegaParsec (https://github.com/mrkkrp/megaparsec).

Static Checks
---

Types, mutability, taking references.

AST & Eval
---

I tried to keep AST minimalistic. All operators and while are desugarated
to calls to builtins. I don't support generics, so if is actually present
in AST since I need to check if both branches returns same type.

Building
---

With cabal. `cabal build` builds the app. Probably you'll need to run
`cabal install megaparsec`, maybe `cabal update`. You can also change the
requirements in rustmm.cabal file, to suit your version of base package or
installed megaparsec. I think megaparsec >=6.* should work.
Run with `cabal run <optionally-file-path-here>` or
`echo "program here" | cabal run`.

More details
---

Interpreter prints output after evaluation is done, so if program loops, then
it won't print anything.

Recursion works only with top definition functions. It does not work with
nested functions (you can take function as an arguments and pass this
function though). It is because I haven't instroduced new variant in AST,
just desugarated `fn x() {}` to `let x = <function>;`.

`src/Interpreter/Defs.hs`:
AST (Exp), Value, Type variants
also Interpreter and StaticChecker type definitions

I have used following monads:

* IO - in main for stdin/out/err
* Except - in Interpreter and StaticChecker for reporting errors
           (Interpreter reports ONLY runtime errors, here only division by 0).
* Reader - for mapping Var name -> location.
* State - Location -> Value for Interpreter and Loc -> Type for static checker.

Grammar - can be somehow read from `src/Interpreter/Parser.hs`.
Top defs must be functions not separated by anything.
Inside a block of code (i.e. function body) all expressions must be separated
by the semicolon ';', even function definitions and while and if.
Also, all expression but last in a block must return unit type!
Assignment expression returns unit.
Let is the only statement and isn't expression.

Some basic staff:

```rust
// defining a function
fn foo(x: int) -> bool {
    x < 5  // last expression is the returned value
           // note if you put semicolon at the end, it means returning unit type
}

// new binding
let x = 4;

// new binding x shadowing previous ones, this one allows to mutate it value
let mut x = 4;
x = 6;

// closure, note how we are marking mut arguments (also applies to function definitions)
let closure = |mut y: int| -> bool {
    let mut res = true;
    while y > 0 {
        y = y - 1;
        res = !res;
    };
    res
}

// calling function requires @ character:
let four = @|| -> int { 4 } ();

// references are tricky; the differences is like in C++ with pointers.
let mut v = 4;
let a = &x; // immutable binding, points to immutable var
let a = &mut x;  // points to mutable var
*a = 6;  // and that's how we can change value of pointed object
let mut a = &x;  // mutable binding, points to immutable var
let mut vv = 42;
a = &vv;  // changing to where references a points to
          // note: i don't support coercing &mut type to &type
          //       so the following fails:
          //       @|x: &int|{}(&mut someInt)
```

Note: code above is just a shortcut; you can't put it outside function body if
      you want to run it.
More in examples in `good/` and `bad/` directories.
