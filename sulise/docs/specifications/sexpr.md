# S-expressions (Core)

S-expressions are the canonical, homoiconic core of this language: the program representation is the same data the language manipulates (lists, symbols, numbers, strings, etc.). All other syntaxes are sugar that desugar to this core.

## Surface vs Core

- **Surface (definition)**: Any user-facing concrete syntax layered over the S-expression core.
- **Examples (surface)**: Infix operators, pipeline `|>`, prefix/postfix, indentation blocks (Profiles B/C).
- **Core (definition)**: The minimal S-expression reader/AST (Profile A) that everything desugars into.
- **Contract**: Parse surface → desugar to canonical S-expressions → macroexpand → evaluate.
- **Notes**: Surface forms may not round-trip textually; the S-expression AST is the source of truth. Profile A is core; B/C are surface.

## See also

- Profile A (S-expression reader): `profiles.md` → [Profile A](profiles.md#profile-a)
- Overview of features and profiles: `FEATURES.md`
- Homoiconicity and desugaring contract: `README.md` → Homoiconicity
