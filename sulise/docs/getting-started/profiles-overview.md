# Profiles (A/B/C)

This page introduces the three composable profiles and shows short examples with desugaring to canonical S-expressions.

## Profile A — S-expressions (Core)
- Canonical reader: lists, dotted pairs, vectors, atoms, quotes.
- Extensions: keywords/escaped symbols, numbers (radix/exactness), nested block comments, characters/bytevectors, maps/sets.
- Link: [profiles.md#profile-a](../specifications/profiles.md#profile-a)

Example (A):
```text
(define xs '(1 2 3))
(sum (map square xs))
```

## Profile B — Application + Infix (minimal precedence + pipeline)
- Right-associative application; optional infix; minimal precedence (+/− vs */); prefix/postfix; pipeline `|>` left-assoc.
- Link: [profiles.md#profile-b](../specifications/profiles.md#profile-b)

Surface (B):
```text
xs |> map f |> sum
```
Desugar (A):
```text
(sum (map f xs))
```

## Profile C — Indentation Blocks (offside rule)
- NEWLINE/INDENT/DEDENT from lexer; optional colons; single-line suites `->`; trailing-operator line join.
- Link: [profiles.md#profile-c](../specifications/profiles.md#profile-c)

Surface (C):
```text
if x > 0 -> return x
```
Desugar (A):
```text
(if (> x 0) (return x))
```

## Composition
- Start from A; add B or C (or both) per needs. All surface forms desugar to A (homoiconicity preserved).
- See also: [FEATURES.md](features.md), [README.md](../../README.md#precomposed-profiles), [examples](../../examples/README.md).
