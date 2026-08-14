<!-- markdownlint-disable -->

# Hardening Report: JamieMagee--wayback/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamieMagee--wayback/v2.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell blocks in release.yml, violating sub-rule (a). Affected lines:
- Line 65: `current="${{ steps.current_version.outputs.current }}"` — step output injected directly into shell
- Line 75: `case "${{ inputs.version_bump }}" in` — workflow_dispatch input injected directly into shell
- Line 101: `git commit -m "chore: release v${{ steps.version_bump.outputs.new_version }}"` — step output injected into git commit message via shell
- Line 104: `git tag "v${{ steps.version_bump.outputs.new_version }}"` — step output injected into git tag via shell
- Line 107: `git push origin "v${{ steps.version_bump.outputs.new_version }}"` — step output injected into git push via shell
- Line 122: `version="${{ steps.version_bump.outputs.new_version }}"` — step output injected directly into shell

An attacker who can influence these values (e.g. via a malicious git tag or a crafted workflow_dispatch input) could inject arbitrary shell commands.

Locations:

- `.github/workflows/release.yml:65`
- `.github/workflows/release.yml:75`
- `.github/workflows/release.yml:101`
- `.github/workflows/release.yml:104`
- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:122`

### github-env-injection (severity: high)

In the 'Calculate new version' step (around line 81), the variable `new_version` is derived from `${{ steps.current_version.outputs.current }}` and `${{ inputs.version_bump }}` — both untrusted sources injected directly into the shell — and then written to `$GITHUB_OUTPUT` via `echo "new_version=$new_version" >> $GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in the value could inject additional key=value pairs into the output file, poisoning downstream step outputs.

Locations:

- `.github/workflows/release.yml:81`

### missing-permissions (severity: medium)

build.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.). Every job should declare minimal required permissions.

Locations:

- `.github/workflows/build.yml:1`

### missing-permissions (severity: medium)

codeql-analysis.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions. The CodeQL workflow should at minimum declare `security-events: write` and `contents: read` explicitly.

Locations:

- `.github/workflows/codeql-analysis.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings:
1. script-injection in release.yml: Moved all six ${{ ... }} expressions from run: shell blocks into step env: blocks (CURRENT_VERSION, VERSION_BUMP, NEW_VERSION) and referenced them as plain environment variables.
2. github-env-injection in release.yml: Added sanitization of new_version using `printf '%s' "$new_version" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
3. missing-permissions in build.yml: Added top-level `permissions: contents: read`.
4. missing-permissions in codeql-analysis.yml: Added top-level `permissions: contents: read` and `security-events: write`.

