# Keywords and Operators (Reference)

This page lists reserved keywords and operator characters by profile and links to defining documents.

Profiles:
- A = S-expressions (core)
- B = Application + infix/prefix/postfix + pipeline
- C = Indentation/offside variants

## Keywords

| Keyword | Profiles | Meaning / Usage | Defined in |
|---|---|---|---|
| `as` | B, C | Type annotation keyword (planned high-level) | [plan task 8](../development/planning/plan.md#8-type-annotations-and-generics-as-of-operators) |
| `of` | B, C | Generic type application (planned high-level) | [plan task 8](../development/planning/plan.md#8-type-annotations-and-generics-as-of-operators) |
| `try` | B, C | Propagate error (planned) | [plan task 6](../development/planning/plan.md#6-zig-style-error-handling) |
| `catch` | B, C | Handle error (planned) | [plan task 6](../development/planning/plan.md#6-zig-style-error-handling) |
| `error` | B, C | Construct error (planned) | [plan task 6](../development/planning/plan.md#6-zig-style-error-handling) |
| `some` | B, C | Option constructor (planned) | [plan task 14](../development/planning/plan.md#14-option-generic-optional-values) |
| `none` | B, C | Option empty (planned) | [plan task 14](../development/planning/plan.md#14-option-generic-optional-values) |
| `await` | B, C | Await async (planned) | [plan task 10](../development/planning/plan.md#10-async--await) |
| `async` | B, C | Create async (planned) | [plan task 10](../development/planning/plan.md#10-async--await) |

## Operators / opchar

| Token | Kind | Profiles | Defined in | Notes |
|---|---|---|---|---|
| `|>` | pipeline | B, C | [13](../specifications/grammar/13-pipeline.md) | Left-assoc; threads left as last arg |
| `+` `-` | infix | B, C | [12](../specifications/grammar/12-minimal-precedence.md) | Low tier |
| `*` `/` | infix | B, C | [12](../specifications/grammar/12-minimal-precedence.md) | High tier |
| prefix ops (e.g., `neg`) | prefix | B, C | [11](../specifications/grammar/11-prefix-postfix.md) | Unary |
| postfix ops (e.g., `++`) | postfix | B, C | [11](../specifications/grammar/11-prefix-postfix.md) | Unary |
| `??` | coalesce | B, C (planned) | [plan task 15](../development/planning/plan.md#15-syntax-sugar-for-result-and-option) | Lowest tier if added |
| `?` | postfix try | B, C (planned) | [plan task 15](../development/planning/plan.md#15-syntax-sugar-for-result-and-option) | Unary postfix |
| `!>` | result piping | B, C (planned) | [plan task 110](../development/planning/plan.md#110-pipeline-error-channel-alias) | Pipe only ok branch |

Notes:
- Reserve keywords only in profiles that enable the corresponding features.
- Keep operator characters out of identifiers where relevant; see each EBNF for `opchar` sets.
