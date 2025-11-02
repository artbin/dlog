# Types Plan (as/of)

Roadmap for type annotations and generics inspired by Visual Basic (`as` / `of`).

## Scope
- `x as T`, params `a as T`, returns `as U`
- Generics: `Name of T, U`
- Nested generics, arrays `T()`, nullable `T?` (policy), Result shorthand `T!` (policy)

## EBNF (sketch)
- Reserve `as`, `of` in profiles that enable types.
- Ensure keywords do not clash with operator characters or identifiers.

## Desugaring
- `x as T` ⇒ `(ann x T)`
- `List of T` ⇒ `(of List T)`
- Return and param annotations carried as metadata in S-expr nodes

## Tasks
- [task 8](plan.md#8-type-annotations-and-generics-as-of-operators)
- [task 14](plan.md#14-option-generic-optional-values)
- [task 15](plan.md#15-syntax-sugar-for-result-and-option) (type sugar `T!`)

## Examples
```text
userIds as List of Int   ->  (ann userIds (of List Int))
```
