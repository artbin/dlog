# Operators Deep Dive

This document elaborates precedence, associativity, and interactions with application and pipelines.

## Tiers and associativity
- Minimal precedence (12): high `*/` (left), low `+/-` (left).
- Application tighter than infix (10): application binds tighter than any infix.
- Pipeline (13): left-associative; sits below `+/-` in minimal precedence.

## Prefix/Postfix (11)
- Unary operators; avoid ambiguity at application boundaries.

## Examples
```text
1 + 2 * 3      ->  (+ 1 (* 2 3))
f x + g y * z  ->  (+ (f x) (* (g y) z))
```

## Planned operators
- Coalesce `??` (lowest tier), postfix try `?`, result piping `!>` (see plan tasks 15, 110).

## References
- Table: [tables/operators.md](../reference/tables/operators.md#operators-table)
- EBNF: [12-minimal-precedence](../specifications/grammar/12-minimal-precedence.md), [10-app-tighter-than-infix](../specifications/grammar/10-app-tighter-infix.md), [11-prefix-postfix](../specifications/grammar/11-prefix-postfix.md), [13-pipeline](../specifications/grammar/13-pipeline.md)
