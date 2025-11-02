# Characters and Bytevectors EBNF

Extends `01-sexpr-ebnf.md` by adding `#\` character literals and `#u8(...)` bytevectors.

Depends on: `01-sexpr-ebnf.md` (base S-expression grammar)

```ebnf
(* Extend atom with character and bytevector literals *)

atom          = boolean | number | string | symbol | character | bytevector ;

character     = "#\\" , char_body ;
char_body     = letter | digit | "space" | "tab" | "newline" ;

bytevector    = "#u8(" , _ , [ byte , { _ , byte } ] , _ , ")" ;
byte          = digits ;                      (* semantic check: 0..255 *)
```

Notes:
- Extend `char_body` with escapes/named controls as needed (e.g., `#\\xNN`).
- `digits` and `_` are reused from `01-sexpr-ebnf.md`.

## Rationale

- Character literals are essential for precise text handling; bytevectors provide a compact binary literal form.

## Examples

```
#\\A           ; letter A
#\\space       ; named space
#\\x41         ; hex escape (A)
#u8(1 255 0 16) ; four bytes
```

## Validation & Integration

- Restrict `byte` to 0..255 after parsing; signal error otherwise.
- Decide whether `#u8()` permits commas: current rule uses whitespace-separated bytes; add commas if desired.
- Extend `char_body` for Unicode named characters if you need portability (`#\\λ`).
