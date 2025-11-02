# Maps and Sets EBNF

Adds Clojure-like map and set literals to `01-sexpr-ebnf.md`.

Depends on: `01-sexpr-ebnf.md` (base S-expression grammar)

```ebnf
(* Extend top-level sexpr to include map and set literals *)

sexpr         = atom | list | vector | map | set | quoted ;

map           = "{" , _ , [ pair , { _ , "," , _ , pair } ] , _ , "}" ;
pair          = sexpr , _ , ":" , _ , sexpr ;

set           = "#{" , _ , [ sexpr , { _ , sexpr } ] , _ , "}" ;
```

Notes:
- Comma in `map` is treated as a separator (optional to accept as whitespace in your reader).
- `set` uses `#{ ... }` to avoid ambiguity with lists/vectors.

## Rationale

- Literal maps/sets are convenient for configuration data and small constants.

## Examples

```
{:a 1, :b 2}
#{1 2 3 2}     ; duplicate `2` — decide whether to deduplicate at read-time
{ |sp key| :x, :y 42 }
```

## Semantics & Options

- Define key equality for `map` (pointer, `eq`, `equal`, structural?).
- Decide whether `set` removes duplicates during reading or at runtime.
- If you prefer pair lists without `:`, you can adopt `(key value)` pairs instead of `key : value`.
