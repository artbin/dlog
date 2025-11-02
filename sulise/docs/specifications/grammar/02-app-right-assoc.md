# Right-Associative Function Application EBNF

A tiny, self-contained EBNF for a surface language where function application is by juxtaposition, right-associative, and there are no precedence levels (only atoms, parenthesized expressions, and application).

## Grammar

```ebnf
(* Lexical skippers *)
_          = { whitespace | comment } ;
whitespace = " " | "\t" | "\n" | "\r" ;
comment    = "#" , { ? any char except newline ? } , ( "\n" | "\r\n" | ? EOF ? ) ;

(* Program *)
program    = _ , { expr , _ } ;

(* Expressions *)
expr       = app ;

(* Right-associative application:
   f x y z  ≡  f (x (y z)) *)
app        = rexpr , [ _ , app ] ;

(* rexpr = "right expression", the atomic operands in application *)
rexpr      = identifier
           | number
           | string
           | "(" , _ , expr , _ , ")" ;

(* Minimal lexemes — adapt as needed *)
identifier = letter , { letter | digit | "_" | "-" | "?" | "!" } ;
number     = [ "+" | "-" ] , digit , { digit } , [ "." , digit , { digit } ] ;
string     = '"' , { string_char } , '"' ;
string_char= escape | ? any char except " and \\ ? ;
escape     = "\\" , ( "\\" | '"' | "n" | "t" | "r" ) ;

letter     = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
           | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
           | "U" | "V" | "W" | "X" | "Y" | "Z"
           | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
           | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
           | "u" | "v" | "w" | "x" | "y" | "z" ;
digit      = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
```

## Desugaring Rule

After parsing, apply a fold-right into unary application:

- `E ::= S0` → `S0`
- `E ::= S0 S1 ... Sn` (n ≥ 1) → `S0 (S1 (... (Sn-1 Sn)...))`

## Examples

- `f x` → `(f x)`
- `f x y` → `(f (x y))`
- `map f xs` → `(map (f xs))`
- `f (g x) h` → `(f ((g x) h))` (because `rexpr` permits parenthesized subexpressions)

## Notes

- There are no infix operators and hence no precedence to remember; introduce any other constructs via parentheses or as separate nonterminals if you later extend the language.
- The grammar uses right recursion in `app` to enforce right-association directly in the concrete syntax. If you prefer an LL(1)-friendlier grammar, parse a nonempty list of `rexpr` and then do the right fold in a post-parse step.
