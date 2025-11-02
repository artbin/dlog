# Examples Guide

## Pairing
- Create pairs with the same base name: `name.surface` and `name.sexpr` in `examples/`.
- Keep snippets small; prefer focused, copyable blocks.

## Categorization
- By profile: A (core S-expr) / B (application/infix/pipeline) / C (indentation)
- By theme: pipelines, precedence/infix, indentation, comments/strings, literals/data

## Verification
- Ensure desugared surface matches the canonical `.sexpr`.
- Use `just validate` and `just check-iso` before submitting.

## Indexing
- Add your files to `examples/README.md` under the appropriate headings.
- Cross-link in DASHBOARD highlights if especially illustrative.
