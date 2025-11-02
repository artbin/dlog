# EBNF Style Guide (ISO/IEC 14977)

## Terminals and metasymbols
- Use double-quoted terminals only: `"..."`.
- Allowed metasymbols: `, | ; ( ) [ ] { }`.
- Non-grammar prose and lexer notes must be inside special sequences `? ... ?`.

## Strings and escapes (canonical)
- `string = '"' , { string_char } , '"' ;`
- `string_char = escape | ? any char except " and \\ ? ;`
- `escape = "\\" , ( "\\" | '"' | "n" | "t" | "r" ) ;`

## Comments
- Line comments: `#` consuming up to `"\n" | "\r\n" | ? EOF ?`.
- Nested block comments (when enabled): `#| ... |#`.

## Ranges and sets
- Do not use Unicode ellipses `…`. Expand ranges explicitly or describe in `? ... ?`.
- Enumerate operator characters (`opchar`) as quoted terminals.

## Indentation (offside)
- Describe NEWLINE/INDENT/DEDENT token emission in prose `? ... ?`; keep grammar portable.

## File guidance
- Place a short “Depends on” section and “Preserves Homoiconicity” note at top of each doc.
- Add explicit anchors for sections referenced by other docs.
- Provide desugaring tables (Surface → S-expressions) for surface features.

## Examples
- Keep examples ASCII and copyable; prefer small before/after blocks.
- Pair surface `.surface` with canonical `.sexpr` files where applicable.
