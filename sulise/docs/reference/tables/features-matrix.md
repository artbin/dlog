# Feature Matrix (Profiles A/B/C)

<a id="features-matrix"></a>

- A: Core S-expressions (Profile A)
- B: Application + infix/prefix/postfix + pipeline (Profile B)
- C: Indentation/offside + variants (Profile C)

Status: Supported | Partial | Planned

| Feature | A | B | C | Notes |
|---|---|---|---|---|
| Core S-expressions (01) | Supported ([Profile A](../../specifications/profiles.md#profile-a)) | Supported (targets A) | Supported (targets A) | All surfaces desugar to A |
| Right-assoc application (02) | — | Supported ([02](../../specifications/grammar/02-app-right-assoc.md)) | Supported (via B semantics) | Application parses as fold-right |
| Infix base (03) | — | Supported ([03](../../specifications/grammar/03-infix.md)) | Supported | No precedence by default |
| Application tighter than infix (10) | — | Supported ([10](../../specifications/grammar/10-app-tighter-infix.md)) | Supported | Needed for mixing `f x + g y` |
| Prefix/postfix operators (11) | — | Supported ([11](../../specifications/grammar/11-prefix-postfix.md)) | Supported | Unary operators layer |
| Minimal precedence (12) | — | Supported ([12](../../specifications/grammar/12-minimal-precedence.md)) | Supported | Two tiers: `*/` > `+/-` |
| Pipeline operator `|>` (13) | — | Supported ([13](../../specifications/grammar/13-pipeline.md)) | Supported | Left-assoc; threads left as last arg |
| Indentation/offside (04) | — | Partial | Supported ([04](../../specifications/grammar/04-indentation.md)) | Lexer emits NEWLINE/INDENT/DEDENT |
| Optional colons (14) | — | — | Supported ([14](../../specifications/grammar/14-indent-colons.md)) | Indentation variant |
| Single-line suites (15) | — | — | Supported ([15](../../specifications/grammar/15-single-line-suites.md)) | Indentation variant |
| Trailing-operator join (16) | — | — | Supported ([16](../../specifications/grammar/16-trailing-join.md)) | Indentation variant |
| Keywords / escaped symbols (05) | Supported ([05](../../specifications/grammar/05-keywords.md)) | Supported | Supported | `:kw`, `|bar-escaped|` |
| Numbers: radix/exactness (06) | Supported ([06](../../specifications/grammar/06-numbers.md)) | Supported | Supported | `_` separators; optional sign |
| Block comments nested (07) | Supported ([07](../../specifications/grammar/07-block-comments.md)) | Supported | Supported | `#| ... |#` |
| Characters / bytevectors (08) | Supported ([08](../../specifications/grammar/08-characters-bytevectors.md)) | Supported | Supported | `#\\`, `#u8(...)` |
| Maps / sets (09) | Supported ([09](../../specifications/grammar/09-maps-sets.md)) | Supported | Supported | `{...}`, `#{...}` |
| Result/Option sugar (15 task) | Planned ([plan 15](../../development/planning/plan.md#15-syntax-sugar-for-result-and-option)) | Planned | Planned | `??`, postfix `?`, `T!` in types; see also [operators](operators.md#operators-table) |
| Option generic (14 task) | Planned ([plan 14](../../development/planning/plan.md#14-option-generic-optional-values)) | Planned | Planned | `Option of T`, `T?` (policy) |
| Async/await (10 task) | Planned ([plan 10](../../development/planning/plan.md#10-async--await)) | Planned | Planned | S-expr forms `(async)`, `(await)` |
| Errors (Zig-style) (6 task) | Planned ([plan 6](../../development/planning/plan.md#6-zig-style-error-handling)) | Planned | Planned | `try`, `catch`, `error(.Tag)` |

Anchors:
- This table: `#features-matrix`
- Operators table: see `tables/operators.md` → `#operators-table`

Notes:
- `—` means not applicable to that profile directly (e.g., A is core data, not surface).
- Profiles B/C depend on A for atoms/literals; all surface forms desugar to A.
