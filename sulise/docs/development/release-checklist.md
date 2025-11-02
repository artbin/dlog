# Release Checklist (Docs)

- [ ] Update docs banners (README, FEATURES, plan.md, DIAGRAMS) with date/version
- [ ] Update tables (features matrix, operators)
- [ ] Add/verify examples (.surface ↔ .sexpr) for changes
- [ ] Regenerate anchors: `just anchors`
- [ ] Validate docs: `just validate` and `just check-iso`
- [ ] Update DASHBOARD status and highlights
- [ ] Add CHANGELOG entry under today’s date with categories
- [ ] Optional: tag release in VCS
