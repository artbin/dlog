# Python-Style Indentation EBNF (No Colons)

This document defines a compact EBNF for a block-structured language that uses **Python-style indentation (offside rule)** to start/end suites **without colons**. `INDENT`/`DEDENT`/`NEWLINE` are *tokens produced by the lexer* from leading whitespace.

```ebnf
(* ---------- LEXING / OFFSIDE RULE ---------- *)

(* Physical characters *)
hspace      = " " | "\t" ;
newline_ch  = "\n" | "\r\n" ;

(* Comments *)
comment     = "#" , { ? any char except newline ? } ;

(* Logical NEWLINEs *)
NEWLINE     = newline_ch ;                 (* only when not inside ( ) [ ] { } *)

(* Indentation tokens (inserted by the lexer at line starts, not written literally) *)
INDENT      = ? emitted when current line’s indent > previous indent ? ;
DEDENT      = ? emitted (possibly multiple times) when current line’s indent < previous indent ? ;

(* Offside rule (lexer behavior, prose):
   - Keep a stack of indent widths; start with [0].
   - At the start of a logical line (after a NEWLINE), count leading hspace.
     Tabs are illegal or expand to fixed width; pick one policy (spaces-only recommended).
   - If width == top, emit nothing.
     If width  > top, push width; emit INDENT.
     If width  < top, pop until top == width; emit a DEDENT for each pop.
   - Blank or comment-only lines: emit NEWLINE but do NOT change indent stack.
   - Inside ( ) [ ] { }, suppress NEWLINE/INDENT/DEDENT (“implicit line joining”).
   - At EOF, emit DEDENT until the stack returns to [0].
*)

(* ---------- PARSER GRAMMAR ---------- *)

program     = { sep } , [ stmts ] , { sep } ;
sep         = NEWLINE , { NEWLINE } ;

stmts       = stmt , { sep , stmt } , [ sep ] ;

stmt        = simple_stmt
            | if_stmt
            | while_stmt
            | for_stmt
            | def_stmt
            | block_stmt ;                  (* optional catch-all, see below *)

suite       = NEWLINE , INDENT , stmts , DEDENT ;

(* ----- Compound statements (no colons) ----- *)

if_stmt     = "if" , _ , expr , suite ,
              { NEWLINE , "elif" , _ , expr , suite } ,
              [ NEWLINE , "else" , suite ] ;

while_stmt  = "while" , _ , expr , suite ;

for_stmt    = "for" , _ , target , _ , "in" , _ , expr , suite ;

def_stmt    = "def" , _ , identifier , _ ,
              "(" , [ params ] , ")" , suite ;

(* Optional generic “header then suite” form you can reuse for other keywords *)
block_stmt  = block_kw , [ _ , header_tail ] , suite ;
block_kw    = "try" | "with" | "match" | "namespace" ;   (* extend as desired *)
header_tail = expr | target | param_list | pattern ;     (* pick per keyword *)

(* ----- Simple statements (single line) ----- *)

simple_stmt = small , { _ , ";" , _ , small } ;          (* optional ; to chain *)
small       = assignment | return | pass | expr ;

assignment  = target , _ , "=" , _ , expr ;
return      = "return" , [ _ , expr ] ;
pass        = "pass" ;

target      = identifier | "(" , _ , target , { _ , "," , _ , target } , _ , ")" ;

params      = param , { _ , "," , _ , param } ;
param       = identifier , [ _ , "=" , _ , expr ] ;

identifier  = letter , { letter | digit | "_" } ;

(* ----- Expressions: plug in your own grammar ----- *)
expr        = <your expression grammar here> ;

(* Lexical trivia inside a line *)
_           = { hspace } , [ comment ] ;
letter      = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
            | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
            | "U" | "V" | "W" | "X" | "Y" | "Z"
            | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
            | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
            | "u" | "v" | "w" | "x" | "y" | "z" ;
digit       = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
```

## What “no colons” means
A **suite** starts immediately after the header’s end-of-line:  
`if expr` **NEWLINE** **INDENT** ... **DEDENT**  
(same for `def`, `while`, `for`, etc.).  
`elif`/`else` must appear **after the preceding suite’s DEDENT** and at the same indentation level as the `if` header.

## Minimal example (valid without colons)
```
def fib(n)
    if n <= 1
        return n
    else
        a = fib(n - 1)
        b = fib(n - 2)
        return a + b

for x in data
    if pred x
        handle x
    else
        pass
```

## Notes & options
- **Tabs vs spaces:** simplest is “spaces only”; otherwise define a fixed tab width (e.g., 8) and convert during lexing.
- **Blank lines / comments:** can appear anywhere; they don’t affect indentation.
- **Implicit line joining:** inside `() [] {}` you can wrap lines freely with no NEWLINE/INDENT/DEDENT tokens.
- **Chaining simple statements:** the optional `;` allows multiple small statements on one physical line.
- **Expressions:** reuse your prior expression grammar (e.g., right-associative application or no-precedence infix) by plugging it into `expr`.
