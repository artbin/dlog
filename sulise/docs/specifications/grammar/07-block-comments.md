# Nested Block Comments EBNF

Extends the `comment` rule in `01-sexpr-ebnf.md` to support nested `#| ... |#` block comments.

Depends on: `01-sexpr-ebnf.md` (base S-expression grammar)

```ebnf
(* Line and block comments (nested) *)

comment       = "#" , { ? any char except newline ? } , ( "\n" | "\r\n" | ? EOF ? )
              | block_comment ;

block_comment = "#|" , { block_char | block_comment } , "|#" ;

block_char    = ? any char except the sequences "#|" and "|#" ? ;
```

Note:
- The recursion in `block_comment` permits arbitrary nesting depth.

## Rationale

- Multi-line, nestable comments are useful for temporarily disabling regions that already contain line comments or other block comments.

## Examples

```
#| outer start
   #| inner |# still commented
end |#
```

## Lexing Considerations

- Recognize the two-character delimiters `#|` and `|#`; they do not nest with strings—treat them only outside string literals.
- Newlines inside block comments are ignored; do not emit `NEWLINE`/indent tokens while inside a block comment.
- Prefer a manual scanner rather than regex for correct nesting.
