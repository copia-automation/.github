# copia-automation/.github

Org-wide defaults for Copia repositories.

| File | Purpose |
|---|---|
| `.github/PULL_REQUEST_TEMPLATE.md` | Default PR template for repos without their own |
| `default.json` | Shared Renovate preset (see below) |

## Renovate base preset

Repos inherit the org dependency-update policy by extending this repo:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>copia-automation/.github"],
  "baseBranchPatterns": ["main"]
}
```

`local>` resolves `default.json` from this repo's default branch, so a change here
reaches every consuming repo on its next Renovate run — no per-repo PR needed.

### What the preset sets

- **`config:best-practices`** — includes `:pinDevDependencies`, `:configMigration`,
  `helpers:pinGitHubActionDigests`, `docker:pinDigests`, and `abandonments:recommended`.
- **`security:minimumReleaseAgeNpm`** — npm releases must age before they are proposed.
- **`rebaseWhen: conflicted`** — branches are rebased only when they actually conflict.
  Open PRs still pick up newer versions as they are released.
- **`prConcurrentLimit: 3`** — at most three routine update PRs open per repo.
- **Non-major updates grouped per manager** (`{{manager}} non-major`) so three PRs cover
  most of a repo's churn; majors get individual PRs.
- **Security updates grouped** into a single `security updates` PR that is exempt from the
  concurrency limit (`vulnerabilityAlerts.prConcurrentLimit: 0`), so a fix is never queued
  behind routine updates.
- **Disabled:** `lockFileMaintenance`, rollback/replacement PRs, and npm `pin`/`pinDigest`.

### What stays in the consuming repo

- `baseBranchPatterns` — the repo's default branch.
- Registry access: `npmrc`, `npmrcMerge`, `hostRules`.
- Repo-specific presets and custom managers, e.g. `customManagers:biomeVersions`.

Anything set in the repo's own config overrides the preset.
