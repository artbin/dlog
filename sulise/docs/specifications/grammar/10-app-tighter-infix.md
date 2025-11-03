# Application Binds Tighter Than Infix EBNF

Extends `03-infix-operators-ebnf.md` so that application binds tighter than infix, allowing `f x + g y` to parse without parentheses.

Depends on: `03-infix-operators-ebnf.md` (operator/lexeme base)

```ebnf
(* Skippers as in 03 *)
_            = { whitespace | comment } ;

(* Expressions *)
expr         = infix ;

(* Infix: left-associative, single level for all operators *)
infix        = app , { _ , operator , _ , app } ;

(* Application: parse a nonempty list of rexpr, then fold-right post-parse *)
app          = rexpr , { _ , rexpr } ;

rexpr        = atom
             | "(" , _ , expr , _ , ")" ;

atom         = identifier | number | string | boolean ;

operator     = opchar , { opchar } ;
opchar       = "!" | "$" | "%" | "&" | "*" | "+" | "-" | "/" | ":" | "<"
             | "=" | ">" | "?" | "@" | "^" | "|" | "~" ;
```

Desugaring:
- Application: `a b c` ⇒ `a (b c)` (fold-right)
- Infix: `lhs op rhs` ⇒ `(op lhs rhs)`

## Examples

```
f x + g y        ; ⇒ (+ (f x) (g y))
map f xs + 1     ; requires parentheses if your `+` expects two args only
(f x) * (g (h y))
```

## Integration Notes

- Use the same `identifier/number/string/boolean` lexemes as in 03 to stay consistent.
- Keep operator set aligned with 03; you can narrow/expand `opchar` as needed.
- Parsing strategy: read a nonempty sequence of `rexpr` for `app`, then fold-right.
