# GitHub repository setup

One-time configuration for a repo created from this template. Everything here is a GitHub setting (not versioned in the repo), so apply it after creating the repo. Commands use the [GitHub CLI](https://cli.github.com/) (`gh auth login` first); replace `OWNER/REPO` with your repo.

```bash
export REPO=OWNER/REPO
```

---

## 1. About box (metadata)

```bash
gh repo edit $REPO \
  --description "Production-ready monorepo template: Hono + Prisma + Postgres API, Next.js + shadcn/ui + Clerk web, pnpm, Biome, Docker, GitHub Actions." \
  --homepage "https://github.com/OWNER/REPO#readme" \
  --add-topic nextjs --add-topic hono --add-topic prisma --add-topic postgres \
  --add-topic clerk --add-topic monorepo --add-topic pnpm --add-topic biome \
  --add-topic typescript --add-topic shadcn-ui --add-topic template --add-topic starter \
  --template
```

`--template` turns on **Use this template**. Upload a social preview image (1280×640) in **Settings → General → Social preview**.

## 2. General settings

```bash
gh repo edit $REPO \
  --enable-discussions \
  --enable-issues \
  --enable-wiki=false \
  --enable-projects=false \
  --enable-merge-commit=false \
  --enable-squash-merge \
  --enable-rebase-merge \
  --delete-branch-on-merge
```

| Setting | Value | Why |
| --- | --- | --- |
| Merge commits | **OFF** | Keeps history Conventional-Commits-pure (see `CONTRIBUTING.md`). |
| Squash merging | **ON** | Use for `feature → dev` PRs. |
| Rebase merging | **ON** | Use for promotion PRs (`dev → test`, `test → main`) so each `feat`/`fix` commit reaches semantic-release. |
| Default squash message | **Pull request title** | Settings → General → Pull Requests. The title is validated. |
| Auto-delete head branches | **ON** | Cleanup after merge. |
| Always suggest updating PR branches | **ON** | Settings → General → Pull Requests. |
| Default branch | `main` | Public landing branch. |

## 3. Branches and protection

Flow: `feature/* → dev → test → main`, then `main → dev` sync. See `CONTRIBUTING.md` § 2.

Create the long-lived branches if missing (`git push origin main:dev main:test`), then protect all three.

### Required status checks

These are the job names from the workflows. Skipped jobs (e.g. `pr-check` on docs-only PRs) count as passing.

| Check | Workflow |
| --- | --- |
| `pr-check` | `ci.yml` |
| `branch-name` | `check-branch.yml` |
| `semantic-pull-request` | `check-pull-request.yml` |
| `commit-messages` | `check-pull-request.yml` |

### Per-branch rules

| Rule | `dev` | `test` | `main` |
| --- | --- | --- | --- |
| Require a pull request | ✅ | ✅ | ✅ |
| Required approvals | 0 solo / 1 team | 0 solo / 1 team | 0 solo / 1 team |
| Require status checks (above), branch up to date | ✅ | ✅ | ✅ |
| Require conversation resolution | ✅ | ✅ | ✅ |
| Require linear history | ✅ | ✅ | ✅ |
| Block force pushes | ✅ | ✅ | ✅ |
| Block deletions | ✅ | ✅ | ✅ |
| Dismiss stale approvals on new commits | ✅ | ✅ | ✅ |
| Require review from Code Owners | team only | team only | team only |
| Allow admins to bypass | ✅ (hotfix escape hatch) | ✅ | ✅ |
| Allowed PR source | `feature/*`, `fix/*`, `chore/*`, `dependabot/*`, `main` (sync) | `dev` only | `test` only |
| Merge method to use | squash | **rebase** | **rebase** |

Notes:

- **Solo maintainer:** GitHub does not let you approve your own PR, so keep approvals at **0** and leave *Require review from Code Owners* **off** (CODEOWNERS is a single user in this template). Admin bypass covers emergencies.
- **Team:** raise approvals to 1, turn on Code Owners, and replace `@morvenlabs` in `.github/CODEOWNERS` with your team handles.
- "Allowed PR source" is a convention, not a GitHub setting; `check-branch.yml` already enforces branch names for features.
- Enable **Do not allow bypassing the above settings** only if you have a second admin.

### Apply with the CLI

Run once per branch (`dev`, `test`, `main`). Set `APPROVALS=0` for a solo maintainer.

```bash
APPROVALS=0
for BRANCH in dev test main; do
  gh api -X PUT "repos/$REPO/branches/$BRANCH/protection" --input - <<JSON
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["pr-check", "branch-name", "semantic-pull-request", "commit-messages"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": $APPROVALS,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": false
  },
  "restrictions": null,
  "required_linear_history": true,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "required_conversation_resolution": true
}
JSON
done
```

## 4. Actions

**Settings → Actions → General**

- Actions permissions: *Allow all actions and reusable workflows* (the workflows use third-party actions such as `dorny/paths-filter` and `amannn/action-semantic-pull-request`).
- Fork pull request workflows: *Require approval for first-time contributors*.
- Workflow permissions: *Read repository contents* (workflows declare the extra permissions they need).

**Settings → Secrets and variables → Actions**

- CI needs **no** secrets or variables.
- Releases are off until you set the variable `ENABLE_RELEASES=true`; then add the secrets listed in [`.github/SECRETS.md`](../.github/SECRETS.md).

## 5. Security

**Settings → Code security**

| Feature | Value |
| --- | --- |
| Private vulnerability reporting | **ON** (`SECURITY.md` and the issue chooser link to it) |
| Dependency graph | ON |
| Dependabot alerts | ON |
| Dependabot security updates | ON |
| Dependabot version updates | driven by `.github/dependabot.yml` (targets `dev`) |
| Secret scanning + push protection | ON |
| Code scanning (CodeQL default setup) | ON (optional, free on public repos) |

```bash
gh api -X PUT repos/$REPO/private-vulnerability-reporting
gh api -X PUT repos/$REPO/vulnerability-alerts
gh api -X PUT repos/$REPO/automated-security-fixes
```

## 6. Labels

Dependabot and the templates reference these labels:

```bash
gh label create dependencies -R $REPO -c 0366d6 -d "Dependency updates" --force
gh label create api          -R $REPO -c 1d76db -d "apps/api" --force
gh label create web          -R $REPO -c 5319e7 -d "apps/web" --force
gh label create ci           -R $REPO -c fbca04 -d "CI/CD and workflows" --force
gh label create bug          -R $REPO -c d73a4a --force
gh label create enhancement  -R $REPO -c a2eeef --force
gh label create "good first issue" -R $REPO -c 7057ff --force
gh label create "help wanted"      -R $REPO -c 008672 --force
```

## 7. Discussions

Enable Discussions (Settings → General → Features) and create the categories **Q&A**, **Ideas** and **Show and tell**. The issue chooser sends usage questions there.

## 8. Final check

- [ ] `README.md` badges resolve (the CI badge shows "no status" until the first workflow run).
- [ ] **Insights → Community Standards** shows every item green (description, README, code of conduct, contributing, license, security policy, issue templates, PR template).
- [ ] A test PR `feat/test-ci → dev` passes all four checks with no secrets set.
- [ ] *Use this template* creates a repo where `pnpm install && pnpm dev` works following only the README.
