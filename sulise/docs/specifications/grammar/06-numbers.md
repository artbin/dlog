# Numbers: Radix, Exactness, Digit Separators

Extends `01-sexpr-ebnf.md` with Scheme/CL-style radix and exactness prefixes and optional `_` digit separators.

Depends on: `01-sexpr-ebnf.md` (base S-expression grammar)

```ebnf
(* Replace/extend number with sign, radix/exactness and separators *)

number        = [ sign ] , [ exactness ] , [ radix ] , num_body , [ exponent ] ;

exactness     = "#e" | "#i" ;
radix         = "#b" | "#o" | "#d" | "#x" ;

num_body      = rdigits , [ "." , rdigits ] | "." , rdigits ;

rdigits       = rdigit , { [ "_" ] , rdigit } ;

(* Base-specific digit sets; enforce per chosen radix in lexer/validator *)
rdigit        = digit ;                      (* #d: 0..9 *)
(* for #b use: rdigit = "0" | "1" *)
(* for #o use: rdigit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" *)
(* for #x use: rdigit = xdigit *)

xdigit        = digit
              | "a" | "b" | "c" | "d" | "e" | "f"
              | "A" | "B" | "C" | "D" | "E" | "F" ;

exponent      = ( "e" | "E" ) , [ sign ] , digits ;   (* same as 01 *)
```

Notes:
- Decimal exponent is kept simple; hex floats can be added later if desired.
- Separators (`_`) are ignored semantically; reject adjacent/edge underscores in the lexer if needed.

## Rationale

- Radix and exactness prefixes mirror Scheme/CL practice and improve literal clarity.
- Separators improve readability of large numerals without affecting value.

## Examples

```
#xFF          ; 255 (hex)
#b1010_1111   ; 0b10101111 with `_` separators
#o755         ; octal
#d123_456     ; decimal with separators
#e10          ; exact 10 (implementation-defined exactness semantics)
#i3.14        ; inexact 3.14
-#x2A         ; negative with radix
1_000_000e-3  ; 1000.0 with exponent and separators
```

## Compatibility & Validation

- Enforce `rdigit` set according to active `radix`; signal error on mismatched digits.
- Exponent token `e|E` is typically decimal-only; if you want hex-float (`p|P`) exponents, add a variant.
- Forbid leading/trailing `_` or doubled separators; allow `1_2_3` but not `_1` or `1__2`.

