<!-- markdownlint-disable -->

# Hardening Report: srz-zumix--post-run-action/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **srz-zumix--post-run-action/v2.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable version tags instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if a tag is moved or a dependency is compromised.

Failing references:
- check-dist.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/upload-artifact@v4`
- ci.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/setup-python@v5`, `actions/setup-dotnet@v4`
- codeql-analysis.yml: `actions/checkout@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`
- linter.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `super-linter/super-linter/slim@v7`
- release-drafter.yml: `release-drafter/release-drafter@v6` (twice)
- update_semver.yml: `actions/checkout@v4`, `haya14busa/action-update-semver@v1`

Locations:

- `.github/workflows/check-dist.yml:29`
- `.github/workflows/check-dist.yml:33`
- `.github/workflows/check-dist.yml:60`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:52`
- `.github/workflows/ci.yml:54`
- `.github/workflows/codeql-analysis.yml:30`
- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:42`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/linter.yml:20`
- `.github/workflows/linter.yml:26`
- `.github/workflows/linter.yml:34`
- `.github/workflows/release-drafter.yml:14`
- `.github/workflows/release-drafter.yml:24`
- `.github/workflows/update_semver.yml:20`
- `.github/workflows/update_semver.yml:21`

### script-injection (severity: high)

Rule (a) violation: `${{ runner.temp }}` is interpolated directly inside `run:` shell command strings in two 'Test file' steps. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because YAML template substitution occurs before the shell ever sees the value. The offending lines are:
- `test ! -f "${{ runner.temp }}/test.txt"` (test-action job, Test file step)
- `test ! -f "${{ runner.temp }}/test.txt"` (test-action-windows job, Test file step)

These should be replaced with the environment variable form `$RUNNER_TEMP` which is already set by the runner.

Locations:

- `.github/workflows/ci.yml:84`
- `.github/workflows/ci.yml:132`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned all 18 unpinned `uses:` references across check-dist.yml, ci.yml, codeql-analysis.yml, linter.yml, release-drafter.yml, and update_semver.yml to full 40-character SHA digests with original version tags preserved as comments. Fixed the two script-injection findings in ci.yml (test-action and test-action-windows jobs) by replacing `${{ runner.temp }}` in `run:` shell commands with the safe `$RUNNER_TEMP` environment variable. The remaining `${{ runner.temp }}` expressions in the file are inside `with: post-run:` action input blocks and are not executed directly as shell commands by the workflow runner.

