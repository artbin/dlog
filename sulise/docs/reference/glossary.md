# Glossary

- **Surface**: Any user-facing concrete syntax layered over the S-expression core. Desugars to core before macroexpand/eval. See `surface.md`.
- **Core**: Canonical S-expression representation (Profile A). Source of truth for macroexpansion and evaluation. See `sexpr.md`.
- **S-expressions (S-expr)**: Lists, symbols, numbers, strings, etc., used as the canonical AST. Homoiconic with programs.
- **Homoiconicity**: Property that code is represented by the same data structures it manipulates (here: S-expressions). Only the core is homoiconic. See `README.md` → Homoiconicity.
- **Desugaring**: Mapping from Surface text to canonical S-expressions. Contract: parse Surface → desugar → S-expr → macroexpand → eval.
- **Profiles (A/B/C)**: Precomposed grammar bundles in `profiles.md`.
  - **A**: Core S-expression reader.
  - **B**: Application + optional infix/prefix/postfix + pipeline `|>`.
  - **C**: Indentation (offside rule) with optional colons and one-line suites.
- **Offside rule (Indentation)**: Block structure via indentation; lexer emits `NEWLINE`/`INDENT`/`DEDENT` tokens (04/14–16).
- **Tokens for indentation**: `NEWLINE`, `INDENT`, `DEDENT` produced by the lexer; consumed by grammar in 04 and variants 14–16.
- **Line comment**: `#` to end of line; consumes `"\n" | "\r\n" | ? EOF ?`.
- **Block comment (nested)**: `#| ... |#` when variant 07 is enabled.
- **Keywords and escaped symbols**: `:keyword` and `|bar-escaped|` symbol forms (05).
- **Numbers (radix/exactness)**: Numeric literals with base/exactness prefixes and `_` separators (06).
- **Characters and bytevectors**: `#\\` character literals and `#u8(...)` bytevectors (08).
- **Maps and sets**: `{ k : v }` and `#{ ... }` literal forms (09).
- **Application (right-associative)**: Juxtaposition `a b c` parsed then desugared to `a (b c)`.
- **Infix operators**: `lhs op rhs` desugars to `(op lhs rhs)` unless precedence rules say otherwise (03, 10–12).
- **Precedence & associativity**: Operator tiering and grouping rules; minimal tiers in 12; application-tighter-than-infix in 10.
- **Prefix/Postfix**: Unary operators that bind per 11’s rules.
- **Pipeline (`|>`)**: Threads left value as last argument: `x |> f` → `(f x)` (13).
- **Result / Option generics**: `Result of T`, `Option of T` types used in plans; `T!` shorthand for `Result of T`.
- **Type annotations (VB-inspired)**: Use `as` and `of` for annotations and generics (e.g., `x as Int`, `List of T`).
- **Zig-style error handling**: Error unions and control (`try`, `catch`, `error`, `errdefer`) in plans/examples; integrates with types.
- **Macroexpansion**: Compile-time transformation over the S-expression core (after desugaring, before evaluation).
- **Round-trip**: Surface text may not round-trip; S-expression read/print is the round-trippable form.
- **ISO/IEC 14977 EBNF**: Standard EBNF used here. Terminals are double-quoted; metasymbols: `, | ; ( ) [ ] { }`.
- **Special sequence**: `? ... ?` encloses prose or lexer-oriented notes (e.g., EOF, newline) allowed by ISO EBNF.
- **Terminals vs nonterminals**: Terminals are quoted strings; nonterminals are rule names per ISO EBNF.
- **Metasymbols**: ISO EBNF punctuation: `, | ; ( ) [ ] { }` with their standard meanings.

See also:
- `profiles.md` (profiles A/B/C and examples)
- `FEATURES.md` (feature overview)
- `plan.md` (tasks and deep feature plans)
- `DIAGRAMS.md` (parse → desugar flows)
