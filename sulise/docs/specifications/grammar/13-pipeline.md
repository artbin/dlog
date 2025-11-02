# Pipeline Operator EBNF (`|>`)

Adds a left-associative pipeline operator that threads the left value as the last argument of the right app.

Depends on: `03-infix-operators-ebnf.md` (expression base)

```ebnf
(* Skippers as in 03 *)
_            = { whitespace | comment } ;

expr         = pipeline ;

pipeline     = app , { _ , "|>" , _ , app } ;   (* left-associative *)

(* Application parsed as list then folded right post-parse *)
app          = rexpr , { _ , rexpr } ;

rexpr        = atom | "(" , _ , expr , _ , ")" ;

atom         = identifier | number | string | boolean ;
```

Desugaring:
- `a |> f`        ⇒ `f a`
- `a |> f x`      ⇒ `f x a`
- `a |> (g x y)`  ⇒ `g x y a`
- `a |> f |> g`   ⇒ `g (f a)`

## Examples

```
x |> f |> g           ; ⇒ g (f x)
data |> map f |> sum  ; ⇒ sum (map f data)
x |> (g y)            ; ⇒ g y x
```

## Integration Notes

- Parsing uses the same `app` as core; desugar pipeline after performing app fold-right.
- If you want placeholder-based piping (`x |> f(_)`), define `_` as a hole and reorder args accordingly.
