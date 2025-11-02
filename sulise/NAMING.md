# Sulise Language - Naming Document

> **Sulise** - A homoiconic programming language with dual syntax: canonical S-expressions and ergonomic surface forms.

## Name

**Sulise** (pronounced "soo-lease" or "soo-lees")

## Etymology

The name **Sulise** is a portmanteau combining:

- **Su** - from **Surface** (the ergonomic, human-friendly syntax layer)
- **Lise** - from **Lisp** / **S-expression** (the canonical, homoiconic core)

This reflects the language's fundamental architecture: surface syntaxes that desugar to a canonical S-expression core, preserving homoiconicity while providing modern ergonomic conveniences.

## File Extensions

- **Primary**: `.sul` - Clean, minimal, easy to type

## Why Sulise?

### Technical Alignment

1. **Dual Nature** - The name captures both aspects of the language:
   - Surface syntax (infix, pipelines, indentation)
   - S-expression core (homoiconic, macro-friendly)

2. **Homoiconicity Preserved** - Like its namesake implies, Sulise maintains the Lisp tradition of code-as-data while providing modern surface conveniences.

3. **Profile System** - Sulise supports three profiles:
   - **Profile A** (S-expr): Pure S-expressions for macro systems
   - **Profile B** (Surface): Infix, pipelines, minimal precedence
   - **Profile C** (Indentation): Python-style offside rule

### Practical Benefits

✅ **Memorable** - Short, distinctive, easy to recall  
✅ **Pronounceable** - Natural phonetics in multiple languages  
✅ **Professional** - Modern programming language aesthetic  
✅ **Unique** - No major conflicts with existing languages  
✅ **Searchable** - Distinctive enough for effective web searches  
✅ **Domain Available** - Strong availability for project domains

## Design Philosophy Reflected in Name

The name **Sulise** embodies the language's core philosophy:

1. **Surface over Core** - Like a beautiful facade over strong foundations, Sulise provides elegant surface syntax backed by a robust S-expression core.

2. **Choice and Flexibility** - The name doesn't favor one style over another, just as Sulise lets developers choose their preferred syntax profile.

3. **Heritage and Innovation** - Honors the Lisp tradition while embracing modern language design.

## Usage Examples

### Code Examples

```sulise
# Profile A - Pure S-expressions
(define factorial (n)
  (if (<= n 1)
      1
      (* n (factorial (- n 1)))))
```

```sulise
# Profile B - Surface syntax with infix
define factorial n =
  if n <= 1 then 1
  else n * factorial (n - 1)
```

```sulise
# Profile C - Indentation-based
define factorial n
  if n <= 1 -> 1
  else -> n * factorial (n - 1)
```

### Project References

- **Repository**: `sulise-lang`
- **Parser/Implementation**: `sulise`
- **Documentation**: "The Sulise Language Specification"
- **Community**: "Sulise developers" / "Sulise community"

## Branding and Identity

### Tagline Options

- "Sulise - Surface meets S-expression"
- "Sulise - Homoiconic by design, ergonomic by choice"
- "Sulise - Write code your way, run it one way"
- "Sulise - Modern syntax, timeless foundations"

## Community and Marketing

### Elevator Pitch

"Sulise is a homoiconic programming language that combines the power of S-expressions with modern surface syntax. Write with infix notation, pipelines, and indentation—or pure S-expressions. Your choice of syntax, one canonical representation."

### Target Audiences

1. **Lisp enthusiasts** - Familiar S-expression core with modern conveniences
2. **Python/Ruby developers** - Approachable syntax with powerful macros
3. **Functional programmers** - Homoiconicity meets modern ergonomics
4. **Language designers** - Modular grammar toolkit and clear desugaring contract
