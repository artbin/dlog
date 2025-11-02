# Cookbook

Quick, copyable recipes showing Surface usage and desugaring to the S-expression core.

See also: `sexpr.md` (Core), `surface.md` (Surface), `profiles.md` (profiles A/B/C).

## Table of Contents

- Pipelines (Profile B)
- Application vs infix (10: app tighter than infix)
- Prefix/Postfix (11)
- Minimal precedence (12)
- Indentation variants (Profile C: 14–16)
- Literals and atoms (05–09)
- Round-trip notes

## Pipelines (Profile B)

Before → after (desugaring):

```text
x |> f            →  (f x)
xs |> map f       →  (map f xs)
xs |> map f |> sum → (sum (map f xs))
```

Tips:
- Pipeline threads the left value as the last argument.
- Combine with prefix/postfix or infix as needed; keep precedence rules in mind.

## Application vs infix (10)

Enable 10 to avoid extra parentheses when mixing application with infix.

```text
f x + g y     →  (+ (f x) (g y))

-- Without 10 (03 only), parenthesize:
(+ (f x) (g y))
```

## Prefix/Postfix (11)

Unary operators (examples are illustrative):

```text
neg x        →  (neg x)
x++           →  (++ x)
```

Combine with pipelines:

```text
x |> neg      →  (neg x)
```

## Minimal precedence (12)

Two tiers (e.g., */ above +/-). Parenthesize when ambiguous.

```text
1 + 2 * 3     →  (+ 1 (* 2 3))
```

## Indentation variants (Profile C, 14–16)

Optional colons after headers (14), single-line suites (15), trailing-operator join (16).

```text
if x > 0:
  return x
→ (if (> x 0) (return x))

if x > 0 -> return x
→ (if (> x 0) (return x))

# Trailing operator join (line ends with operator)
a
  |> f
  |> g
→ (g (f a))
```

Lexer notes:
- The lexer emits `NEWLINE`, `INDENT`, `DEDENT` for 04/14–16.
- Line comments are `#` and consume to "\n" | "\r\n" | ? EOF ?.

## Literals and atoms (05–09)

```text
:kw              ; keyword (05)
|bar-escaped|    ; symbol with spaces/specials (05)
#| nested comment |#  ; block comment (07)
#\\x             ; character (08)
#u8(1 2 255)     ; bytevector (08)
{ a : 1, b : 2 } ; map (09)
#{ 1, 2, 3 }     ; set (09)
```

## Round-trip notes

- Surface text may not round-trip textually after desugaring; S-expressions are the canonical round-trippable form.
- The evaluation pipeline is: parse Surface → desugar → canonical S-expressions → macroexpand → evaluate.
