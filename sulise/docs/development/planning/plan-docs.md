# Documentation Improvement Plan

This plan outlines concrete steps to improve navigation, reference depth, examples, and validation of the docs. It complements `plan.md`, `ROADMAP.md`, `DIAGRAMS.md`, `BACKLOG.md`, and `FEATURES.md`.

## Goals
- Make complex docs quickly navigable (mini-TOCs, anchors, tag index).
- Provide matrices/tables for at-a-glance feature availability and operators.
- Add end-to-end examples and before/after desugar galleries.
- Establish a repeatable authoring style and validation checks.

## Navigation
- Quick links + global TOC at the top of long docs (done for `plan.md`).
- Mini-TOCs per section (High-Level, Extensions) (done).
- Stable per-task anchors: `#task-<num>-<slug>` (in progress; extend to Extensions).
- Back-to-top/index links at section ends (added to key sections; extend across file).
- Profile landing page: add to `FEATURES.md` with links to A/B/C examples.

Deliverables:
- Extend anchors/back-links across all task blocks in `plan.md`.
- Add a short profile selector to `README.md` and `FEATURES.md`.

## Summaries and matrices
- Feature matrix per profile (A/B/C): rows = features; cols = profiles; cells link to tasks or 17-*.md.
- Operator table: token, precedence, associativity, tier, profile availability.

Deliverables:
- `tables/features-matrix.md`
- `tables/operators.md`

Links:
- See also `FEATURES.md` → Navigation → Feature matrix and Operators table.

Tasks:
- Define feature taxonomy and profile mapping (A/B/C) aligned with `FEATURES.md` sections.
- Create `tables/features-matrix.md` skeleton: headers `Feature | A | B | C | Notes` with anchor IDs.
- Populate rows; link each cell to `plan.md` task anchors, `01–16-*.md` files, or `profiles.md` profiles.
- Add status marks per cell (Supported / Partial / Planned); include brief Notes with constraints.
- Insert the matrix via links in `FEATURES.md` and reference it from README “Features overview”.
- Create `tables/operators.md` with headers `Token | Kind | Precedence | Associativity | Profiles | Link`.
- Inventory operators from 03, 10–13, 11, 12; normalize names/tokens; add links to defining docs.
- Validate operator tiers and associativity; ensure consistency with “application tighter than infix”.
- Add anchors to both tables and include them in the Docs TOC; cross-link from `plan.md` operator tasks.
- Integrate link checks for both tables into `scripts/validate_docs.py` (no-network link check).

## Examples
- Quickstarts: parse → desugar → macroexpand → eval for Profiles A/B/C.
- Before/after desugaring galleries per feature (short, copyable snippets).
- Round-trips (surface → S-expr → print) highlighted in each `17-*.md`.

Deliverables:
- `examples/profile-a.md`, `examples/profile-b.md`, `examples/profile-c.md`
- Add “Examples” section to each new `17-*.md`.

Tasks:
- Create `examples/profile-a.md` with 3–5 S-expr-only snippets and evaluation notes.
- Create `examples/profile-b.md` showing infix, prefix/postfix, minimal precedence, and pipelines with desugaring.
- Create `examples/profile-c.md` with indentation headers, optional colons, single-line suites, and line-join examples + desugaring.
- For each example, include: Surface input, S-expression desugar, and a brief rationale.
- Add a “before → after” gallery per major feature (03, 10–13, 14–16) with copyable code blocks.
- Add round-trip notes (surface → S-expr → pretty-print) and caveats about textual non-round-trips.
- Cross-link examples to `plan.md` task anchors and `profiles.md` profile sections; add back-to-top links.
- Reference `sexpr.md` and `surface.md` in example intros to reinforce the contract.
- Ensure examples avoid non-ISO constructs; rely on terminals and `? ... ?` only for prose.
- Add example anchors and include them in a mini-TOC at top of each examples page.

## Reference
- Grammar index: list nonterminals and link to defining docs.
- Reserved keywords and `opchar` by profile; note conflicts and policy toggles.

Deliverables:
- `reference/grammar-index.md`
- `reference/keywords-and-operators.md`

Tasks:
- Create `reference/grammar-index.md` skeleton with headers `Nonterminal | File | Section | Profiles | Notes`.
- Inventory nonterminals from `01–16-*.md` and future `17-*.md` feature docs; deduplicate and normalize names.
- Add stable anchors near each nonterminal rule in the source docs; link rows to those anchors.
- Mark profile applicability (A/B/C) per row; add short Notes when behavior differs by profile.
- Create `reference/keywords-and-operators.md` with tables:
  - `Keyword | Profiles | Meaning | Link` and `Operator/Opchar | Kind | Profiles | Link | Notes`.
- List reserved keywords and operator characters by profile; document conflicts and policy toggles.
- Cross-link reference tables to `plan.md` task anchors and `profiles.md` profile sections.
- Add back-to-top/index links and a mini‑TOC to both reference pages; include tag badges.
- Integrate link checks for both reference pages into `scripts/validate_docs.py` (no-network).

## Style and contribution
- Authoring template for `17-*.md` (EBNF blocks, desugaring, examples, dependencies, tags, homoiconicity note).
- EBNF styleguide: quoting, `? ... ?` specials, comment rules, string/escape canon, line comments `#`.

Deliverables:
- `CONTRIBUTING.md` (docs authoring section)
- `docs-template-17.md`
- `STYLE-EBNF.md`

Tasks:
- Draft `docs-template-17.md` sections: Overview; Rationale; EBNF (ISO‑clean); Desugaring (Surface→Core); Examples (before/after); Dependencies; Profiles/Tags; Anchors; Links; “Back to top”.
- Write `STYLE-EBNF.md`: terminals double‑quoted; metasymbols only; prose only in `? ... ?`; unified line comments `#`; nested block comments `#| ... |#` when 07; normalized string/escape; numbers (06); no Unicode ellipses; explicit ranges.
- Create `CONTRIBUTING.md` (docs authoring): PR checklist (ISO checks, anchors, links); anchor stability policy (`#task-<num>-<slug>`); link style (relative paths, stable anchors); examples style (copyable, before→after, no non‑ISO); diagram placement rules.
- Add scripts section to `CONTRIBUTING.md` with usage of `scripts/validate_docs.py` and `scripts/check_iso_shape.py` (to be added in Validation and CI).
- Define profiles/tags badges syntax at top of each doc (e.g., `[Profiles: A,B] [Tags: errors, types]`).
- Specify internal link conventions: relative links; avoid bare URLs; add back‑links to index/TOC.
- Document allowed Markdown: fenced code blocks with language tags; Mermaid allowed; avoid raw HTML where possible.
- Provide anchor naming guidance: lowercase, hyphenated, stable; avoid renaming; add redirects if needed.
- Include “Back to index / Back to top” snippet in `docs-template-17.md` and apply to new docs.

## Diagrams and dependency views
- Per-feature mini-diagram showing parse → desugar mapping; embed in each `17-*.md`.
- Split dependency graphs by families (types, errors, async) in `DIAGRAMS.md`.

Deliverables:
- Update `DIAGRAMS.md` with family diagrams.
- Add Mermaid blocks to `17-*.md` where helpful.

Tasks:
- Define a consistent Mermaid style (node shapes, colors, labels) for parse → desugar flows and dependency graphs.
- Create family diagrams in `DIAGRAMS.md` for: Expressions (03, 10–13), Indentation (04, 14–16), Literals (05–09), and High-level (links to `plan.md`).
- Add “from/to” links in diagram captions to the first relevant task anchor in `plan.md` and to profile anchors in `profiles.md`.
- For each new `17-*.md`, embed a compact per-feature parse → desugar Mermaid diagram at the top of the doc.
- Ensure diagrams use ASCII tokens and avoid non-ISO prose; keep any prose in captions, not in code blocks.
- Add an index of diagrams at the top of `DIAGRAMS.md` with anchors and a mini-TOC.
- Validate Mermaid blocks render (locally) and include link checks in `scripts/validate_docs.py`.

## Validation and CI
- Add link checker, Markdown lint, Mermaid render checks.
- Script to verify ISO shape across docs (no ellipses, unified strings/comments, specials usage).

Deliverables:
- `scripts/validate_docs.py` (no network), `scripts/check_iso_shape.py`
- Optional CI config (document-only for now)

Tasks:
- Implement `scripts/check_iso_shape.py` to fail on:
  - Unicode ellipses `…`; require ASCII and explicit ranges
  - Prose outside `? ... ?` for EOF/newline/lexer notes
  - Non `"..."` terminals; stray single quotes in terminals
  - Non-`#` line comments in examples/EBNF; invalid `#| ... |#` usage without 07
- Implement `scripts/validate_docs.py` to:
  - Check internal links/anchors across docs (README, FEATURES, ebnf, 01–16, plan, DIAGRAMS, COOKBOOK, FAQ, GLOSSARY)
  - Verify `#task-<num>-<slug>` anchors in `plan.md` referenced by other docs exist
  - Ensure Mermaid code fences exist and are balanced; optionally dry-run render (no-network)
  - Run Markdown lint (if available locally) or minimal whitespace checks (no tabs, trailing spaces)
- Add a single entry point (e.g., `just validate_docs.py`) documented in `CONTRIBUTING.md`.
- Optionally add a pre-commit hook snippet in `CONTRIBUTING.md` to run both scripts locally.
- Optionally add a CI workflow (`.github/workflows/docs-ci.yml`) to run both scripts on PRs (document-only step).

## Release hygiene
- Keep CHANGELOG categories consistent; Doc-only tag.
- Version banner in docs (Unreleased vs date) where relevant.

Deliverables:
- Update `CHANGELOG.md` policy section
- Add version banner snippet to top of long docs

## Discoverability
- Expand tag index in `FEATURES.md`; add tags at top of each `17-*.md` (profiles, types, async, errors).
- Add “Search tips” to `README.md`.

Deliverables:
- Tag headers in `17-*.md`
- `SEARCH-TIPS.md`

Tasks:
- Define tag taxonomy: Profiles (A, B, C) and feature families (types, async, errors, literals, expressions, indentation, ergonomics).
- Add a standard tag/badge header snippet to `docs-template-17.md` (e.g., `[Profiles: A,B] [Tags: types, errors]`).
- Build a Tag Index in `FEATURES.md` with anchors (`#tag-<name>`) and per-tag lists linking to the first relevant task anchor in `plan.md` and to key spec files.
- Cross-link `plan.md` Profiles/Tags index to `FEATURES.md` Tag Index; ensure each tag links back to its first occurrence.
- Expand `README.md` Search tips to point to Tag Index and provide quick grep patterns (e.g., `#task-`, `Profiles:`).
- Verify every `17-*.md` includes tag headers; backfill tags on existing docs where clear.
- Add link and anchor checks for Tag Index and tag headers to `scripts/validate_docs.py`.

## Site (optional)
- Publish docs via static site (mkdocs/docusaurus) while keeping repo as source of truth.

Deliverables:
- `docs/` placeholder and site config (optional, later milestone)

## Additional documentation (new files)

Deliverables:
- `CONTRIBUTING.md` (how to propose EBNF docs; PR checklist; anchors/links rules)
- `STYLE-EBNF.md` (ISO rules, quoting, specials, string/escape canon, comments)
- `docs-template-17.md` (template for new feature specs: EBNF, desugar, examples)
- `SEARCH-TIPS.md` (anchors, link patterns, grep recipes, navigation hints)
- `PROFILES.md` (landing for Profiles A/B/C with side-by-side examples)
- `LEXER-GUIDE.md` (NEWLINE/INDENT/DEDENT emission, comments, numeric validation)
- `PARSER-GUIDE.md` (integrating profiles, operator tiers, desugaring pipeline)
- `TESTING.md` (round-trip fixtures, diagram checks, link checks)
- `RELEASE-CHECKLIST.md` (docs banner, matrices, examples, anchors, changelog)
- `OPERATORS-DEEP-DIVE.md` (precedence, associativity, examples, pitfalls, links)
- `TYPES-PLAN.md` (as/of types and sugar; links to tasks 8, 14, 15)
- `ERRORS-PLAN.md` (Zig-style errors; Result/Option; async×errors; tasks 6, 15, 29)
- `EXAMPLES-GUIDE.md` (how to add surface↔sexpr pairs, naming, categories, verification)
- `SITE.md` (optional: publishing via mkdocs/docusaurus; source-of-truth rules)
- `SECURITY.md` and `CODE_OF_CONDUCT.md` (community docs)

Tasks:
- Draft `CONTRIBUTING.md`: scope (EBNF docs), PR checklist (ISO checks, anchors present, links valid), naming and placement, review workflow.
- Author `STYLE-EBNF.md`: terminals double-quoted, `? ... ?` for prose, canonical string/escape, comment rules, no Unicode ellipses, explicit ranges.
- Prepare `docs-template-17.md`: Overview; Rationale; EBNF (ISO-clean); Desugaring (Surface→Core); Examples (before/after); Dependencies; Profiles/Tags; Anchors; Links; Back-to-top snippet.
- Write `SEARCH-TIPS.md`: anchor naming conventions; linking to profile/task anchors; rg recipes; using `ANCHORS.md`.
- Create `PROFILES.md`: describe A/B/C with short side-by-side examples and links to `profiles.md` sections.
- Create `LEXER-GUIDE.md`: token emission for indentation; comment consumption; numeric base validation; integration notes.
- Create `PARSER-GUIDE.md`: operator tiers, application-tighter-than-infix, pipeline associativity, desugar phase.
- Create `TESTING.md`: how to add surface→S-expr fixtures; expected outputs; validation scripts; mermaid checks; link checks.
- Create `RELEASE-CHECKLIST.md`: banner set, matrices updated, examples paired, anchors regenerated, changelog updated.
- Author `OPERATORS-DEEP-DIVE.md`: full table, associativity rules, examples, pitfalls; cross-link to `tables/operators.md` and `reference/keywords-and-operators.md`.
- Draft `TYPES-PLAN.md` and `ERRORS-PLAN.md`: constraints, examples, sequencing; link to plan tasks (8, 14, 15, 6, 29).
- Create `EXAMPLES-GUIDE.md`: file naming, pairing policy, profile/theme categorization, verification.
- Optional `SITE.md`: build and deploy steps; keep repo as canonical source; link hygiene.
- Add `SECURITY.md` and `CODE_OF_CONDUCT.md` (templates acceptable initially).
- Cross-link all new docs from `DASHBOARD.md` and `README.md` Navigation where appropriate.

## Improvements to existing documents

Deliverables:
- README: per-section mini-TOCs for long sections; links to `LICENSE` and `AUTHORS.md`; "Profiles quick compare" table (A/B/C) with tradeoffs and links.
- plan.md: "Done/Closed" appendix linking to implemented tasks; badges/tags (profiles, types, errors, async) on task headings.
- profiles.md: "Common Lexical Snippets" box referencing `STYLE-EBNF.md`; per-profile "Desugar map" mini-table (surface → S-expr) with 2–3 forms.
- DIAGRAMS.md: diagram index at top with one-liner purpose blurbs; per-diagram "Examples" links to files in `examples/`.
- Tables: features/operators—add Status column (Supported/Partial/Planned) and cross-links to examples demonstrating each row.
- Reference: add "Last validated on" at top of `reference/*`; add "See in examples" column; ensure anchors per nonterminal and keyword/operator entries.
- examples/README.md: add "missing pairs" list (generated); "most useful sets" shortlist; footer with validation commands.
- DASHBOARD.md: add "Open PR checklist" box (links to `RELEASE-CHECKLIST.md` and `CONTRIBUTING.md`); optional "Top 5 recently edited docs" list (scripted).
- Scripts: enhance `scripts/generate-status.sh` to include "docs updated since <date>"; enhance `scripts/validate_docs.py` to flag headings missing back-to-top links (advisory).
- STYLE-EBNF.md / CONTRIBUTING.md: add "Common gotchas" section (Unicode ellipsis, prose outside `? ... ?`, mixed quotes); include an example PR diff for a new `17-*` doc.

Tasks:
- README: add mini-TOCs; add links to `LICENSE` and `AUTHORS.md`; create a three-column "Profiles quick compare" with pros/cons and links.
- plan.md: add a "Done/Closed" section that lists completed task anchors; add badge syntax for profiles/tags on each task heading.
- profiles.md: insert a "Common Lexical Snippets" callout near top; add per-profile desugaring mini-table and link to examples.
- DIAGRAMS.md: prepend an index with anchor links; add "Related examples" bullet after each diagram.
- Tables: update `tables/features-matrix.md` with a Status column and emoji bullets; update `tables/operators.md` rows with example links.
- Reference: add timestamps and an examples column; verify and add missing anchors for deep links.
- examples/README.md: write a short script plan to compute missing pairs and most useful sets; add validation footer with `just` commands.
- DASHBOARD.md: add a small PR checklist box; optional section for "Recently edited" (script planned).
- Scripts: extend `generate-status.sh` with a git commit count since a provided date; extend `validate-docs.sh` with a check for back-to-top links per major section.
- STYLE-EBNF / CONTRIBUTING: add a "Common gotchas" section and a minimal example PR diff for a `17-*` spec including anchors, examples, and desugaring.
