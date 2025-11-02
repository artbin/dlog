# Indentation: Optional Colons After Headers

Extends `04-indentation-ebnf.md` to allow optional `:` after compound-statement headers.

Depends on: `04-indentation-ebnf.md` (offside rule and suites)

```ebnf
(* Replacements in the parser grammar of 04 *)

if_stmt       = "if" , _ , expr , [ _ , ":" ] , suite ,
                { NEWLINE , "elif" , _ , expr , [ _ , ":" ] , suite } ,
                [ NEWLINE , "else" , [ _ , ":" ] , suite ] ;

while_stmt    = "while" , _ , expr , [ _ , ":" ] , suite ;

for_stmt      = "for" , _ , target , _ , "in" , _ , expr , [ _ , ":" ] , suite ;

def_stmt      = "def" , _ , identifier , _ ,
                "(" , [ params ] , ")" , [ _ , ":" ] , suite ;
```

Note:
- Lexing and INDENT/DEDENT behavior remain unchanged.

## Examples

```
if x > 0:            
    return x

def f(n):            
    if n == 0        
        return 0     
    else:            
        return f(n-1)
```

## Integration Notes

- The colon is purely syntactic sugar; it does not change where `suite` begins.
- You may forbid `:` inside parentheses continuation lines to avoid ambiguity.
