# Search Tips

Use anchors and grep to navigate the docs quickly.

## Anchors
- Plan tasks: `#task-<num>-<slug>` (see [plan.md](../development/planning/plan.md))
- Profiles: [A](../specifications/profiles.md#profile-a), [B](../specifications/profiles.md#profile-b), [C](../specifications/profiles.md#profile-c)
- Tables: [Feature matrix](../reference/tables/features-matrix.md#features-matrix), [Operators](../reference/tables/operators.md#operators-table)
- Clickable index: [ANCHORS.md](../reference/anchors.md)

## Grep recipes
```bash
# Find task anchors
rg "#task-" plan.md

# Explicit anchors across docs
rg "<a id=\"" -n **/*.md

# Profile sections in ebnf
rg "^## Profile" profiles.md -n

# Examples by theme
rg "pipeline" examples -n
rg "indentation" examples -n
```

## Navigation habits
- Prefer explicit anchors for stability; add them when cross-linking sections.
- Use README (File index) and FEATURES (Navigation) to find entry points.
- DASHBOARD provides fast links and “Fast actions” for validation.
