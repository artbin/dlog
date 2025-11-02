# Single-Line Suites (Arrow Form) EBNF

Adds an alternative single-line suite form using `->` to `04-indentation-ebnf.md`.

Depends on: `04-indentation-ebnf.md` (offside rule and suites)

```ebnf
(* Replace suite in 04 with a second alternative *)

suite         = NEWLINE , INDENT , stmts , DEDENT
              | _ , "->" , _ , small ;
```

Notes:
- Use for concise one-liners: `if cond -> return x`.
- `small` is the same nonterminal used by `simple_stmt` in 04.

## Examples

```
if x > 0 -> return x
while pred x -> pass
def inc(n) -> return n + 1
```

## Edge Cases

- Only a single `small` statement is permitted after `->`; for multi-step bodies, use an indented suite.
- `->` binds to the nearest header; do not allow chaining like `if a -> if b -> ...` without parentheses.
