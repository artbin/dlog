# Contributing

Thanks for improving the EBNF docs and examples. This repo is documentation-first and adheres to ISO/IEC 14977 EBNF.

## Scope

- Small, focused EBNF documents (`01–16`, `17-*.md`) with rationale, desugaring, examples, and dependencies.
- Reference/overview docs (README, FEATURES, DIAGRAMS, guides) that help navigation and understanding.

## Authoring checklist (PRs)

- ISO shape
  - Terminals are double-quoted; metasymbols only; prose in `? ... ?`.
  - Unified `string`/`escape` wording; comment rules consume newline/EOF.
  - No Unicode ellipses `…`; expand ranges explicitly.
- Anchors & links
  - Add explicit anchors where referenced (e.g., `#task-...`, tables).
  - Use relative links; avoid bare URLs; add back-to-top/index links in long docs.
- Content
  - For `17-*.md`: include EBNF, desugaring (Surface→S-expr), 2–3 examples, dependencies, profiles/tags.
  - Ensure surface forms desugar to canonical S-exprs (homoiconicity preserved).
  - Add examples (.surface ↔ .sexpr) when applicable.
- Validation
  - `just validate` (links, mermaid fences, whitespace advisories)
  - `just check-iso` (ISO shape checks)
  - `just anchors` (regenerate anchors index)
- Release hygiene
  - Update tables (features/operators) if applicable.
  - Update DASHBOARD status and examples spotlight if notable.
  - Add a CHANGELOG entry (Docs: …) under today’s date.

## File naming

- EBNF variants: `NN-title-ebnf.md` (01–16 established; 17+ for high-level).
- Examples: name by theme; keep .surface and .sexpr pairs consistent (same base name).
- Guides: `*-GUIDE.md`; deep dives and plans: `*-DEEP-DIVE.md`, `*-PLAN.md`.

## Communication

- Prefer small, reviewable PRs.
- Explain rationale and cross-link tasks/anchors.
