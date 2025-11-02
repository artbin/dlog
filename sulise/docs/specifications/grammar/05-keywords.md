# Keywords and Bar-Escaped Symbols EBNF

Extends `01-sexpr-ebnf.md` by adding `:keywords` and `|bar-escaped|` symbols while keeping existing `symbol_initial`/`symbol_subsequent` rules.

Depends on: `01-sexpr-ebnf.md` (base S-expression grammar)

```ebnf
(* Extend atom/symbol to support keywords and bar-escaped symbols *)

atom          = boolean | number | string | keyword | symbol ;

keyword       = ":" , plain_symbol ;        (* :foo, :max-depth *)

symbol        = bar_symbol | plain_symbol ;

bar_symbol    = "|" , { bar_char } , "|" ;  (* allow spaces/punct/case *)
bar_char      = sym_escape | ? any char except | and \ ? ;
sym_escape    = "\" , ( "\\" | "|" | "n" | "t" | "r" | "x" , hex , { hex } ) ;

plain_symbol  = symbol_initial , { symbol_subsequent } ;   (* from 01 *)

hex           = digit | "a" | "b" | "c" | "d" | "e" | "f"
              | "A" | "B" | "C" | "D" | "E" | "F" ;
```

Notes:
- Keeps backward compatibility with existing unescaped symbols.
- `sym_escape` allows hex escapes inside bar-escaped symbols.

## Rationale

- Keywords (`:name`) are ubiquitous in many Lisp families for lightweight, self-evaluating identifiers.
- Bar-escaped symbols allow spaces, punctuation, and case sensitivity when needed.

## Examples

```
(:ok :max-depth 3)
(set! |Case-Sensitive| 1)
(call |has space| :Π :file-name |a:b|)
|\x41|               ; bar-escaped with hex A inside (becomes "A")
||                    ; empty symbol (allowed)
```

## Integration Notes

- `keyword` stays in `atom` alongside `symbol` and retains normal reader semantics.
- If your dialect distinguishes packages/namespaces (e.g., `cl:car`), keep that as a separate extension.
- Bar-escaped symbols preserve exact spelling; consider whether your symbol table is case-sensitive or case-folding.

## Edge Cases

- Empty bar symbol `||` is permitted and denotes the empty symbol.
- Inside `bar_symbol`, `\x..` escapes insert a code point; validate hex length/limits in the lexer.
- Disallow unescaped `|` and `\` within `bar_symbol`.
