# Diagrams

> Docs status: Unreleased (updated 2025-10-15). See [CHANGELOG](CHANGELOG.md#2025-10-15).

## Evaluation pipeline (all profiles)

```mermaid
flowchart LR
  A[Source text] --> B[Parse (selected profile)]
  B --> C[Desugar to canonical S-expressions]
  C --> D[Macroexpand (hygienic macros)]
  D --> E[Evaluate over S-exprs]
```

## Infix and pipeline desugaring

```mermaid
graph LR
  s1["f x + g y"] --> d1["(+ (f x) (g y))"]
  s2["x |> f |> g"] --> d2["(g (f x))"]
```

## Result/Option sugar

```mermaid
flowchart TD
  start((res ?? def))
  start --> chk{res}
  chk -->|ok x| x1[(x)]
  chk -->|err e| def1[(def)]
```

```mermaid
flowchart TD
  ptry["e ?"] --> t1["(try e)"]
```

## Async × error handling

```mermaid
flowchart TD
  aw["await? t"] --> awt["(try (await t))"]
  awc["await t within ms catch e -> b"] --> awc2["(catch (await-with-timeout t ms) (lambda (e) b))"]
```

## Option if-some binding

```mermaid
flowchart TD
  is["if some x <- opt -> body else alt"] --> m1["(match opt ((some x) body) ((none) alt))"]
```

## Comprehension expansion

```mermaid
flowchart LR
  c1["[ e | x in xs, if p ]"] --> c2["(for (x in xs (if p)) e)"]
```

## Pipeline into binding

```mermaid
flowchart LR
  pin["x |> f |> into y -> body"] --> let1["(let ((y (f x))) body)"]
```

## Record update sugar

```mermaid
flowchart LR
  upd["with obj { k = v }"] --> upd2["(update obj :k v)"]
```

