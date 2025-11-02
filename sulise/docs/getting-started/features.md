# Features

> Docs status: Unreleased (updated 2025-10-15). See [CHANGELOG](../../CHANGELOG.md#2025-10-15).

This document summarizes the grammar features and surface options available in this repository. For detailed EBNF, examples, desugaring, dependencies, and status, see `plan.md`, `ROADMAP.md`, `DIAGRAMS.md`, and `BACKLOG.md`.

## Profiles

- Profile A (S-expressions)
  - Canonical reader: lists, dotted pairs, vectors, atoms, quote/quasiquote/unquote.
  - Data/literal extensions: keywords and bar-escaped symbols, numbers (radix/exactness, separators), nested block comments, characters/bytevectors, maps/sets.
- Profile B (application + infix)
  - Right-associative application with fold-right; optional infix with no precedence, minimal precedence (+/−, */), prefix/postfix, and pipeline `|>`.
- Profile C (indentation)
  - Offside rule (NEWLINE/INDENT/DEDENT); variants for optional colons, single-line suites `->`, trailing-operator line join.

## Expressions

- Application (right-assoc) with fold-right desugaring to unary application.
- Infix options: non-associative single-level; minimal precedence (+/− vs */); prefix/postfix tiers.
- Pipeline `|>`: left-assoc threading; optional placeholder piping in plan.

## Indentation (offside rule)

- NEWLINE/INDENT/DEDENT produced by the lexer; suite forms without colons; option for colons and single-line suites.
- Trailing-operator join: suppress NEWLINE when physical line ends with operator characters.

## High-level constructions (selected)

- Modules and imports (S-expr and indentation forms), exports/re-exports.
- Function definitions (arrow return form, multi-clause variants in plan), defaults/named/variadics.
- Let/where bindings (head and tail sugar).
- Records/destructuring: tagged maps or ctor style; field access/update sugar.
- Pattern matching with or/as/guard enrichments; dict/array patterns; view patterns.
- Comprehensions: generators, guards, let/do clauses; parallel comprehensions (plan).
- Traits/protocols and implementations; associated types and deriving.
- Types and generics (Visual Basic–inspired `as` / `of`), aliases/unions/intersections, ADTs/newtype.

## Errors, Option, and Async

- Zig-style error handling: `try` to propagate, `catch` to handle, `error(.Tag[, payload])` to construct.
- Option generic `(of Option T)`: `some`/`none`; optional `T?` type sugar.
- Result/Option sugar: coalesce `??`, postfix `?` for try, if-some binding; type sugar `T!` = `Result of T`.
- Async/await: `await`, `await within ms`, cancellation; integration with errors via `await?` and `catch` forms.

## Ergonomics (selected)

- String interpolation `i"...${expr}..."` → `(format ...)`.
- Pipeline extras: numbered placeholders, compose `>>`, tap `|>>`, into-binding.
- Records: lenses and nested updates with `(get)/(set)/(update)`.
- Literals: raw/byte/heredoc strings; date/time; duration/size; path; BigInt/Rational.
- Control: do-notation (Result/Option/Task), do-while, foreach with index, labeled blocks with value break.
- Misc: property tests, FSMs, effects/handlers, FFI, conditional compilation, const-eval.

## Homoiconicity (contract)

- Canonical program representation is S-expressions; all surface syntaxes desugar to S-exprs.
- Desugaring order: parse (chosen profile) → desugar → macroexpand (hygienic) → evaluate (S-exprs only).
- Quasiquote/unquote/unquote-splicing apply to S-exprs; read/print round-trips S-exprs (surface not guaranteed to round-trip textually).

## ISO EBNF compliance (shape)

- Strict metasymbols and double-quoted terminals; prose inside `? ... ?`.
- Expanded `letter`/`digit` ranges; normalized string/escape and whitespace/comment rules.
- Line comments use `#` (consume to newline/EOF); optional nested block comments `#| ... |#`.

## Navigation

- Core EBNF files: see `README.md` (File index) and `profiles.md` (Profiles A/B/C).
- Plans and status: `plan.md` (tasks, dependencies, mini‑TOCs, anchors, tag index), `ROADMAP.md`, `BACKLOG.md`.
- Diagrams: `DIAGRAMS.md`.
 - Feature matrix: `tables/features-matrix.md`.
 - Operators table: `tables/operators.md`.
 - Reference: `reference/grammar-index.md` (nonterminals), `reference/keywords-and-operators.md` (keywords/opchar).
