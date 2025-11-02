# Errors Plan (Zig-style)

Explicit errors via error unions and control constructs; integrates with Result/Option and async.

## Scope
- `try e` (propagate), `e catch err -> body` (handle), `error(.Tag[, payload])` (construct)

## Desugaring
- `try e` ⇒ `(try e)`
- `e catch err -> body` ⇒ `(catch e (lambda (err) body))`
- `error(.Tag[, p])` ⇒ `(error Tag [p])`

## Integrations
- Result/Option sugar (coalesce `??`, postfix try `?`, if-some binding)
- Async×errors: `await? t` ⇒ `(try (await t))`; `await t catch err -> b`

## Tasks
- [task 6](plan.md#6-zig-style-error-handling)
- [task 15](plan.md#15-syntax-sugar-for-result-and-option)
- [task 29](plan.md#29-integration-async-concurrency-x-zig-style-error-handling)

## Examples
```text
try open p catch e -> default   ->  (catch (try (open p)) (lambda (e) default))
```
