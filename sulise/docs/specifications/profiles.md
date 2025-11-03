# Unified EBNF Profiles

This document provides three ready-to-use grammar profiles assembled from the modular files in this repo. Use as-is or as a template for your own composition.

- Profile A: Lisp S-expression reader (01 + 05–09)
- Profile B: Application + infix surface (03 + 10 + 12 + 13)
- Profile C: Python-style indentation blocks (04 + 14–16)

---

## Profile A — Lisp S-expression Reader (with rich literals)
<a id="profile-a"></a>

Depends on: 01, 05, 06, 07, 08, 09

```ebnf
(* Program *)
program        = _ , { sexpr , _ } ;

(* Core *)
sexpr          = atom | list | vector | map | set | quoted ;
quoted         = quote_macro , _ , sexpr ;
quote_macro    = "'" | "`" | ",@" | "," ;   (* quote, quasiquote, unquote-splicing, unquote *)

list           = "(" , _ ,
                 [ sexpr , { _ , sexpr } , [ _ , "." , _ , sexpr ] ] ,
                 _ , ")" ;
vector         = "#(" , _ , [ sexpr , { _ , sexpr } ] , _ , ")" ;

map            = "{" , _ , [ pair , { _ , "," , _ , pair } ] , _ , "}" ;
pair           = sexpr , _ , ":" , _ , sexpr ;
set            = "#{" , _ , [ sexpr , { _ , sexpr } ] , _ , "}" ;

(* Atoms *)
atom           = boolean | number | string | keyword | symbol | character | bytevector ;
boolean        = "#t" | "#f" | "t" | "nil" ;

(* Numbers with sign, exactness, radix, separators *)
number         = [ sign ] , [ exactness ] , [ radix ] , num_body , [ exponent ] ;
sign           = "+" | "-" ;
exactness      = "#e" | "#i" ;
radix          = "#b" | "#o" | "#d" | "#x" ;
num_body       = rdigits , [ "." , rdigits ] | "." , rdigits ;
rdigits        = rdigit , { [ "_" ] , rdigit } ;
rdigit         = digit ;                        (* adjust per active radix in lexer *)
xdigit         = digit | "a" | "b" | "c" | "d" | "e" | "f"
               | "A" | "B" | "C" | "D" | "E" | "F" ;
exponent       = ( "e" | "E" ) , [ sign ] , digits ;

(* Strings *)
string         = '"' , { string_char } , '"' ;
string_char    = escape | ? any char except " and \\ and control ? ;
escape         = "\\" , ( "\\" | '"' | "n" | "t" | "r" ) ;

(* Symbols and keywords *)
keyword        = ":" , plain_symbol ;
symbol         = bar_symbol | plain_symbol ;
bar_symbol     = "|" , { bar_char } , "|" ;
bar_char       = sym_escape | ? any char except | and \\ ? ;
sym_escape     = "\\" , ( "\\" | "|" | "n" | "t" | "r" | "x" , xdigit , { xdigit } ) ;
plain_symbol   = symbol_initial , { symbol_subsequent } ;
symbol_initial = letter | special_initial ;
symbol_subsequent
               = symbol_initial | digit | "+" | "-" | "." | "@" | "#" ;
special_initial= "!" | "$" | "%" | "&" | "*" | "/" | ":" | "<" | "=" | ">"
               | "?" | "~" | "_" | "^" ;

(* Characters and bytevectors *)
character      = "#\\" , ( letter | digit | "space" | "tab" | "newline" ) ;
bytevector     = "#u8(" , _ , [ byte , { _ , byte } ] , _ , ")" ;
byte           = digits ;                       (* semantic check: 0..255 *)

(* Lexical basics *)
digits         = digit , { digit } ;
digit          = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
letter         = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
               | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
               | "U" | "V" | "W" | "X" | "Y" | "Z"
               | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
               | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
               | "u" | "v" | "w" | "x" | "y" | "z" ;

(* Skippers *)
_              = { whitespace | comment } ;
whitespace     = " " | "\t" | "\n" | "\r" ;
comment        = "#" , { ? any char except newline ? } , ( "\n" | "\r\n" | ? EOF ? )
               | block_comment ;
block_comment  = "#|" , { block_char | block_comment } , "|#" ;
block_char     = ? any char except the sequences "#|" and "|#" ? ;
```

Notes:
- Enforce base-specific digits for `rdigit` in the lexer when a non-decimal radix prefix is present.
- Bar-escaped symbols preserve exact spelling; `sym_escape` permits hex escapes.
- `map` commas are separators only; treat them as optional whitespace if desired.

### Examples

```
# atoms
:ok :max-depth 3
|Case Sensitive| |has space|
#xFF #b1010_1111 #e10 #i3.14
#\x41  #u8(1 255 0 16)

# compound
({:a 1, :b 2} #{1 2 3})
'(a b . c)  `,(+ 1 2)
# vectors, dotted pairs, nested maps/sets
#(1 2 3)   (a . b)
{ :user {:id 1, :name |Ada Lovelace|} }
#{ :a :a :b }   ; decide duplication semantics
# keywords as map keys, escaped symbols
{ :file-name |a:b|, :Π 3.14 }
|\x41|  ; becomes symbol "A"
```

---

## Profile B — Application + Infix (minimal precedence + pipeline)
<a id="profile-b"></a>

Depends on: 03, 10, 12, 13

```ebnf
(* Skippers *)
_            = { whitespace | comment } ;
whitespace   = " " | "\t" | "\n" | "\r" ;
comment      = "#" , { ? any char except newline ? } , ( "\n" | "\r\n" | ? EOF ? ) ;

(* Expressions *)
expr         = pipeline ;

pipeline     = add , { _ , "|>" , _ , add } ;             (* left-assoc *)
add          = mul , { _ , ( "+" | "-" ) , _ , mul } ;    (* left-assoc *)
mul          = app , { _ , ( "*" | "/" ) , _ , app } ;    (* left-assoc *)

(* Application parsed as list then folded right post-parse *)
app          = rexpr , { _ , rexpr } ;
rexpr        = atom | "(" , _ , expr , _ , ")" ;

atom         = identifier | number | string | boolean ;

(* Lexemes *)
identifier   = letter , { letter | digit | "_" | "-" | "?" | "!" } ;
number       = [ "+" | "-" ] , digit , { digit } , [ "." , digit , { digit } ] ;
string       = '"' , { string_char } , '"' ;
string_char  = escape | ? any char except " and \\ ? ;
escape       = "\\" , ( "\\" | '"' | "n" | "t" | "r" ) ;
boolean      = "true" | "false" ;
letter       = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
             | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
             | "U" | "V" | "W" | "X" | "Y" | "Z"
             | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
             | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
             | "u" | "v" | "w" | "x" | "y" | "z" ;
digit        = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
```

Desugaring:
- Application: `a b c` ⇒ `a (b c)` (fold-right)
- Infix: `a + b` ⇒ `(+ a b)`; `a |> f` ⇒ `f a`; `a |> f x` ⇒ `f x a`

Options:
- Add prefix/postfix (see 11) by inserting the `prefix`/`postfix` tier between `mul` and `app`.

### Examples

```
f x + g y            ; (+ (f x) (g y))
data |> map f |> sum ; sum (map f data)
(a + b) * c          ; parentheses override minimal precedence
f (g x) + h (k y z)  ; (+ (f (g x)) (h (k y z)))
x |> (g y) |> h      ; h (g y x)
```

---

## Profile C — Indentation Blocks (offside rule)
<a id="profile-c"></a>

Depends on: 04, 14, 15, 16

```ebnf
(* LEXER SUMMARY — offside rule *)
NEWLINE     = '\n' | '\r\n' ;                   (* only when not inside ( ) [ ] { } *)
INDENT      = ? emitted when current line’s indent > previous indent ? ;
DEDENT      = ? emitted (possibly multiple times) when indent < previous indent ? ;

(* Trailing-operator join: if a physical line ends with opchar, suppress NEWLINE *)
opchar      = "!" | "$" | "%" | "&" | "*" | "+" | "-" | "/" | ":" | "<"
            | "=" | ">" | "?" | "@" | "^" | "|" | "~" ;

(* PARSER GRAMMAR *)
program     = { sep } , [ stmts ] , { sep } ;
sep         = NEWLINE , { NEWLINE } ;

stmts       = stmt , { sep , stmt } , [ sep ] ;

stmt        = simple_stmt
            | if_stmt | while_stmt | for_stmt | def_stmt ;

suite       = NEWLINE , INDENT , stmts , DEDENT
            | _ , "->" , _ , small ;          (* single-line suite *)

if_stmt     = "if" , _ , expr , [ _ , ":" ] , suite ,
              { NEWLINE , "elif" , _ , expr , [ _ , ":" ] , suite } ,
              [ NEWLINE , "else" , [ _ , ":" ] , suite ] ;

while_stmt  = "while" , _ , expr , [ _ , ":" ] , suite ;
for_stmt    = "for" , _ , target , _ , "in" , _ , expr , [ _ , ":" ] , suite ;

def_stmt    = "def" , _ , identifier , _ ,
              "(" , [ params ] , ")" , [ _ , ":" ] , suite ;

simple_stmt = small , { _ , ";" , _ , small } ;
small       = assignment | return | pass | expr ;
assignment  = target , _ , "=" , _ , expr ;
return      = "return" , [ _ , expr ] ;
pass        = "pass" ;

target      = identifier | "(" , _ , target , { _ , "," , _ , target } , _ , ")" ;
params      = param , { _ , "," , _ , param } ;
param       = identifier , [ _ , "=" , _ , expr ] ;

identifier  = letter , { letter | digit | "_" } ;
letter      = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
            | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
            | "U" | "V" | "W" | "X" | "Y" | "Z"
            | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
            | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
            | "u" | "v" | "w" | "x" | "y" | "z" ;
digit       = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;

_           = { " " | "\t" } , [ "#" , { ? any char except newline ? } ] ;
```

Notes:
- The lexer emits NEWLINE/INDENT/DEDENT as described; parser consumes them.
- Optional colons after headers and single-line suites are supported.
- Trailing-operator join suppresses NEWLINE when a line ends with `opchar`.

### Examples

```
def fib(n)
    if n <= 1
        return n
    else
        return fib(n-1) + fib(n-2)

if x > 0 -> return x

a +      # trailing operator join
    b

while pred x
    handle x
    if done x -> return result
```
