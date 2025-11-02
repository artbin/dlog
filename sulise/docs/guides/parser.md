# Parser Guide

Guidance for integrating profiles and expression tiers.

## Application (right-associative)
- Parse juxtaposition as a list of operands then fold-right to unary application: `a b c` ⇒ `a (b c)`.

## Infix base (03)
- Single level, no precedence by default; require parentheses when mixing infix and application unless (10) is enabled.

## Minimal precedence (12)
- Two tiers: high `*/`, low `+/-`.
- Associativity: left for both tiers.

## Application tighter than infix (10)
- Make application bind tighter than any infix operator, enabling `f x + g y` without extra parentheses.

## Prefix/Postfix (11)
- Parse unary prefix and postfix operators; enforce no ambiguity with application.

## Pipeline `|>` (13)
- Left-associative; threads left value as last argument: `x |> f` ⇒ `(f x)`.
- Compose left-to-right; ensure it is lower than `+/-` in the minimal-precedence setup.

## Desugaring stage
- After parsing surface syntax (B/C), perform a desugar pass into canonical S-expressions (A).
- Order: parse → desugar → macroexpand → evaluate.

## Indentation profile (04, 14–16)
- Treat `NEWLINE`/`INDENT`/`DEDENT` as tokens provided by the lexer.
- Optional colons and single-line suites affect surface only; desugaring yields canonical forms.

## Error handling and types (planned)
- Reserve keywords per profile; ensure grammar nonterminals leave room for `as`/`of`, `try`/`catch`, `Option`, etc.

## Testing
- Include fixtures: surface input → expected S-expr output.
- Validate pipelines, precedence mixes, and indentation edge cases.
