# Trailing-Operator Line Continuations (Implicit Join)

Extends `04-indentation-ebnf.md` with a lexer policy that suppresses `NEWLINE` when a physical line ends with an operator.

Depends on: `04-indentation-ebnf.md` (offside rule and NEWLINE policy)

```ebnf
(* Lexer sketch; integrate into 04's offside rule section *)

(* Recognize operator tail at end-of-line *)
opchar        = "!" | "$" | "%" | "&" | "*" | "+" | "-" | "/" | ":" | "<"
              | "=" | ">" | "?" | "@" | "^" | "|" | "~" ;

(* If a physical line ends with opchar (possibly after spaces/comments),
   suppress NEWLINE token (treat as hspace) and continue lexing the next line.
   Do not affect INDENT/DEDENT; they are computed only when a NEWLINE is emitted. *)
```

Examples:
- `x +` newline `y`  is treated as `x + y` (no `NEWLINE` in between).
- Works well with `03`/`12` expression grammars.

## Lexer Details

- Implement a lookback at end-of-line: after stripping trailing spaces and comments, if the last nontrivia token is `opchar`, do not emit `NEWLINE`.
- Inside `()` `[]` `{}`, you already suppress `NEWLINE` per 04; this rule complements that behavior.
- Consider allowing a backslash `\` as an explicit join marker as an alternative policy.

## Edge Cases

- Lines ending with `:` in indentation language are not operators; do not suppress `NEWLINE` unless `:` is in your `opchar` set intentionally.
- Avoid suppression if the operator is part of a literal (e.g., `"a+b"`).
