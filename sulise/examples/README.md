# Examples Index

This directory contains Surface and S-expression example pairs. Surface files use Profiles B/C; S-expressions are canonical core forms (Profile A). Each Surface example has a matching desugared `.sexpr` where applicable.

## Profiles
- A: Core S-expressions (`*.sexpr`)
- B: Application + infix/prefix/postfix + pipeline (`*.surface` without indentation tokens)
- C: Indentation/offside, optional colons, single-line suites, trailing-operator join (`*.surface` using indentation)

## Index by feature

### Pipelines (B)
- pipeline-composed.surface ↔ pipeline-composed.sexpr
- pipeline-foldl.surface ↔ pipeline-foldl.sexpr
- pipeline-into-reduce.surface ↔ pipeline-into-reduce.sexpr
- pipeline-nesting.surface ↔ pipeline-nesting.sexpr
- pipeline-compose-2.surface ↔ pipeline-compose-2.sexpr
- complex-mixed.surface ↔ complex-mixed.sexpr

### Infix / precedence / application (B)
- operators-precedence.surface ↔ operators-precedence.sexpr
- paren-precedence.surface ↔ paren-precedence.sexpr
- matrix-mul-precedence.surface ↔ matrix-mul-precedence.sexpr
- paren-force-assoc-2.surface ↔ paren-force-assoc-2.sexpr
- application-with-parens.surface ↔ application-with-parens.sexpr
- app-right-assoc.surface ↔ app-right-assoc.sexpr
- prefix-postfix-mix.surface ↔ prefix-postfix-mix.sexpr
- line-join-infix.surface ↔ line-join-infix.sexpr
- map-filter-reduce.surface ↔ map-filter-reduce.sexpr

### Indentation / offside (C)
- indentation-blocks.surface ↔ indentation-blocks.sexpr
- indentation-complex.surface ↔ indentation-complex.sexpr
- indent-singleline-colon.surface ↔ indent-singleline-colon.sexpr
- multiline-join-infix-pipe.surface ↔ multiline-join-infix-pipe.sexpr
- indent-pipeline-nest.surface ↔ indent-pipeline-nest.sexpr
- trailing-join-2.surface ↔ trailing-join-2.sexpr

### Literals / data (A)
- literals.sexpr
- numbers-radix.sexpr
- numbers-radix-mix.sexpr
- nested-collections.sexpr
- literals-nested-2.sexpr
- vectors-and-dotted.sexpr

### Comments / strings (A/B)
- block-comments.surface ↔ block-comments.sexpr
- block-comments-inline-2.surface ↔ block-comments-inline-2.sexpr
- line-comment-eof.sexpr
- strings-escapes.sexpr
- strings-escapes-2.sexpr

### Core complex program (A/B)
- complex.sexpr
- complex.surface
