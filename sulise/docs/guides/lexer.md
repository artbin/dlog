# Lexer Guide

Guidance for lexing behaviors referenced by the EBNF documents.

## Indentation (offside rule)
- Emit tokens: `NEWLINE`, `INDENT`, `DEDENT`.
- Track a stack of indentation levels; on each physical newline, compute current indent.
- Rules:
  - Increase indent → emit `INDENT` once.
  - Decrease indent → emit one or more `DEDENT` until stacks match.
  - Equal indent → do not emit indent/dedent tokens.
- Suppress `NEWLINE` when line ends with a trailing operator (variant 16).
- Optional colon after headers (variant 14) does not affect indentation tokens.

## Comments
- Line comments: start with `#` and consume characters up to `"\n" | "\r\n" | EOF`.
- Nested block comments: `#| ... |#` (variant 07) must be counted (nesting depth).

## Strings and escapes
- Use canonical rules from STYLE-EBNF.md for escapes.
- Enforce that backslash escapes only include `\\`, `"`, `n`, `t`, `r`.

## Numbers
- Radix and exactness prefixes (see 06): accept `#b`, `#o`, `#d`, `#x`, `#e`.
- Digit separators `_`: allow between digits but not at start/end.
- Validate digit sets per base (e.g., hex digits for `#x`).

## Tokens and operators
- Reserve keywords (`as`, `of`, `try`, `catch`, `error`, etc.) only in profiles that enable them.
- Enumerate operator characters per EBNF; avoid overlap with identifiers.

## Newline policy
- Normalize CRLF to `\n` internally; still accept both.
- Ensure comments consume the trailing newline; the parser sees `NEWLINE` after comments only when appropriate.
