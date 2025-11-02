# Grammar Index (Nonterminals)

This index lists common nonterminals and points to their defining documents. Profiles indicate where the rules apply.

Profiles:
- A = S-expressions (core)
- B = Application + infix/prefix/postfix + pipeline
- C = Indentation/offside variants

| Nonterminal | File | Section/Anchor | Profiles | Notes |
|---|---|---|---|---|
| string | 01-sexpr-ebnf.md | string / string_char / escape | A, B, C | Canonical string/escape reused across files |
| string_char | 01-sexpr-ebnf.md | string_char | A, B, C | `? any char except " and \\ ?` |
| escape | 01-sexpr-ebnf.md | escape | A, B, C | `\\` followed by one of `\\`, `"`, `n`, `t`, `r` |
| comment | 01-sexpr-ebnf.md | comment | A, B, C | `#` to `"\n" | "\r\n" | ? EOF ?` |
| block_comment | 07-block-comments-ebnf.md | (nested) | A, B, C | `#| ... |#` when 07 enabled |
| symbol | 01-sexpr-ebnf.md | identifier/symbol | A, B, C | Core atom; see also keywords/escaped symbols (05) |
| :keyword | 05-keywords-and-escaped-symbols-ebnf.md | keywords | A, B, C | `:kw` literal symbols |
| |bar-escaped| | 05-keywords-and-escaped-symbols-ebnf.md | bar-escaped | A, B, C | `bar_char` escapes inside `| ... |` |
| number | 06-numbers-radix-exactness-ebnf.md | number | A, B, C | Radix/exactness prefixes; `_` separators |
| char | 08-characters-and-bytevectors-ebnf.md | character | A, B, C | `#\\` prefixed |
| bytevector | 08-characters-and-bytevectors-ebnf.md | bytevector | A, B, C | `#u8(...)` |
| map | 09-maps-and-sets-ebnf.md | map | A, B, C | `{ key : val }` |
| set | 09-maps-and-sets-ebnf.md | set | A, B, C | `#{ ... }` |
| list | 01-sexpr-ebnf.md | list | A, B, C | `( ... )`; dotted pairs supported |
| vector | 01-sexpr-ebnf.md | vector | A, B, C | `#( ... )` if present in 01 |
| expr | 03-infix-operators-ebnf.md | expression | B, C | Base infix layer over application |
| application | 02-right-assoc-app-ebnf.md | application | B, C | Fold-right application |
| pipeline | 13-pipeline-operator-ebnf.md | pipeline | B, C | `|>` threading |
| prefix/postfix | 11-prefix-postfix-ebnf.md | prefix/postfix | B, C | Unary tiers |
| precedence | 12-minimal-precedence-ebnf.md | precedence | B, C | Two tiers: `*/` > `+/-` |
| indentation tokens | 04-indentation-ebnf.md | NEWLINE/INDENT/DEDENT | C | Lexer-emitted tokens |
| header colon | 14-indentation-optional-colons-ebnf.md | optional `:` | C | Optional colon after headers |
| single-line suite | 15-single-line-suites-ebnf.md | `->` | C | One-line suites |
| trailing operator join | 16-trailing-operator-join-ebnf.md | line-join | C | Suppress NEWLINE after trailing operator |

Notes:
- Exact rule names may vary slightly per file; see each document for authoritative definitions.
- Indentation behavior is specified via prose/special sequences and requires lexer support.
