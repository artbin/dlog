# Lisp/S-Expression EBNF

A compact, dialect-agnostic EBNF for Lisp/S-expressions (good for Scheme/CL readers). It covers lists, dotted pairs, vectors, symbols, numbers, strings, booleans, and quote macros.

## Grammar

```ebnf
program        = _ , { sexpr , _ } ;

sexpr          = atom | list | vector | quoted ;

quoted         = quote_macro , _ , sexpr ;
quote_macro    = "'" | "`" | ",@" | "," ;     (* quote, quasiquote, unquote-splicing, unquote *)

list           = "(" , _ ,
                 [ sexpr , { _ , sexpr } , [ _ , "." , _ , sexpr ] ] ,
                 _ , ")" ;                     (* supports dotted pairs *)

vector         = "#(" , _ , [ sexpr , { _ , sexpr } ] , _ , ")" ;

atom           = boolean | number | string | symbol ;

boolean        = "#t" | "#f" | "t" | "nil" ;

number         = [ sign ] ,
                 ( digits , [ "." , digits ] | "." , digits ) ,
                 [ exponent ] ;
sign           = "+" | "-" ;
digits         = digit , { digit } ;
digit          = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
exponent       = ( "e" | "E" ) , [ sign ] , digits ;

string         = '"' , { string_char } , '"' ;
string_char    = escape | ? any char except " and \ and control ? ;
escape         = "\" , ( "\" | '"' | "n" | "t" | "r" ) ;

symbol         = symbol_initial , { symbol_subsequent } ;
symbol_initial = letter | special_initial ;
symbol_subsequent
               = symbol_initial | digit | "+" | "-" | "." | "@" | "#" ;
letter         = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
               | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
               | "U" | "V" | "W" | "X" | "Y" | "Z"
               | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
               | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
               | "u" | "v" | "w" | "x" | "y" | "z" ;
special_initial= "!" | "$" | "%" | "&" | "*" | "/" | ":" | "<" | "=" | ">"
               | "?" | "~" | "_" | "^" ;

_              = { whitespace | comment } ;
whitespace     = " " | "\t" | "\n" | "\r" ;
comment        = "#" , { ? any char except newline ? } , ( "\n" | "\r\n" | ? EOF ? ) ;
```

## Notes

- This treats `_` as "skip any amount of whitespace/comments"
- `list` allows dotted pairs via the optional `.` rule
- `quote_macro` includes `'`, `` ` ``, `,`, `,@`
- `vector` handles `#(...)`
- The exact symbol and number rules vary by dialect/readtable; adapt as needed (e.g., package prefixes in Common Lisp, `#\c` characters, radix notations like `#xFF`, block comments `#|...|#`, etc.). If you want a version tailored precisely to R7RS Scheme or ANSI Common Lisp, say which one and I'll specialize this.
