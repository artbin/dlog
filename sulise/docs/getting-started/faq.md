# FAQ

## Table of Contents

- Profiles and Concepts
- Surface/Core and Evaluation
- Expressions and Operators
- Indentation and Tokens
- ISO EBNF and Style
- Examples and Diagrams
- Customization and Composition
- Troubleshooting
- References and Contribution

## Profiles and Concepts

### What are Profiles A/B/C?
- A: Core S-expressions only. Best for homoiconic workflows and macros. See `sexpr.md` and `profiles.md` → Profile A.
- B: Surface with application + optional infix, prefix/postfix, pipeline `|>`. See `profiles.md` → Profile B.
- C: Surface with indentation/offside blocks; optional colons and one‑line suites. See `profiles.md` → Profile C.

### Do Surface forms round‑trip?
- Not textually. Surface parses are desugared to canonical S‑expressions, then macroexpanded and evaluated. The S‑expression AST is the round‑trippable form. See `surface.md` and `sexpr.md`.

## Surface/Core and Evaluation

### What is the evaluation pipeline?
- Parse Surface → Desugar → Canonical S‑expressions → Macroexpand → Evaluate. See `README.md` → Homoiconicity and `DIAGRAMS.md`.

## Expressions and Operators

### How do application and infix interact?
- Base infix (03): no precedence; parenthesize when mixing.
- With 10: application binds tighter than infix, so `f x + g y` → `(+ (f x) (g y))`.
- Minimal precedence (12): two tiers (*/ above +/-). See `COOKBOOK.md` and `profiles.md`.

### Is application right‑associative?
- Yes. `a b c` desugars to `a (b c)` post‑parse. See `02-right-assoc-app-ebnf.md`.

### How does the pipeline operator work?
- `x |> f` → `(f x)`; chains left‑to‑right threading the left value as the last argument. See `13-pipeline-operator-ebnf.md` and `COOKBOOK.md`.

## Indentation and Tokens

### How does indentation (offside rule) work?
- In 04/14–16, the lexer, not the grammar, emits `NEWLINE`/`INDENT`/`DEDENT`.
- 14 adds optional `:`; 15 enables single‑line suites `->`; 16 allows implicit join when a line ends with an operator. See `04-*.md`, `14–16-*.md` and `COOKBOOK.md`.

### What are the comment forms?
- Line comments: `#` … up to `"\n" | "\r\n" | ? EOF ?`.
- Nested block comments: `#| ... |#` when 07 is included. See `01-*.md` and `07-block-comments-ebnf.md`.

## ISO EBNF and Style

### Are the grammars ISO/IEC 14977 EBNF compliant?
- Yes. Terminals are double‑quoted; metasymbols `, | ; ( ) [ ] { }` are used per ISO.
- Prose/lexer notes appear only inside `? ... ?`. No Unicode ellipses; ranges are explicit. See `README.md` → ISO EBNF conformance.

### What belongs in the lexer vs grammar?
- Lexer: indentation tokens, numeric base validation, byte ranges, newline consumption for comments.
- Grammar: structure, operator forms, token sequencing. See `README.md` → Composition guidance.

## Examples and Diagrams

### Where are types and error handling?
- Planned high‑level constructions (VB‑style `as`/`of`, Option/Result, Zig‑style errors) live in `plan.md` tasks. They are not part of the 01–16 baseline yet.

### Where can I find examples?
- `COOKBOOK.md` has copyable before/after snippets; `profiles.md` shows per‑profile examples; `DIAGRAMS.md` visualizes parse → desugar flows.

## Customization and Composition

### How do I extend the grammar set?
- Add a small, focused EBNF doc with rationale, examples, desugaring, and dependencies.
- Follow ISO rules (terminals quoted; prose in `? ... ?`), and document Surface→Core mapping.
- See `plan-docs.md` for documentation tasks and structure to keep navigation consistent.

### How do I choose between Profiles A/B/C?
- Prefer A when you want pure S-expressions and maximal macro power/homoiconicity.
- Prefer B for ergonomic expressions (infix/pipeline) that still desugar to core.
- Prefer C when indentation readability and lightweight blocks are desired.

### Why unify line comments to `#`?
- Consistency across profiles, no clash with `;` in EBNF, and easy lexer handling.
- The rule consumes to `"\n" | "\r\n" | ? EOF ?` to avoid dangling comments. See `README.md` → ISO EBNF conformance.

### Where are string/escape rules defined?
- Canonical forms live in `01-sexpr-ebnf.md` and are reused by extensions (`05`, `06`).
- Keep prose like EOF/newline only inside `? ... ?` special sequences.

### Can I use nested block comments?
- Yes, when `07-block-comments-ebnf.md` is included: `#| ... |#` with nesting. Line comments `#` remain unchanged.

### Where do precedence and associativity come from?
- `03` has no precedence; `10` makes application tighter than infix; `12` adds two tiers (+/- vs */). See `profiles.md` and `COOKBOOK.md`.

### Who emits `NEWLINE`/`INDENT`/`DEDENT`?
- The lexer. The grammar consumes these tokens (04/14–16) but does not compute indent levels itself. See `04-indentation-ebnf.md`.

### Does Surface change macroexpansion semantics?
- No. Desugaring yields canonical S-expressions; macroexpansion operates on core only. See `sexpr.md` and `surface.md`.

## Troubleshooting

### How can I test a composition quickly?
- Use examples in `profiles.md` and `COOKBOOK.md` as fixtures; verify your desugaring matches the shown S-expressions.
- Cross-check flows in `DIAGRAMS.md` (parse → desugar) for tricky mixes (pipeline × infix × application).

### Where are numeric literal details?
- `06-numbers-radix-exactness-ebnf.md` specifies radix/exactness prefixes and `_` separators; validation of digit sets is a lexer concern.

### How do I build a custom profile?
- Start with Profile A (core) or B/C (surface) in `profiles.md`.
- Add atoms/literals (05–09) and expression/indentation variants (10–16) as needed.
- Ensure the lexer emits `NEWLINE`/`INDENT`/`DEDENT` if using 04/14–16.
- Validate ISO shape (no Unicode ellipses; prose only inside `? ... ?`).
- Test with examples from `COOKBOOK.md` and compare desugaring to the shown S‑expressions.

### Where is the operator precedence/associativity reference?
- See 10–12 docs and examples in `COOKBOOK.md`.
- A compact operator table is planned in `plan-docs.md` → Summaries and matrices (to be published in `tables/operators.md`).

### How are keywords and operator characters tracked?
- `05` defines keywords and bar‑escaped symbols. Use `|bar-escaped|` to avoid conflicts.
- A consolidated reference is planned in `plan-docs.md` → Reference (to be published in `reference/keywords-and-operators.md`).

### Are Unicode identifiers supported?
- Baseline examples use ISO‑friendly explicit ranges. For broader identifier sets, prefer lexer‑level handling or bar‑escaped symbols. Keep the EBNF ISO‑clean.

### How do I debug parsing or desugaring issues?
- Step through: tokens → grammar rule → desugar mapping (see each 01–16 doc and `COOKBOOK.md`).
- Check indentation token emission and comment consumption first.
- Use `DIAGRAMS.md` to visualize parse → desugar flows for tricky mixes (pipeline × infix × application).

### Are macros and hygiene included?
- Not in the 01–16 baseline. They’re planned as extensions in `plan.md` with notes on hygiene and expansion over the core.

### Where do I see dependencies between features?
- `plan.md` lists per‑task dependencies and grouped indexes; `DIAGRAMS.md` includes dependency graphs by family (types, errors, async).

## References and Contribution

### What should I read first?
- `profiles.md` (Profiles A/B/C examples) → `README.md` (Quickstart, How to use) → `COOKBOOK.md` (copyable recipes) → `DIAGRAMS.md` (flows). For definitions, see `GLOSSARY.md`.

### Are links and anchors stable?
- Yes; task anchors follow `#task-<num>-<slug>` in `plan.md`. Tables and examples will include anchors as they are added. See `plan-docs.md` for navigation tasks.

### What does `_` mean in rules and examples?
- `_` is used in docs to denote intra-line trivia spacing in examples; it is not a terminal unless explicitly defined.

### Are identifiers case-sensitive?
- Typically yes; specifics depend on your lexer. The EBNF stays neutral; define case policy at the lexer layer.

### How do EOF and newline appear in EBNF?
- Only inside `? ... ?` special sequences (e.g., `? EOF ?`) to remain ISO-conformant.

### Can I use a parser generator with these grammars?
- Yes—ensure terminals and special sequences are translated appropriately, and handle indentation tokens and comment consumption in the lexer.

### Is there an automated ISO validation?
- A script is planned in `plan-docs.md` (Validation and CI) to check for Unicode ellipses, comment shape, and special sequence usage across docs.

### Are the grammars executable as-is?
- They are specification-first. Adapt terminals/special sequences to your parser generator and implement indentation/comment/token rules in the lexer.

### How do I disable a Surface feature?
- Compose by omission: exclude the corresponding 10–16 (or 05–09) docs from your profile. Your lexer should then reject tokens not produced by the chosen grammar.

### Can I customize operator precedence beyond the minimal tier?
- Not in the baseline. See planned extensions in `plan.md`; a consolidated operator table is tracked in `plan-docs.md`.

### How should errors refer to Surface vs Core?
- Report diagnostics on Surface source spans (pre‑desugar) while retaining mappings to the Core S‑expression AST for tooling.

### Where do new diagrams go?
- Add them to `DIAGRAMS.md` and link back from the relevant docs section. Keep per‑family graphs compact.

### How do I rename/add a nonterminal?
- Update the source doc with a stable anchor, then reflect it in `reference/grammar-index.md` (see `plan-docs.md`). Avoid breaking existing anchors.

### Are examples ASCII-only?
- Yes, for ISO readability. For Unicode identifiers/operators, use bar‑escaped symbols or handle at the lexer level.

### What’s the license?
- MIT (see `README.md`).

### How can I contribute?
- Open a PR. Follow ISO EBNF rules and the docs style (see `plan-docs.md`; forthcoming `STYLE-EBNF.md`). Include desugaring and examples.
