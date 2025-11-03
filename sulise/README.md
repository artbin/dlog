# Lisp / Surface Language EBNF Set

> Docs status: Unreleased (updated 2025-10-15). See [CHANGELOG](CHANGELOG.md#2025-10-15).

Quick links: [S-expr](docs/specifications/sexpr.md) · [Surface](docs/specifications/surface.md) · [Features](docs/getting-started/features.md) · [Profiles](docs/specifications/profiles.md) · [Plan](docs/development/planning/plan.md) · [Shortlist](docs/development/planning/plan.md#shortlist-next) · [Roadmap](docs/development/planning/roadmap.md) · [Diagrams](DIAGRAMS.md) · [Examples](examples/) · [Backlog](docs/development/planning/backlog.md) · [Changelog](CHANGELOG.md)

## Table of Contents

- [What is this?](#what-is-this)
- [Profile selector](#profile-selector)
- [Quickstart](#quickstart)
- [Features overview](#features-overview)
- [How to use this repo](#how-to-use-this-repo)
- [Composition guidance](#composition-guidance)
- [Precomposed profiles](#precomposed-profiles)
- [Grammar file index](#grammar-file-index)
- [Conventions](#conventions)
- [ISO EBNF conformance](#iso-ebnf-conformance)
- [Homoiconicity](#homoiconicity)
- [Reference implementation (Lipsi)](#reference-implementation-lipsi)
- [Project docs](#project-docs)
- [Search tips](#search-tips)
- [Examples](#examples)
- [Example profiles](#example-profiles)
- [Inspired by](#inspired-by)
- [License](#license)

Each file is intentionally small, self-contained, and includes “Depends on” notes so you can layer them safely.

## What is this?

This repo is a modular grammar toolkit for Lisp/S-expression and surface syntaxes (infix, pipelines, indentation). It provides:

- Canonical S-expression core (Profile A) that all surfaces desugar to (homoiconicity preserved).
- Optional surface conveniences (Profile B/C) with explicit, documented desugaring.
- A large catalog of high-level constructs as small, composable EBNF variants.

See `sexpr.md` (Core) and `surface.md` (Surface) for the Surface vs Core split and the desugaring contract.

## Profile selector

- Profile A (S-expr): read/write S-expressions; best for macro systems and data-first workflows.
- Profile B (infix/pipeline): application with optional infix, minimal precedence, prefix/postfix, and pipelines.
- Profile C (indentation): Python-style offside rule; optional colons and one-line suites.

See `profiles.md` for precomposed profiles and `FEATURES.md` for a high-level overview.

## Quickstart

1) Pick a profile (A/B/C) and skim `profiles.md`.
2) Parse a tiny snippet using that profile; conceptually desugar to S-exprs:

```text
Input (B):   f x + g y
Desugar:     (+ (f x) (g y))

Input (B):   data |> map f |> sum
Desugar:     (sum (map f data))

Input (C):
if x > 0 -> return x
Desugar:     (if (> x 0) (return x))
```

3) Use `plan.md` to explore high-level constructs and their EBNF/desugaring.

## How to use this repo

1) Pick a core
- 01-sexpr-ebnf.md: general Lisp S-expression reader grammar.
- 02-right-assoc-app-ebnf.md: minimal surface language with right-associative application.
- 03-infix-operators-ebnf.md: adds infix (no precedence by default) over an app-based core.

2) Choose a block structure (optional)
- 04-indentation-ebnf.md: Python-style offside rule (INDENT/DEDENT/NEWLINE tokens).

3) Add atoms and literals (optional)
- 05: keywords and bar-escaped symbols
- 06: numbers with radix/exactness and `_` separators (adds optional sign)
- 07: nested block comments (#| ... |#)
- 08: characters and bytevectors (`#\\`, `#u8(...)`)
- 09: maps and sets (`{...}`, `#{...}`)

4) Choose expression conveniences (optional)
- 10: application binds tighter than infix
- 11: prefix/postfix operators
- 12: minimal precedence (+/- vs */)
- 13: pipeline operator `|>` (threads left as last arg)

5) Indentation refinements (optional, extend 04)
- 14: optional colons after headers
- 15: single-line suites with `->`
- 16: trailing-operator line join (suppresses NEWLINE after line-ending operator)

6) Compose a profile (A/B/C)
- Start from `profiles.md` → Profile A/B/C anchors.
- A = pure S-expr; B = application+infix/pipeline; C = indentation blocks.
- Add optional atoms/literals (05–09) and expression/indentation variants (10–16) as needed.

7) Integrate with a lexer/parser
- Terminals are double-quoted; prose-only hints live in `? ... ?`.
- Emit `NEWLINE`/`INDENT`/`DEDENT` in the lexer for 04/14–16; the grammar treats them as tokens.
- Line comments are `#` and consume to `"\n" | "\r\n" | ? EOF ?`. Enable `#| ... |#` only if 07 is included.
- Desugar after parse: right-assoc application (`a b c` ⇒ `a (b c)`), infix `lhs op rhs` ⇒ `(op lhs rhs)`, pipeline `x |> f` ⇒ `(f x)`.

8) Validate ISO shape (docs and grammars)
- No Unicode ellipses; use ASCII and expand ranges explicitly.
- Use `? ... ?` for EOF/newline and other lexical commentary.
- Reuse the normalized string/escape and comment rules from 01/05/06.

9) Try examples
- See `profiles.md` profile examples and the Quickstart above.
- Browse `plan.md` for feature-specific examples and desugaring notes.

## Composition guidance

- Application fold-right: Wherever `app` is parsed as a list of operands, desugar post-parse into right-nested unary application: `a b c` ⇒ `a (b c)`.
- Infix desugaring: `lhs op rhs` ⇒ `(op lhs rhs)` unless a precedence/associativity variant says otherwise.
- Mixing infix and application: If you want `f x + g y` without parentheses, use 10 (application tighter than infix). Otherwise keep 03’s no-precedence rule and require parentheses.
- Indentation: 04 defines how a lexer should emit `NEWLINE`, `INDENT`, `DEDENT`. Variants 14–16 only adjust header sugar, single-line suites, or line-join behavior.
- Validation: Some rules (e.g., numeric base digit sets, byte range 0..255, nested comments) are best enforced in the lexer/scanner rather than pure grammar.

## Grammar file index

- 01-sexpr-ebnf.md — Base S-expressions: lists, dotted pairs, vectors, atoms, quotes.
- 02-right-assoc-app-ebnf.md — Right-associative application-only surface.
- 03-infix-operators-ebnf.md — Infix layer (no precedence by default) over application.
- 04-indentation-ebnf.md — Python-style indentation (no colons) with NEWLINE/INDENT/DEDENT.
- 05-keywords-and-escaped-symbols-ebnf.md — Adds `:keywords` and `|bar-escaped|` symbols. Depends on 01.
- 06-numbers-radix-exactness-ebnf.md — Radix/exactness and digit separators. Depends on 01.
- 07-block-comments-ebnf.md — Nested `#| ... |#` comments. Depends on 01.
- 08-characters-and-bytevectors-ebnf.md — `#\\` characters and `#u8(...)` bytevectors. Depends on 01.
- 09-maps-and-sets-ebnf.md — `{ key : val }` maps and `#{ ... }` sets. Depends on 01.
- 10-app-tighter-than-infix-ebnf.md — Application tighter than infix. Depends on 03.
- 11-prefix-postfix-ebnf.md — Prefix/postfix operators with simple precedence. Depends on 03.
- 12-minimal-precedence-ebnf.md — Two-tier precedence (+/- vs */). Depends on 03.
- 13-pipeline-operator-ebnf.md — `|>` pipeline operator. Depends on 03.
- 14-indentation-optional-colons-ebnf.md — Optional `:` after headers. Depends on 04.
- 15-single-line-suites-ebnf.md — One-line suites with `->`. Depends on 04.
- 16-trailing-operator-join-ebnf.md — NEWLINE suppression on trailing operator. Depends on 04.

## Precomposed profiles

- Lisp S-expression reader: see `profiles.md` → [Profile A](docs/specifications/profiles.md#profile-a)
- Application + infix surface: see `profiles.md` → [Profile B](docs/specifications/profiles.md#profile-b)
- Indentation blocks: see `profiles.md` → [Profile C](docs/specifications/profiles.md#profile-c)

## Features overview

See `FEATURES.md` for a concise summary of profiles, expressions, indentation, high‑level constructions, errors/option/async, ergonomics, homoiconicity, and ISO EBNF shape.

## Conventions

- EBNF style: uses `? ... ?` for prose/lexical descriptions, block `(* ... *)` comments, and `_` for intra-line trivia.
- Identifiers, numbers, strings, booleans: kept minimal; extend to your dialect needs.
- “Depends on” lines in each file indicate the expected base grammar.
- Line comments are `#` across all grammars; some variants may enable nested block comments `#| ... |#`.

## ISO EBNF conformance

These grammars follow ISO/IEC 14977 EBNF metasymbols (`, | ; ( ) [ ] { }`) and use double-quoted terminals.

- Ellipses have been expanded to explicit alternatives where present.
- Prose and out-of-band tokens (e.g., EOF, newline policies) appear only inside special sequences `? ... ?` or in comments.
- String/escape rules are normalized to ISO-friendly definitions.
 - Line comments use `#` and consume up to `"\n" | "\r\n" | ? EOF ?`.

Notes:
- Indentation tokens (`NEWLINE`, `INDENT`, `DEDENT`) are described as lexer behavior; semantics are specified in comments/special sequences and may require a scanner.
- Numeric base validation (e.g., hex digit sets) is documented and typically enforced at the lexer/validator.

## Homoiconicity

- Profile A (S-expressions) is homoiconic: programs are lists, symbols, numbers—the same data structures the language manipulates.
- Profiles B/C (infix, pipeline, indentation) are surface sugar that desugars to the S-expression core.
- Desugaring contract:
  - All B/C parses are transformed to canonical S-expressions before macro-expansion and evaluation.
  - Quasiquote/unquote/unquote-splicing apply to the S-expression representation.
  - Read/print round-trips S-expressions; surface sugar is not guaranteed to round-trip textually.

## Reference implementation (Lipsi)

- Purpose: validate the EBNF and desugaring rules by implementing a prototype reader/parser, indentation-aware lexer, desugar pass, and S-expression pretty-printer.
- Profiles: targets A (core), with surface front-ends for B (infix/pipeline) and C (indentation).
- Status: planned (docs-first); the `examples/` directory serves as fixtures. A link will be added here when the code is available.

## Project docs

- Plan: `plan.md` (tasks, dependencies, examples, grouped index)
- Roadmap: `ROADMAP.md` (milestones, implementation order)
- Diagrams: `DIAGRAMS.md` (evaluation and desugaring flows)
- Backlog: `BACKLOG.md` (prioritized next/near/medium/long-term work)
 - Features: `FEATURES.md` (feature summary and navigation)
 - Changelog: `CHANGELOG.md`
 - S-expressions: `sexpr.md` (core explainer and Surface vs Core)
 - Surface: `surface.md` (Surface syntax explainer)
  - Reference: `reference/grammar-index.md` (nonterminal index)
  - Reference: `reference/keywords-and-operators.md` (reserved words and operators)
  - Profiles: `PROFILES.md` (A/B/C landing)
  - Guides: `LEXER-GUIDE.md`, `PARSER-GUIDE.md`, `TESTING.md`, `RELEASE-CHECKLIST.md`
  - Deep dives/plans: `OPERATORS-DEEP-DIVE.md`, `TYPES-PLAN.md`, `ERRORS-PLAN.md`
  - Contributing: `CONTRIBUTING.md`, `STYLE-EBNF.md`, `docs-template-17.md`, `SEARCH-TIPS.md`
 - Shortlist (Next): see `plan.md` → [Shortlist](docs/development/planning/plan.md#shortlist-next)
 - **Crawler**: see `CRAWLER.md` for link-graph checks and reports
 - **Crawler Architecture**: see `CRAWLER-ARCHITECTURE.md`

## Search tips

- Use the Table of Contents and mini‑TOCs in `plan.md` to jump to sections quickly.
- Task anchors follow `#<num>-<slug>`; paste into the browser to deep‑link (e.g., `#29-integration-async-concurrency-x-zig-style-error-handling`).
- Browse by tag in `plan.md` (Profiles/Tags index) or use the feature matrix (see `plan-docs.md`).
- Diagrams in `DIAGRAMS.md` show parse → desugar flows; use them to locate relevant tasks.

## Examples

See `examples/` for runnable-style snippets. Highlights:

- Complex core and surface: `examples/complex.sexpr`, `examples/complex.surface`
- Indentation (complex): `examples/indentation-complex.surface` ↔ `examples/indentation-complex.sexpr`
- Operators/precedence: `examples/operators-precedence.surface` ↔ `examples/operators-precedence.sexpr`
- Pipelines/composition: `examples/pipeline-compose-2.surface` ↔ `examples/pipeline-compose-2.sexpr`
- Literals: `examples/literals.sexpr`, `examples/numbers-radix-mix.sexpr`

## Example profiles

- Lisp reader with rich literals: 01 + (05, 06, 07, 08, 09)
- Juxtaposition application only: 02
- Infix without precedence (parenthesize when mixing): 03
- Infix with minimal precedence and tight application: 03 + 10 + 12
- Indentation-based language with one-line suites and optional colons: 04 + 14 + 15 (+ 16 if you want trailing-operator joins)

## Inspired by

- Lisp family (Lisp, Scheme, Racket, Common Lisp, Clojure): S-expressions and homoiconicity as the canonical core.
- Python: offside rule (indentation), optional colons, and single-line suites.
- F#/Elixir: pipeline operator `|>` threading the left value as the last argument.
- ML/OCaml/Haskell: operator precedence/associativity conventions (adapted to minimal tiers in 12).
- Zig: explicit error handling patterns (error unions, `try`/`catch`), planned as extensions in `plan.md`.
- Visual Basic: `as`/`of`-style type annotations/generics (planned high-level constructions in `plan.md`).
- Scheme/RnRS and SRFIs: numeric prefixes (`#x`, `#b`, `#d`, `#e`), characters `#\\`, and bytevectors `#u8(...)`.
- Clojure: literal shapes for maps `{ ... }` and sets `#{ ... }`.
