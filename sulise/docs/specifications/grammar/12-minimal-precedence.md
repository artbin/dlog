# Minimal Precedence Table EBNF (Add/Mul)

Adds a two-level precedence (`*`/`/` above `+`/`-`) on top of right-associative application.

Depends on: `03-infix-operators-ebnf.md` (operator/lexeme base)

```ebnf
(* Skippers as in 03 *)
_            = { whitespace | comment } ;

expr         = add ;

add          = mul , { _ , ( "+" | "-" ) , _ , mul } ;

mul          = appl , { _ , ( "*" | "/" ) , _ , appl } ;

appl         = atom , { _ , atom } ;         (* fold-right to application *)

atom         = identifier | number | string | boolean | "(" , _ , expr , _ , ")" ;
```

Desugaring:
- `a * b + c` ⇒ `(+ (* a b) c)` ⇒ `App(App(+, App(App(*, a), b)), c)`
- Application within `atom`/`appl` remains right-associative after folding.

## Examples

```
1 + 2 * 3          ; ⇒ (+ 1 (* 2 3))
f x * g y + h z    ; application tighter than infix within `mul`/`add`
(a + b) * c        ; parentheses override
```

## Integration Notes

- Keep only two tiers to avoid scope creep. If you later add power `^`, decide on associativity.
- Reuse the atom/app constructs from the core so function application remains right-associative.
