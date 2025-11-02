# Infix Operators EBNF (No Precedence)

This document defines a compact **syntax-sugar EBNF** for a surface language that adds **infix operators** to a core where function application is by juxtaposition, **right-associative**, and there are **no precedence levels**.

The safe default below makes infix **non-associative** (so `a + b + c` is illegal unless parenthesized) and **does not give infix tighter/looser binding than application**—you must parenthesize when mixing.

```ebnf
(* Skippers *)
_          = { whitespace | comment } ;
whitespace = " " | "\t" | "\n" | "\r" ;
comment    = "#" , { ? any char except newline ? } , ( "\n" | "\r\n" | ? EOF ? ) ;

(* Program *)
program    = _ , { expr , _ } ;

(* Expressions *)
expr       = infix | app ;

(* Infix (non-associative): exactly one operator without chaining *)
infix      = operand , _ , operator , _ , operand ;

(* Application (right-associative, by juxtaposition) *)
app        = operand , [ _ , app ] ;

(* Operands in infix/application *)
operand    = atom
           | "(" , _ , expr , _ , ")" ;

atom       = identifier | number | string | boolean ;

(* Operator token: one-or-more operator chars *)
operator   = opchar , { opchar } ;
opchar     = "!" | "$" | "%" | "&" | "*" | "+" | "-" | "/" | ":" | "<"
           | "=" | ">" | "?" | "@" | "^" | "|" | "~" ;  (* exclude parentheses and quotes *)

(* Minimal lexemes — tweak as needed *)
identifier = letter , { letter | digit | "_" | "-" | "?" | "!" } ;
number     = [ "+" | "-" ] , digit , { digit } , [ "." , digit , { digit } ] ;
string     = '"' , { string_char } , '"' ;
string_char= escape | ? any char except " and \\ ? ;
escape     = "\\" , ( "\\" | '"' | "n" | "t" | "r" ) ;
boolean    = "true" | "false" ;

letter     = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
           | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
           | "U" | "V" | "W" | "X" | "Y" | "Z"
           | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
           | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
           | "u" | "v" | "w" | "x" | "y" | "z" ;
digit      = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
```

## Desugaring (to the core with right-assoc application)

- `a op b`  ⇒  `((op a) b)`  
  (So `+ 1 2` and `1 + 2` both end up as the same AST: `App(App(+, 1), 2)`.)
- Mixing requires parentheses:
  - `(f x) + y` is OK ⇒ `((+ (f x)) y)`
  - `f x + y` is **not allowed** without parentheses under this no-precedence policy.

---

## Variants (choose one if preferred)

### 1) Allow chaining, all operators same level, **left-associative**

Replace `infix` with:

```ebnf
infix = operand , { _ , operator , _ , operand } ;
```

Desugar a chain by **fold-left**:

```
a1 op1 a2 op2 a3 ...  ⇒  (((a1 op1 a2) op2 a3) ...)
                     ⇒  App(App(op2, App(App(op1, a1), a2)), a3) ...
```
This truly has **no precedence**: `a + b * c` desugars as `((a + b) * c)`.

### 2) Per-operator associativity without precedence

Keep the grammar from Variant 1, but during desugaring:

- Group runs of the **same** operator by the operator’s declared associativity (e.g., `+` left, `^` right).
- When **different operators** appear adjacent, require parentheses (otherwise error).

This keeps “no precedence levels” while still letting you write `a - b - c` as `(a - b) - c`, but forces `a + b * c` to be parenthesized.

---

## Examples

- `1 + 2` ⇒ `((+ 1) 2)`
- `(f x) * (g y)` ⇒ `((* (f x)) (g y))`
- `a + b + c`
  - **Non-assoc default:** error; write `(a + b) + c` or `a + (b + c)`.
  - **Left-assoc variant:** `((a + b) + c)`.

---

## Notes

- The grammar is LL(1)-friendly except for the `app` right recursion; if desired, parse a **nonempty list of `operand`** then fold-right into applications in a post-parse step.
- Extend `atom`/lexemes to your needs (booleans, identifiers, Unicode ops).  
- If you add prefix/postfix operators, give them *explicit* nonterminals and keep the "no precedence" rule by requiring parentheses when mixing with infix/application.
