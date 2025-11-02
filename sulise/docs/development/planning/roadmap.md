# Roadmap

This roadmap summarizes the planned work for the grammar set across cleanup, homoiconicity, core features, and extensions. See `plan.md` for detailed tasks, dependencies, examples, and diagrams.

## Milestones

- M1: ISO EBNF compliance (Phase 1)
  - Expand ranges, unify quoting/strings/escapes/skippers, wrap prose in `? ... ?`, normalize comment rules.
- M2: ISO EBNF wording and uniformity (Phase 2)
  - Standardize phrasing and special-sequence wording; unify examples and endings.
- M3: Homoiconicity enforcement
  - Document and implement desugaring to canonical S-expressions for all profiles and features.
- M4: Core high-level constructions
  - Modules/imports, function definitions, let/where, records/destructuring, pattern matching, Zig-style error handling, comprehensions, types/generics, traits/impls, async/await, pipelines.
- M5: Option/Result ergonomics and async×error integration
  - Option generic and sugar (T?), Result sugar (T!), coalesce, postfix try, if-some; integrate with async (await?, cancellation, timeouts).
- M6: Extensions (waves)
  - Wave A: exports/re-exports, defaults/named args/variadics, where-block sugar, record access/update, pattern or/as/guard, comprehension clauses.
  - Wave B: types alias/union/intersect, ADTs/newtype, units, assertions/contracts/docstrings.
  - Wave C: macros/hygiene, effects/handlers, FFI, conditional compilation, const-eval.
  - Wave D: literals/lexical (interpolation, raw/heredoc/bytes, datetime, duration/size, path, Unicode ops), ranges/slicing.
  - Wave E: generators/FRP, SQL-like queries, DSL blocks, pipelines (sections/compose/tap/into), property tests, FSMs, caching/memoization.
- M7: Docs and examples
  - Enrich `README.md`, `profiles.md` profiles, per-feature `17-*.md` with EBNF + desugaring + examples; diagrams.
- M8: Validation
  - Round-trip examples (surface → S-expr → print); conformance checks for ISO EBNF shape.

## Implementation order (suggested)

1) M1 → M2: Complete ISO compliance and uniformity.
2) M3: Finalize desugaring contract and mapping tables in `profiles.md` and `README.md`.
3) M4: Ship core high-level constructions (prioritize: modules/imports, defn, match, errors, types/generics, async/await, pipelines, comprehensions, records).
4) M5: Add Option/Result ergonomics and async×error integration (await?, coalesce ??, postfix try ?, if-some).
5) M6: Deliver extensions in waves, each small and self-contained; keep homoiconicity.
6) M7–M8: Documentation and validation (round-trips, examples, diagrams).

## Status tracking

- Phase 1/2 ISO: see `plan.md` (Phase 1 Tasks 1–10, Phase 2 Tasks 1–5).
- Homoiconicity: see `plan.md` (“Homoiconicity Preservation Plan”, Tasks 1–5).
- High-Level Constructions and Extensions: see `plan.md` (Tasks 1–15 and 16–114).

## Dependencies and coupling

- Base: `01-sexpr-ebnf.md` + profiles (`03` infix/operators; `04` indentation).
- Expression tiers: `10–13` (application precedence, prefix/postfix, minimal precedence, pipeline).
- Data/literals: `05–09` (keywords/escaped symbols, numbers, block comments, chars/bytevectors, maps/sets).
- Option/Result: Tasks 14–15; async/error integration: Tasks 25, 29.

## Deliverables per feature file

- New features follow the pattern: `17-<feature>.md` with:
  - EBNF (S-expr + surface), desugaring table to S-exprs, 2–3 examples, dependencies, “Preserves Homoiconicity” note.
  - Cross-links to base specs and `profiles.md` profiles.

## Diagrams

See `plan.md` “Diagrams” for reference flows (evaluation pipeline, desugaring for infix/pipeline, Option/Result, async×errors, pipeline-into, record update).

## Validation

- Round-trip examples for each profile and feature.
- ISO EBNF shape checks (no ellipses, consistent terminals, `? ... ?` prose, unified strings/escapes/skippers/comments).

## Scope control

- Prefer small, composable variants with explicit desugaring.
- Keep evaluation over S-expressions only; surface features must not change runtime representation.
