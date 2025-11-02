# Backlog

This is a pragmatic, prioritized backlog derived from `plan.md` and `ROADMAP.md`. Items reference task numbers from `plan.md` for traceability.

## Next (high priority, ready to implement)

- Core features (M4)
  - Modules and imports (Task 1): author `17-modules-imports.md` (S-expr + indentation), examples, desugaring.
  - Function definitions (Task 2): `17-defn.md` with arrow form, defaults/named/variadics hooks (Task 17 alignment).
  - Pattern matching (Task 5): `17-match.md` + examples and mapping table.
  - Records and destructuring (Task 4): `17-records.md` with access/update sugar.
  - Zig-style error handling (Task 6): `17-errors-zig.md` (try/catch/error) + integration seeds.
  - Type annotations and generics as/of (Task 8): `17-types-as-of.md` (+ VB-inspired details).
  - Pipelines (Task 11) and placeholders: `17-pipeline.md` (+ sections/compose/tap from Task 26).
  - Comprehensions (Task 7): `17-comprehensions.md` (+ multi-clause Task 22).

- Option/Result Ergonomics (M5)
  - Option generic (Task 14): `17-option.md` (T? policy), some/none.
  - Result/Option sugar (Task 15): coalesce ??, postfix try ?, if-some; type sugar T!.

- Async × Error integration (M5)
  - Async/await (Task 10) + integration (Task 29): await?, cancellation, timeouts.

## Near-term (design + EBNF authoring)

- Traits / implementations (Task 9): `17-traits-impls.md` + associated types/derive (Task 24).
- Where-block (Task 18) and Let sugar (Task 3): `17-where.md`.
- Interpolation and literals (Tasks 38, 90, 76, 70, 58, 72, 71): `17-literals.md` bundle.
- Types: aliases, unions/intersections, ADTs/newtype (Tasks 23, 53, 54): `17-types-adt.md`.
- Docs/infra (Task 28): `lexical-snippets.md` and per-file headers.

## Medium-term (extensions waves)

- Exports/re-exports (Task 16).
- Function extras (Task 17): defaults/named/variadics/overloads.
- Pattern enrichments (Task 20), dict/array patterns (Task 109), view patterns (Task 102).
- SQL-like queries (Task 42) and pipeline into (Task 98).
- Effects/handlers (Task 43).
- Generators / FRP streams (Tasks 32, 74).
- Conditional compilation (Task 36) and const-eval (Task 69).
- Error context/tags (Task 46) and error-channel alias !> (Task 110).

## Longer-term / nice-to-have

- Macros & hygiene (Task 30).
- FFI (Task 35) and inline assembly (Task 100).
- Property-based tests (Task 64) and FSMs (Task 65).
- Contracts, units, assertions, docstrings (Tasks 49–52).
- Ranges/slicing advanced (Tasks 33, 94, 106, 114) and bit/SIMD (Tasks 81–82).
- Do-notation (Task 37), match-lambda (Task 60), curry/uncurry (Task 85).
- Safe navigation (!!/?.) and casts as?/as! (Tasks 55, 93, 92).

## Integrations & cross-cutting

- Async × Errors × Pipelines: ensure precedence and desugaring order; finalize examples (Tasks 10, 15, 26, 29, 110).
- Types × Errors: `Result of T`, `T!` sugar, return type annotations (Tasks 8, 14–15, 29, 103).
- Modules packaging: wildcard/filters/using/alias (Tasks 16, 45, 63, 112).

## Documentation & validation

- Profiles (`profiles.md`): add mapping tables and round-trip examples for A/B/C after each feature lands.
- Diagrams (`DIAGRAMS.md`): update when desugarings change.
- Round-trip tests: surface → S-expr → print; collect golden examples per feature.

## Open questions

- Policy toggles: enable/disable `T?`, `T!`, postfix `?`, coalesce `??` per profile.
- Operator tiers: final placement for `??`, `|>>`, `!>`.
- Unicode operators: normalization rules and allowed set.

## Risks / blockers

- Grammar drift across files; mitigate with `lexical-snippets.md` and shared snippets.
- Ambiguity from overlapping sugars (e.g., pipelines + do-notation + if-some); resolve with explicit precedence and desugaring order.
- Tooling acceptance of ISO EBNF (Mermaid/markdown usage does not affect grammar files).
