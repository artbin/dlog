# 17-<feature-name>.md (Template)

[Profiles: A,B,C] [Tags: types, errors]

## Overview
- One-paragraph summary. State homoiconicity: surface desugars to canonical S-exprs.

## Rationale
- Why this exists; tradeoffs; interactions with other features.

## EBNF (ISO-clean)
- Depends on: 01-sexpr, (03 infix), (04 indentation), (10–13 variants) as needed.
- Provide productions using double-quoted terminals; prose in `? ... ?` only.

## Desugaring (Surface → S-expressions)
- Mapping table or bullet list of forms and their canonical S-expr equivalents.

## Examples
```text
# surface
...
```
```text
# S-expr (canonical)
...
```

## Dependencies
- Profiles and features required; note operator tiers; lexer requirements (indent tokens).

## Notes
- Validation points (e.g., numeric ranges); implementation hints; pitfalls.

[Back to top](#17-feature-namemd-template)
