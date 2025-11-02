# Operators (Profiles B/C)

<a id="operators-table"></a>

Conventions: minimal precedence (12) defines two tiers; application tighter than infix (10) places application above infix. Pipeline `|>` (13) is left-associative. Prefix/postfix (11) are unary.

| Token | Kind | Precedence | Associativity | Profiles | Defined in |
|---|---|---|---|---|---|
| `|>` | pipeline | lower than `+/-` | left | B, C | [13](../../specifications/grammar/13-pipeline.md)
| `+` | infix | low | left | B, C | [12](../../specifications/grammar/12-minimal-precedence.md)
| `-` | infix | low | left | B, C | [12](../../specifications/grammar/12-minimal-precedence.md)
| `*` | infix | high | left | B, C | [12](../../specifications/grammar/12-minimal-precedence.md)
| `/` | infix | high | left | B, C | [12](../../specifications/grammar/12-minimal-precedence.md)
| prefix ops (e.g., `neg`) | prefix | n/a (unary) | n/a | B, C | [11](../../specifications/grammar/11-prefix-postfix.md)
| postfix ops (e.g., `++`) | postfix | n/a (unary) | n/a | B, C | [11](../../specifications/grammar/11-prefix-postfix.md)
| `??` | coalesce | lowest (planned) | left | B, C | [plan 15](../../development/planning/plan.md#15-syntax-sugar-for-result-and-option)
| `?` | postfix try | unary (planned) | n/a | B, C | [plan 15](../../development/planning/plan.md#15-syntax-sugar-for-result-and-option)
| `!>` | result piping | special (planned) | left | B, C | [plan 110](../../development/planning/plan.md#110-pipeline-error-channel-alias)

Notes:
- Base infix (03) has no precedence; enable (12) to activate the two-tier precedence.
- With (10), application binds tighter than any infix operator.
- Additional operators (e.g., `??`, `!>`) are tracked in `plan.md` tasks 15 and 110 and will be added when implemented.
