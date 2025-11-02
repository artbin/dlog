# Site (optional)

Optional static site for easier browsing (mkdocs or docusaurus). The repo remains the source of truth.

## Principles
- Do not diverge content between site and repo.
- Prefer relative links and stable anchors; validate link hygiene.

## mkdocs (example)
- Install mkdocs and theme locally.
- Map docs directory; include navigation for Profiles, Plan, Diagrams, Examples.
- Validate locally; deploy as needed.

## docusaurus (example)
- Initialize site; import markdown docs.
- Configure sidebar and route mapping; ensure anchors remain stable.

## CI (optional)
- Add a workflow to build and preview on PRs.
- Keep publishing disabled until content stabilizes.
