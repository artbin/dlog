# Prefix and Postfix Operators EBNF

Introduces prefix/postfix operators with a simple precedence: `application > prefix/postfix > infix`.

Depends on: `03-infix-operators-ebnf.md` (operator/lexeme base)

```ebnf
(* Skippers as in 03 *)
_            = { whitespace | comment } ;

expr         = infix ;

(* Infix at lowest level, left-associative single tier *)
infix        = prefix , { _ , operator , _ , prefix } ;

(* Prefix: right-associative chain of prefix ops *)
prefix       = { prefix_op , _ } , postfix ;

(* Postfix: left-associative chain of postfix ops *)
postfix      = app , { _ , postfix_op } ;

(* Application binds tighter than prefix/postfix; fold-right after parse *)
app          = rexpr , { _ , rexpr } ;

rexpr        = atom | "(" , _ , expr , _ , ")" ;

atom         = identifier | number | string | boolean ;

operator     = opchar , { opchar } ;
opchar       = "!" | "$" | "%" | "&" | "*" | "+" | "-" | "/" | ":" | "<"
             | "=" | ">" | "?" | "@" | "^" | "|" | "~" ;

prefix_op    = "-" | "!" | "~" ;
postfix_op   = "?" | "!" ;
```

Desugaring:
- Prefix: `- - x` ⇒ `neg (neg x)` or `(- (- x))`
- Postfix: `x ! ?` ⇒ `(? (! x))` (left fold for postfix into nested applications)

## Examples

```
!-x            ; ⇒ (! (- x))
f x! + g ~y    ; application binds tight: ( (+ (f (x!)) (g (~ y))) )
(~ - a) ?      ; ⇒ (? (~ (- a)))
```

## Integration Notes

- If `!` is both an operator and part of identifiers in your core, restrict `postfix_op`.
- Prefix chains are naturally right-associative; ensure your AST mirrors nesting order.
- Consider adding unary `+` if desired; keep it distinct from binary `+`.
