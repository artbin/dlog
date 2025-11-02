# Testing Docs & Examples

## Fixtures
- Add surface `.surface` and expected `.sexpr` files with the same base name to `examples/`.
- Keep snippets small and copyable; group by theme/profile.

## Validation scripts
- Links & fences: `just validate`
- ISO shape: `just check-iso`
- Anchors: `just anchors`

## Round-trip
- Validate surface → desugar → canonical S-expr printing matches `.sexpr`.
- Surface text is not guaranteed to round-trip; only S-exprs are canonical.

## Edge cases
- Indentation regressions: optional colons, single-line suites, trailing operator join.
- Pipelines mixed with precedence; prefix/postfix interactions; application tighter than infix.

## Diagrams
- Ensure Mermaid fence balance; cross-link from DIAGRAMS.md to relevant tasks.
