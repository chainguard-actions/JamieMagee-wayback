<!-- markdownlint-disable -->

# Hardening Report: JamieMagee--wayback/v1.3.51

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamieMagee--wayback/v1.3.51** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` blocks in the 'release' job directly interpolate GitHub Actions expressions into shell commands without routing through env vars.

1. 'fetch pr' step (line ~107): `git fetch origin ${{ github.ref }}` — `github.ref` is attacker-controllable via pull request and is interpolated directly into the shell command.

2. 'Merge' step (line ~112): `git merge --allow-unrelated-histories --no-ff -m 'skip: merge (${{ github.sha }}) [skip release]' ${{ github.sha }}` — `github.sha` is interpolated directly into the shell command twice.

3. 'Release' step (line ~130): `export GITHUB_REF=refs/heads/release GITHUB_SHA=${{ steps.merge.outputs.commit }}` — `steps.merge.outputs.commit` (a step output, which is a workflow-controllable context) is interpolated directly into the shell command.

All three should be moved to `env:` blocks and referenced as quoted shell variables (e.g., `"$GITHUB_REF_VAR"`) instead of using `${{ ... }}` directly inside `run:`.

Locations:

- `.github/workflows/build.yml:107`
- `.github/workflows/build.yml:112`
- `.github/workflows/build.yml:130`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and no individual job within either file defines a `permissions:` block. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g., write access to contents, pull-requests, etc.). A minimal `permissions:` block should be added at the top level or per-job level in both files.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/codeql-analysis.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed 3 script injection vulnerabilities in build.yml by moving GitHub Actions expressions (${{ github.ref }}, ${{ github.sha }}, ${{ steps.merge.outputs.commit }}) out of run: shell strings and into step-level env: blocks, then referencing them as quoted shell variables. Added permissions: {} at the top level of build.yml to deny all token permissions by default. Added minimal permissions (contents: read, security-events: write) to codeql-analysis.yml for CodeQL scanning requirements.

