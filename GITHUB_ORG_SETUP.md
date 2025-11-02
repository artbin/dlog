# GitHub Organization Setup Checklist

Complete guide for setting up the Pyralog GitHub organization.

## ✅ Phase 1: Organization Profile

### Profile Settings
Go to: `https://github.com/organizations/pyralog/settings/profile`

- [ ] **Profile Picture**: Upload 🔺 pyramid logo (512x512px PNG)
- [ ] **Display Name**: `Pyralog`
- [ ] **Description**: `🔺 The Pyramid of Distributed Logs - Built to Last Millennia`
- [ ] **Website**: `https://pyralog.io`
- [ ] **Email**: `hello@pyralog.io`
- [ ] **Twitter**: `@pyralog`

### Organization README
1. [ ] Create a new repository: `https://github.com/organizations/pyralog/repositories/new`
   - Repository name: `.github`
   - Description: "Organization profile and community health files"
   - Public
2. [ ] Create `profile/README.md` (use template from `.github-org-profile.md`)
3. [ ] This will display on your organization's main page

---

## ✅ Phase 2: Repository Settings

### Main Repository (pyralog/pyralog)
Go to: `https://github.com/pyralog/pyralog/settings`

#### General Settings
- [ ] **Description**: `🔺 Platform for secure, parallel, distributed, and decentralized computing`
- [ ] **Website**: `https://pyralog.io`
- [ ] **Topics** (add these tags):
  ```
  distributed-systems
  database
  rust
  actor-model
  wireguard
  quantum-resistant
  multi-model
  time-series
  append-only-log
  consensus
  replication
  cryptography
  zero-trust
  lisp
  batuta
  ```
- [ ] **Features**:
  - ✅ Wikis (for community documentation)
  - ✅ Issues
  - ✅ Discussions
  - ❌ Projects (unless you want to use them)
  - ❌ Sponsorships (enable if you want GitHub Sponsors)

#### Branch Protection
Go to: `https://github.com/pyralog/pyralog/settings/branches`

Create rule for `main`:
- [ ] **Branch name pattern**: `main`
- [ ] **Require a pull request before merging**
  - ✅ Require approvals: 1
  - ❌ Dismiss stale PR approvals when new commits are pushed (optional)
  - ❌ Require review from Code Owners (if you create CODEOWNERS file)
- [ ] **Require status checks to pass before merging**
  - ✅ Require branches to be up to date before merging
  - Add status checks: (after setting up CI/CD)
    - `test`
    - `clippy`
    - `fmt`
- [ ] **Require conversation resolution before merging** (optional)
- [ ] **Require signed commits** (recommended for security)
- [ ] **Include administrators** (enforce rules for everyone)
- [ ] **Restrict who can push to matching branches** (optional)
- [ ] **Allow force pushes**: ❌ Nobody
- [ ] **Allow deletions**: ❌

#### Security
Go to: `https://github.com/pyralog/pyralog/settings/security_analysis`

- [ ] **Private vulnerability reporting**: ✅ Enable
- [ ] **Dependabot alerts**: ✅ Enable
- [ ] **Dependabot security updates**: ✅ Enable
- [ ] **Dependabot version updates**: Configure `.github/dependabot.yml`
- [ ] **Code scanning**: Set up CodeQL (for Rust)
- [ ] **Secret scanning**: ✅ Enable (automatically enabled for public repos)

#### Collaborators & Teams
Go to: `https://github.com/pyralog/pyralog/settings/access`

Create teams:
- [ ] **Core Team** (Admin access)
  - Add yourself
- [ ] **Maintainers** (Maintain access)
  - For trusted contributors
- [ ] **Contributors** (Write access)
  - For regular contributors
- [ ] **Triagers** (Triage access)
  - For issue/PR management

---

## ✅ Phase 3: Community Health Files

Create these files in the `.github` repository (organization-wide) or in `pyralog/pyralog`:

### Issue Templates
Go to: `https://github.com/pyralog/pyralog/issues/templates/edit`

Or create manually in `.github/ISSUE_TEMPLATE/`:

1. [ ] **Bug Report** (`.github/ISSUE_TEMPLATE/bug_report.yml`)
2. [ ] **Feature Request** (`.github/ISSUE_TEMPLATE/feature_request.yml`)
3. [ ] **Documentation** (`.github/ISSUE_TEMPLATE/documentation.yml`)
4. [ ] **Performance** (`.github/ISSUE_TEMPLATE/performance.yml`)

### Pull Request Template
- [ ] Create `.github/PULL_REQUEST_TEMPLATE.md`

### Community Guidelines
- [ ] **CODE_OF_CONDUCT.md** (adopt Contributor Covenant)
- [ ] **SECURITY.md** (security policy and vulnerability reporting)
- [ ] **SUPPORT.md** (how to get help)
- [ ] **GOVERNANCE.md** (optional: project governance model)

### CODEOWNERS
- [ ] Create `.github/CODEOWNERS` to auto-assign reviewers

Example:
```
# Default owners for everything
* @artbin

# Rust core
/pyralog-*/src/ @artbin
/src/ @artbin

# Documentation
*.md @artbin
/docs/ @artbin

# CI/CD
/.github/ @artbin
```

---

## ✅ Phase 4: GitHub Actions & CI/CD

Create `.github/workflows/` directory with these workflows:

### 1. CI Pipeline (`.github/workflows/ci.yml`)
- [ ] **Test**: Run `cargo test --all-features`
- [ ] **Clippy**: Run `cargo clippy -- -D warnings`
- [ ] **Format**: Run `cargo fmt -- --check`
- [ ] **Build**: Run `cargo build --release`
- [ ] **Benchmarks**: Run performance tests
- [ ] **Coverage**: Upload to codecov.io

### 2. Security Audit (`.github/workflows/security.yml`)
- [ ] **Cargo Audit**: Check for security vulnerabilities
- [ ] **Cargo Deny**: Check licenses and advisories

### 3. Release (`.github/workflows/release.yml`)
- [ ] **Automatic releases** on version tags
- [ ] **Build binaries** for Linux, macOS, Windows
- [ ] **Publish to crates.io** (with API token)
- [ ] **Generate changelog** from commits

### 4. Documentation (`.github/workflows/docs.yml`)
- [ ] **Build docs**: `cargo doc --no-deps`
- [ ] **Deploy to GitHub Pages**: `gh-pages` branch

---

## ✅ Phase 5: Labels & Milestones

### Issue Labels
Go to: `https://github.com/pyralog/pyralog/labels`

Organize by category:

**Type:**
- [ ] `bug` (🐛, #d73a4a)
- [ ] `feature` (✨, #a2eeef)
- [ ] `documentation` (📚, #0075ca)
- [ ] `performance` (⚡, #fbca04)
- [ ] `security` (🔒, #ee0701)

**Priority:**
- [ ] `P0: Critical` (#b60205)
- [ ] `P1: High` (#d93f0b)
- [ ] `P2: Medium` (#fbca04)
- [ ] `P3: Low` (#0e8a16)

**Status:**
- [ ] `good first issue` (🌱, #7057ff)
- [ ] `help wanted` (🙏, #008672)
- [ ] `needs-triage` (#ededed)
- [ ] `blocked` (#000000)
- [ ] `wontfix` (#ffffff)

**Component:**
- [ ] `obelisk-sequencer` (🗿)
- [ ] `pharaoh-network` (☀️)
- [ ] `scarab-ids` (🪲)
- [ ] `batuta` (🎼)
- [ ] `actor-model`
- [ ] `wireguard`
- [ ] `cryptography`
- [ ] `storage`
- [ ] `consensus`

### Milestones
- [ ] **v0.1.0**: Initial alpha release
- [ ] **v0.2.0**: Actor model implementation
- [ ] **v0.3.0**: WireGuard integration
- [ ] **v0.4.0**: Batuta language
- [ ] **v1.0.0**: Production-ready release

---

## ✅ Phase 6: GitHub Pages (Optional)

For hosting documentation at `pyralog.github.io/pyralog`:

1. [ ] Go to: `https://github.com/pyralog/pyralog/settings/pages`
2. [ ] **Source**: Deploy from a branch
3. [ ] **Branch**: `gh-pages` / `/(root)`
4. [ ] **Custom domain**: `docs.pyralog.io` (after DNS setup)
5. [ ] **Enforce HTTPS**: ✅

### Set up `gh-pages` branch:
```bash
cargo doc --no-deps
echo '<meta http-equiv="refresh" content="0; url=pyralog">' > target/doc/index.html
ghp-import -n -p -f target/doc
```

Or use GitHub Actions to auto-deploy.

---

## ✅ Phase 7: Integrations & Services

### Third-Party Services
- [ ] **Codecov**: Code coverage tracking
  - Add `CODECOV_TOKEN` to repository secrets
  - Badge: `![Coverage](https://codecov.io/gh/pyralog/pyralog/branch/main/graph/badge.svg)`

- [ ] **Docs.rs**: Automatic Rust documentation
  - Automatically indexes crates published to crates.io

- [ ] **Crates.io**: Rust package registry
  - Set up publishing with API token
  - Add to GitHub secrets: `CARGO_REGISTRY_TOKEN`

- [ ] **Discord**: Server notifications
  - Create webhooks for:
    - New releases
    - Pull requests
    - Issues
    - GitHub Stars

### Badges for README
Update `README.md` with:
```markdown
[![CI](https://github.com/pyralog/pyralog/workflows/CI/badge.svg)](https://github.com/pyralog/pyralog/actions)
[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-blue.svg)](https://opensource.org/licenses/MIT-0)
[![Crates.io](https://img.shields.io/crates/v/pyralog.svg)](https://crates.io/crates/pyralog)
[![Docs.rs](https://docs.rs/pyralog/badge.svg)](https://docs.rs/pyralog)
[![Discord](https://img.shields.io/discord/YOUR_SERVER_ID?label=discord)](https://discord.gg/pyralog)
```

---

## ✅ Phase 8: Organization Settings

### Member Privileges
Go to: `https://github.com/organizations/pyralog/settings/member_privileges`

- [ ] **Base permissions**: Read
- [ ] **Repository creation**: Disable (or limit to admin)
- [ ] **Repository forking**: Enable
- [ ] **Pages creation**: Disable (or limit to admin)
- [ ] **Team creation**: Limit to admin

### Security
Go to: `https://github.com/organizations/pyralog/settings/security`

- [ ] **Two-factor authentication**: Require for all members
- [ ] **SSO**: Configure if using enterprise (optional)
- [ ] **IP allow list**: Add if needed (optional)

### Code Security & Analysis
- [ ] **Dependency graph**: ✅ Enable
- [ ] **Dependabot alerts**: ✅ Enable for all repos
- [ ] **Dependabot security updates**: ✅ Enable
- [ ] **Secret scanning**: ✅ Enable

### Third-party access
- [ ] Review OAuth applications
- [ ] Review GitHub Apps

---

## ✅ Phase 9: Repository Creation Scripts

### Create Additional Repositories

1. **pyralog/batuta** (Batuta language implementation)
2. **pyralog/examples** (Real-world examples)
3. **pyralog/benchmarks** (Performance benchmarks)
4. **pyralog/docs** (Standalone documentation site)
5. **pyralog/.github** (Organization-wide community files)

### Quick Script (Manual on GitHub):
For each repository:
- Description matching its purpose
- Topics/tags relevant to content
- Same branch protection rules as main repo
- Enable Issues, Discussions, Wikis as needed

---

## 📝 Quick Commands

### Update local repository after org move:
```bash
# Already done!
git remote set-url origin https://github.com/pyralog/pyralog.git
```

### Create organization profile:
```bash
# Clone .github repo
git clone https://github.com/pyralog/.github.git
cd .github

# Create profile directory
mkdir -p profile
cp /path/to/.github-org-profile.md profile/README.md

# Commit and push
git add profile/README.md
git commit -m "Add organization profile README"
git push origin main
```

---

## 🎯 Priority Order

**Do First (Critical):**
1. ✅ Organization profile (picture, description, links)
2. ✅ Repository settings (topics, description, features)
3. ✅ Branch protection rules
4. ✅ Security settings (Dependabot, vulnerability reporting)

**Do Soon (Important):**
5. Issue/PR templates
6. Community health files (CODE_OF_CONDUCT, SECURITY, CONTRIBUTING)
7. CI/CD workflows
8. Labels and milestones

**Do Later (Nice to Have):**
9. GitHub Pages
10. Additional repositories
11. Third-party integrations
12. Team structure

---

## 🚀 Getting Started

Start here:
1. Go to https://github.com/pyralog
2. Click ⚙️ Settings
3. Work through each section above
4. Check off items as you complete them

---

**Questions?** Open an issue or ask on Discord!

**License**: CC0-1.0 (Public Domain)

