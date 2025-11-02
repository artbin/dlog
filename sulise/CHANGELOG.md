# Changelog

All notable changes to this repository will be documented in this file.

The format is inspired by Keep a Changelog and adheres to ISO/IEC 14977 EBNF constraints across referenced grammar files.

## Policy

- Use dated sections (YYYY-MM-DD) and an "Unreleased" section for in-flight work.
- Categories: Added, Changed, Fixed, Removed, Deprecated, Security, Notes (and optionally Docs).
- Doc-only updates: mention "Docs" in the category entry or include a short "Docs:" line.
- Link to relevant files and anchors where possible (e.g., `plan.md#...`).


## [Unreleased]

### Added

- **Mistune Parser**: New context-aware markdown parser (now default)
  - Eliminates false positives from code blocks and examples
  - 2-5x faster than markdown-it parser
  - 78% reduction in false positives (18 → 4 broken links)
  - Automatically detects and skips special syntax (`@errors`, `$var`, etc.)
  - New dependency: `mistune` 3.1.4
- **Test Suite Implementation** 🧪 (Phase 4 - Week 1 COMPLETE! ✅):
  - Test infrastructure setup with pytest and coverage tools
  - Base test fixtures (tmp_repo, sample_files, cli_runner, etc.)
  - Helper functions for test assertions and data generation
  - **Week 1, Day 2**: Manifest parser tests (14 tests, 86% coverage of manifest.py)
  - **Week 1, Day 3**: Operations tests (17 tests, 53% coverage of operations.py)
  - **Week 1, Day 4**: Path calculator tests (19 tests, 100% coverage of path_calculator.py) 🎯
  - **Week 1, Day 5**: Snapshot tests (14 tests, 97% coverage of core.py)
  - pytest configuration with markers and coverage targets
  - Test directory structure (unit/integration/e2e/performance/regression)
  - **Week 1 Summary**: 64 tests (183% of target!), 27% overall coverage, <3s execution
- **Documentation Migrations System** ✅ (Phase 1-3 Complete):
  - Complete system design for safe documentation restructuring
  - Auto-extraction feature to generate migrations from git/docs
  - 13 new justfile recipes for migration management
  - **Phase 1 - Core Engine**: File moves, snapshots, rollback
  - **Phase 2 - Link Updates**: Automatic link analysis and updates
    - Link analyzer finds all affected files
    - Path calculator computes relative paths
    - Link updater applies changes to markdown files
    - Automatic link updates when files move
    - Preserves anchor fragments (#section)
    - Handles inline and reference-style links
  - **Phase 3 - Validation & Safety**: Pre/post migration validation
    - Pre-migration validation checks (uncommitted changes, file existence)
    - Post-migration validation (broken links detection)
    - Integration with existing crawler
    - Automatic rollback on validation failure
    - Custom validation rules per migration
    - CLI commands: validate-pre, validate-post
  - **Phase 4 - Auto-Extraction**: Automatic migration generation ✨ NEW
    - Git detector extracts file moves from git history
    - Doc detector parses markdown tables for file moves
    - YAML generator creates complete migration manifests
    - Extract from working tree (staged changes)
    - Extract from commits or commit ranges
    - Extract from markdown plan documents
    - Preview mode to inspect detected moves
    - CLI command: extract with multiple source options
  - **Link Analyzer Bug Fixes** (3 critical bugs fixed):
    - Fixed reverse mapping to recognize moved files at new locations
    - Fixed link resolution to use old location for moved files
    - Fixed cascade moves (when both referrer and target moved)
    - Result: 96% accuracy (447 links updated, 381→8 broken links)
  - **Non-Interactive Rollback**: Added `--yes` flag to rollback command
    - Enables automation and CI/CD workflows
    - Matches existing `--yes` flag behavior from apply command
    - Usage: `migrate rollback <snapshot> --yes`
    - Default behavior unchanged (still requires confirmation)
  - Git history preservation with `git mv`
  - Snapshot-based rollback
  - YAML-based migration manifests
  - Pre/post migration validation
  
- **Documentation**:
  - `DOC-MIGRATIONS-DESIGN.md` (1,064 lines) - Complete system architecture
  - `DOC-MIGRATIONS-QUICKSTART.md` (540+ lines) - 15-minute tutorial
  - `DOC-MIGRATIONS-AUTO-EXTRACT.md` (980 lines) - Auto-extraction design
  - `DOC-MIGRATIONS-IMPLEMENTATION-PLAN.md` (680+ lines) - Detailed implementation plan
  - `MIGRATION-PLAN-V2.md` (685 lines) - Updated comprehensive restructure plan
  - `DOC-SYSTEM-SUMMARY.md` (570 lines) - Complete documentation system overview
  
- **Migration Commands** (via justfile):
  - `migrate-list` - List pending/applied migrations
  - `migrate-plan <name>` - Preview migration (dry run)
  - `migrate-apply <name>` - Execute migration (interactive)
  - `migrate-rollback <name>` - Rollback migration (interactive)
  - `migrate-validate <name>` - Validate YAML syntax
  - `migrate-status` - Show system status
  - `migrate-restructure` - Interactive comprehensive restructure
  - `migrate-extract-working <name>` - Extract from staged changes ✅
  - `migrate-extract-commit <name>` - Extract from git commit ✅
  - `migrate-extract-doc <doc> <name>` - Extract from markdown tables ✅
  - `migrate-extract-preview` - Preview extraction ✅
  - `migrate-extract-restructure` - Extract comprehensive restructure ✅
  - For automation: Add `--yes` flag to apply/rollback for non-interactive mode
  
- **Example Migrations**:
  - `migrations/pending/example_file_move.yml` - Simple file move example
  - `migrations/pending/example_anchor_normalization.yml` - Anchor update example

### Notes
- Documentation migrations system enables safe large-scale restructuring
- Auto-extraction feature makes migration creation 100x faster ✅
- System designed for comprehensive docs restructure (54→6 root files)
- **Phase 1 Complete**: Core engine (~1,680 lines)
- **Phase 2 Complete**: Link updates (~900 lines)
- **Phase 3 Complete**: Validation & safety (~500 lines)
- **Phase 4 Complete**: Auto-extraction (~792 lines) ✨
- **Phase 4 In Progress**: Test suite implementation (Week 1, Day 2 complete)
  - 8 tests implemented and passing
  - Test infrastructure operational
  - 12% overall coverage, 70% manifest.py coverage
  - Target: 110 tests, 85%+ coverage by Week 4
- Total implementation: ~3,872 lines of production code + ~530 lines test code
- **Link updater validated**: 96% accuracy on 49-file migration
- **System fully validated and operational**: Extract → Review → Apply → Rollback

### Usage Notes
- **File moves are staged, not committed**: After migration, file moves are staged with `git mv`
  - Check with `git status` to see moved files (shows as `R old -> new`)
  - Commit the changes: `git add -A && git commit -m "docs: restructure"`
  - Or rollback: `uv run python scripts/migrate_docs.py rollback <snapshot>`
  - This is intentional design to allow review before committing

## [2025-10-16]

### Added
- **Python Project Infrastructure**:
  - Initialized Python project using `uv` package manager (v0.8.13)
  - Created `pyproject.toml` with project metadata and console script entry points
  - Created `scripts/` package with Python implementations of all validation scripts
  - Added `.gitignore` for Python artifacts and virtual environments
  - Python requires: >=3.12, zero external dependencies (pure stdlib)
- **New justfile recipes**:
  - `install`: Install Python package with uv
  - `update`: Update dependencies
  - `generate`: Run all generators (anchors + status update)
  - `ci`: Full CI pipeline (install, generate, validate)
  - `clean`: Remove generated files and Python cache
  - `fmt`: Placeholder for future code formatting
  - `run-*`: Direct script execution for development/debugging
  - Legacy `*-bash` recipes for backward compatibility

### Changed
- **Scripts Migration**: All validation scripts implemented in Python with improved functionality:
  - `scripts/validate_docs.py`: Cleaner validation logic, better error messages
  - `scripts/check_iso_shape.py`: Enhanced ISO EBNF shape checks with 7 validations:
    - Unicode ellipsis detection (fails build)
    - Single-quoted terminals in EBNF detection (fails build)
    - Bare lexer terms (EOF/NEWLINE/INDENT/DEDENT) outside `? ... ?` (advisory)
    - Canonical string/escape wording validation (advisory)
    - Non-`#` line comment detection in EBNF blocks (fails build)
    - Orphan block comment `#|` detection (advisory)
    - Tab detection in markdown (advisory)
  - `scripts/generate_status.py`: Streamlined status generation with better git integration
  - `scripts/generate_anchors.py`: More efficient anchor extraction, cleaner output (421 lines vs 1296)
- **Python Scripts Documentation**: Added comprehensive documentation to all scripts (+241 lines, +46%):
  - Module-level docstrings (20-30 lines each) with purpose, usage examples, exit codes, and cross-references
  - Function docstrings for all 10 functions with type hints, parameters, and return values
  - Inline comments explaining validation logic, severity levels, and rationale
  - Doctest example in `slugify()` function
  - Documentation viewable via `pydoc` and IDE tooltips
  - References to STYLE-EBNF.md, CONTRIBUTING.md, plan-docs.md
- **Anchor Generation Redesign**: Simplified to fully automated single-file approach:
  - `ANCHORS.md` (212 lines) now auto-generated, replacing manual curation
  - Explicit anchors grouped by file with descriptive labels: "(tasks)", "(profiles)", "(features)", etc.
  - Priority file system: Only includes ~12 key files (README, FEATURES, FAQ, etc.) in headings
  - Smart sorting: tables → plan → ebnf → reference → others for better discoverability
  - Added to `.gitignore` to avoid git churn from regeneration
  - Clean output with proper indentation and visual hierarchy
  - Removed `ANCHORS-FULL.md` in favor of single unified approach
- **justfile**: Completely redesigned with better organization and documentation
  - Uses `uv run` for consistent Python environment
  - Added descriptive comments for all recipes
  - Improved success messages with checkmarks
  - Simplified `anchors` recipe generates single `ANCHORS.md` file
  - Updated `clean` recipe to remove auto-generated `ANCHORS.md`

### Fixed
- More accurate line number reporting in validation errors
- Better handling of edge cases in markdown parsing
- Improved UTF-8 encoding handling across all platforms
- More robust error messages with file paths and line numbers

### Notes
- Python scripts are faster and more maintainable than bash equivalents
- Zero external dependencies required - uses only Python stdlib
- `just all` runs both validate and check-iso validations
- `just ci` provides complete CI-ready validation pipeline
- Current known findings: Unicode ellipses in `plan.md`, `FAQ.md`, `CHANGELOG.md`; single-quoted terminals in several EBNF files (legitimate detections)

## [2025-10-15]

### Added
- New documentation files:
  - `plan.md` (comprehensive task plan with dependencies, examples, grouped index)
  - `ROADMAP.md` (milestones, implementation order, status)
  - `DIAGRAMS.md` (Mermaid diagrams for evaluation and desugaring flows)
  - `BACKLOG.md` (prioritized next/near/medium/long-term items)
  - `CHANGELOG.md` (this file)
  - `FEATURES.md` (feature summary and navigation pointers)
  - `plan-docs.md` (documentation improvement plan and deliverables)
  - `sexpr.md` (S-expressions core explainer and Surface vs Core terminology)
  - `surface.md` (Surface syntax explainer and contract)
  - `DASHBOARD.md` (central navigation, status, commands)
  - `PROFILES.md` (landing page for Profiles A/B/C with examples)
  - `CONTRIBUTING.md` (PR checklist and authoring rules)
  - `STYLE-EBNF.md` (ISO style guide and canonical lexemes)
  - `docs-template-17.md` (template for new feature specs)
  - `SEARCH-TIPS.md` (anchors/grep/navigation guidance)
  - `LEXER-GUIDE.md` (indentation tokens, comments, numerics)
  - `PARSER-GUIDE.md` (operator tiers, application, pipeline, desugaring)
  - `TESTING.md` (fixtures and validations)
  - `RELEASE-CHECKLIST.md` (docs release checklist)
  - `OPERATORS-DEEP-DIVE.md` (precedence/associativity details)
  - `TYPES-PLAN.md` (as/of roadmap and examples)
  - `ERRORS-PLAN.md` (Zig-style errors and integrations)
  - `EXAMPLES-GUIDE.md` (pairing and verification)
  - `SITE.md` (optional site publishing guidance)
  - `reference/grammar-index.md` (nonterminal index)
  - `reference/keywords-and-operators.md` (reserved words and operators)
  - `tables/features-matrix.md` (feature availability across profiles)
  - `tables/operators.md` (operator precedence/associativity)
  - `LICENSE` (MIT-0, No Attribution)
  - `AUTHORS.md` (project authors)
- High-level EBNF variant specifications (as separate docs already in repo prior to this changelog):
  - 05–09 (keywords/escaped symbols, numbers radix/exactness, nested block comments, characters/bytevectors, maps/sets)
  - 10–13 (application tighter than infix, prefix/postfix, minimal precedence, pipeline)
  - 14–16 (indentation variants: optional colons, single-line suites, trailing-operator join)
- Plan: extensive extension tasks (16–114+) with explanations, examples, dependencies, grouped index, mini‑TOCs, anchors, tag index, and dependency graphs.
 - Examples directory populated with surface/S-expr pairs (pipelines, precedence, indentation, literals, comments, nested collections, etc.)
 - Validation scripts and tooling:
  - `scripts/check-iso-shape.sh`, `scripts/validate-docs.sh`
  - `scripts/generate-anchors.sh`, `scripts/generate-status.sh`
  - `justfile` recipes: `validate`, `check-iso`, `anchors`, `status`, `status-update`, `all`

### Changed
- README updates:
  - Added unified comment marker note (`#`) and line‑comment consumption
  - Added quick links bar and a Table of Contents
  - Linked to `plan.md`, `ROADMAP.md`, `DIAGRAMS.md`, `BACKLOG.md`, and `FEATURES.md`
  - Added links and pointers to `sexpr.md` (Surface vs Core)
  - Added quick link and docs reference to `surface.md` (Surface syntax)
  - Added Quick links to Shortlist and Examples; added Examples section and Shortlist link in Project docs
  - Added links to new references and guides (PROFILES, LEXER-GUIDE, PARSER-GUIDE, TESTING, RELEASE-CHECKLIST, OPERATORS-DEEP-DIVE, TYPES-PLAN, ERRORS-PLAN, CONTRIBUTING, STYLE-EBNF, docs-template-17, SEARCH-TIPS)
  - Added "Reference implementation (Lipsi)" section
- Plan diagrams moved from `plan.md` to `DIAGRAMS.md` with a link back.
- Unified line comment markers across grammars to `#`; retained `#| ... |#` where applicable.
 - DASHBOARD significantly enhanced: TOC, status (commit and example counts), grid quick links, examples spotlight, quality gates, release readiness, search tips, references bundle, fast actions; added STATUS markers and `just status` / `just status-update` commands.
 - Added docs version banners to major docs (README, FEATURES, plan.md, DIAGRAMS).

### Fixed
- ISO EBNF compliance improvements:
  - Expanded `letter`/`digit` ranges to explicit terminals (no ellipses)
  - Normalized `string`/`escape` and whitespace/comment rules
  - Replaced Unicode ellipses `…` with ASCII `...`
  - Ensured “any char except …” wording is consistently wrapped in `? ... ?`

### Notes
- Homoiconicity clarified in README: all surface profiles desugar to canonical S‑expressions before macro‑expansion and evaluation.
- See `plan.md` for per‑task dependencies and detailed examples.
