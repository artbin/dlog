# Dashboard

A central landing page with quick links, status, and common commands.

## Table of Contents

- [Status](#status)
- [Quick links](#quick-links)
- [Tables](#tables)
- [References](#references)
- [Examples](#examples)
- [Fast actions](#fast-actions)
- [Shortlist (Next)](#shortlist-next)
- [Quality gates](#quality-gates)
- [Release readiness](#release-readiness)
- [Search tips](#search-tips)
- [Diagrams](#diagrams)
- [Anchors](#anchors)
- [Contribution](#contribution)
- [Recent changes](#recent-changes)

## Status

<!-- STATUS:BEGIN -->
## Status

- Docs status: Unreleased (updated 2025-10-16)
- Commit: 6250955
- Examples: 25 surface · 35 sexpr · 25 pairs
<!-- STATUS:END -->

Back to top · [TOC](#table-of-contents)

## Quick links

| Core | Planning |
|---|---|
| [README](README.md) | [Shortlist](docs/development/planning/plan.md#shortlist-next) |
| [Profiles (A/B/C)](docs/specifications/profiles.md#unified-ebnf-profiles) | [Plan](docs/development/planning/plan.md) |
| [Examples index](examples/README.md) | [Roadmap](docs/development/planning/roadmap.md) |
| [Tables](docs/reference/tables/features-matrix.md#features-matrix) · [Operators](docs/reference/tables/operators.md#operators-table) | [Backlog](docs/development/planning/backlog.md) |
| [References](docs/reference/grammar-index.md#grammar-index-nonterminals) | [Diagrams](DIAGRAMS.md) |
| [Anchors](docs/reference/anchors.md) | [Changelog](CHANGELOG.md#2025-10-15) |

Back to top · [TOC](#table-of-contents)

## Tables

- Feature matrix — [tables/features-matrix.md](docs/reference/tables/features-matrix.md#features-matrix)
- Operators table — [tables/operators.md](docs/reference/tables/operators.md#operators-table)

Back to top · [TOC](#table-of-contents)

## References

- Grammar index (nonterminals) — [reference/grammar-index.md](docs/reference/grammar-index.md#grammar-index-nonterminals)
- Keywords and operators — [reference/keywords-and-operators.md](docs/reference/keywords-operators.md#keywords-and-operators-reference)
- Profiles — [PROFILES.md](docs/getting-started/profiles-overview.md)
- Guides — [LEXER-GUIDE.md](docs/guides/lexer.md), [PARSER-GUIDE.md](docs/guides/parser.md), [TESTING.md](docs/development/testing.md), [RELEASE-CHECKLIST.md](docs/development/release-checklist.md)
- Deep dives / plans — [OPERATORS-DEEP-DIVE.md](docs/guides/operators.md), [TYPES-PLAN.md](docs/development/planning/types-plan.md), [ERRORS-PLAN.md](docs/development/planning/errors-plan.md)
- Contributing — [CONTRIBUTING.md](docs/development/contributing.md), [STYLE-EBNF.md](docs/specifications/style-ebnf.md), [docs-template-17.md](docs/development/docs-template-17.md), [SEARCH-TIPS.md](docs/guides/search-tips.md)

Back to top · [TOC](#table-of-contents)

## Examples

- Index — [examples/README.md](examples/README.md)

### By profile

- A (core S-expr):
  - `examples/complex.sexpr`, `examples/literals.sexpr`, `examples/numbers-radix-mix.sexpr`, `examples/nested-collections.sexpr`, `examples/vectors-and-dotted.sexpr`
- B (application/infix/pipeline):
  - `examples/pipeline-compose-2.surface` ↔ `.sexpr`, `examples/operators-precedence.surface` ↔ `.sexpr`, `examples/map-filter-reduce.surface` ↔ `.sexpr`, `examples/matrix-mul-precedence.surface` ↔ `.sexpr`, `examples/paren-precedence.surface` ↔ `.sexpr`
- C (indentation/offside):
  - `examples/indentation-blocks.surface` ↔ `.sexpr`, `examples/indentation-complex.surface` ↔ `.sexpr`, `examples/indent-singleline-colon.surface` ↔ `.sexpr`, `examples/multiline-join-infix-pipe.surface` ↔ `.sexpr`, `examples/trailing-join-2.surface` ↔ `.sexpr`

### By theme

- Pipelines: `pipeline-composed.*`, `pipeline-compose-2.*`, `pipeline-into-reduce.*`, `pipeline-nesting.*`, `pipeline-foldl.*`
- Precedence/infix: `operators-precedence.*`, `paren-precedence.*`, `paren-force-assoc-2.*`, `matrix-mul-precedence.*`
- Indentation: `indentation-blocks.*`, `indentation-complex.*`, `indent-singleline-colon.*`, `multiline-join-infix-pipe.*`, `trailing-join-2.*`
- Comments/strings: `block-comments.*`, `block-comments-inline-2.*`, `strings-escapes*.sexpr`, `line-comment-eof.sexpr`

Back to top · [TOC](#table-of-contents)

## Fast actions

Use the justfile recipes:

```bash
just validate      # links, mermaid fence balance, whitespace advisories
just check-iso     # ISO shape (stub checks)
just anchors       # regenerate ANCHORS.md (stub generator)
just status        # print Status block
just status-update # update Status in DASHBOARD.md between markers
just all           # run both
```

Individual scripts:

```bash
just validate
just check-iso
just anchors
```

Grep tips:

```bash
rg "#task-" plan.md
rg "<a id=\"" -n **/*.md
```

Back to top · [TOC](#table-of-contents)

## Shortlist (Next)

See [plan.md#shortlist-next](docs/development/planning/plan.md#shortlist-next). Top items:

| Item | Link |
|---|---|
| Types/generics (as/of) | [task 8](docs/development/planning/plan.md#8-type-annotations-and-generics-as-of-operators) |
| Zig-style errors | [task 6](docs/development/planning/plan.md#6-zig-style-error-handling) |
| Option generic | [task 14](docs/development/planning/plan.md#14-option-generic-optional-values) |
| Result/Option sugar | [task 15](docs/development/planning/plan.md#15-syntax-sugar-for-result-and-option) |
| Async/await | [task 10](docs/development/planning/plan.md#10-async--await) |
| Async × Errors integration | [task 29](docs/development/planning/plan.md#29-integration-async-concurrency-x-zig-style-error-handling) |

Back to top · [TOC](#table-of-contents)

## Quality gates

- ISO shape clean (no Unicode ellipses; strings/escapes unified; comments consume newline/EOF)
  - `just check-iso`
- Anchors regenerated and clickable
  - `just anchors`
- Mermaid fences balanced
  - `just validate`
- No trailing spaces in markdown (advisory)
  - `just validate`

Back to top · [TOC](#table-of-contents)

## Release readiness

- Docs banner set (README, FEATURES, plan.md, DIAGRAMS)
- Feature matrix and operators table updated
- Examples added/paired for new features
- Anchors updated (ANCHORS.md regenerated)
- CHANGELOG updated with date and categories

Back to top · [TOC](#table-of-contents)

## Search tips

- Jump to tasks: search for `#task-` or use [ANCHORS](docs/reference/anchors.md)
- Profiles: [A](docs/specifications/profiles.md#profile-a) · [B](docs/specifications/profiles.md#profile-b) · [C](docs/specifications/profiles.md#profile-c)
- Operators: [table](docs/reference/tables/operators.md#operators-table) and [reference](docs/reference/keywords-operators.md#keywords-and-operators-reference)

Back to top · [TOC](#table-of-contents)

## Diagrams

- See [DIAGRAMS.md](DIAGRAMS.md) sections for evaluation pipeline, infix×pipeline desugaring, Result/Option sugar, async×errors, if-some, comprehensions, pipeline into, and record update.

Back to top · [TOC](#table-of-contents)

## Anchors

- Clickable anchors index — [ANCHORS.md](docs/reference/anchors.md)
- Regenerate (stub generator): `just anchors`

Back to top · [TOC](#table-of-contents)

## Contribution

- Follow upcoming `CONTRIBUTING.md`, `docs-template-17.md`, and `STYLE-EBNF.md` (see plan-docs).
- For new features, start from `plan.md` tasks; ensure desugaring to S-exprs and ISO shape.

Back to top · [TOC](#table-of-contents)

## Recent changes

- Latest entry — [CHANGELOG.md#2025-10-15](CHANGELOG.md#2025-10-15)
