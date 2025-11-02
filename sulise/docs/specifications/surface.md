# Surface Syntax

"Surface" refers to any user-facing concrete syntax layered over the S-expression core. Surface syntax exists to improve readability and ergonomics, but it has no independent semantics: it desugars to the canonical S-expression form before macro-expansion and evaluation.

## Contract

- Parse Surface → Desugar → Canonical S-expressions → Macroexpand → Evaluate.
- The S-expression AST is the source of truth; Surface may not round-trip textually.
- Homoiconicity applies to the core (S-expressions), not to Surface text.

## Profiles

- Profile A (Core): canonical S-expressions (no Surface).
- Profile B (Surface): application with optional infix, prefix/postfix, pipeline `|>`.
- Profile C (Surface): indentation/offside syntax; optional colons, single-line suites.

See `profiles.md` → [Profile B](profiles.md#profile-b) and [Profile C](profiles.md#profile-c).

## Examples (before → after)

```text
f x + g y        →  (+ (f x) (g y))

data |> map f
            |> sum → (sum (map f data))

if x > 0 -> return x
                 →  (if (> x 0) (return x))
```

## Design rules

- No new semantics: Surface is purely syntactic sugar over the core.
- Explicit desugaring: every Surface rule documents its S-expression mapping.
- ISO EBNF shape: prose only inside `? ... ?`; terminals are double-quoted metasymbols.
- Comments: line comments use `#`; if 07 is enabled, nested block comments are `#| ... |#`.
- Indentation tokens: when using 04/14–16, the lexer emits `NEWLINE`/`INDENT`/`DEDENT`.

## Terminology

- Surface forms: concrete forms written by users (e.g., infix, pipelines, indentation headers).
- Core forms: the desugared S-expressions that drive macroexpansion and evaluation.

## See also

- Core explainer: `sexpr.md`
- Precomposed profiles: `profiles.md`
- Overview: `FEATURES.md`
- Homoiconicity contract: `README.md` → Homoiconicity
