<!-- markdownlint-disable -->

# Hardening Report: JamieMagee--wayback/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamieMagee--wayback/v2.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell commands in release.yml, violating sub-rule (a). (1) The 'Calculate new version' step interpolates `${{ steps.current_version.outputs.current }}` and `${{ inputs.version_bump }}` directly into the shell script — a user-controlled workflow_dispatch input and a prior step output are both injected verbatim before the shell parses them. (2) The 'Create release branch and tag' step interpolates `${{ steps.version_bump.outputs.new_version }}` into git commit message, git tag, and git push commands. (3) The 'Update major version tag' step interpolates `${{ steps.version_bump.outputs.new_version }}` into the shell script. Any of these values containing shell metacharacters would result in command injection.

Locations:

- `.github/workflows/release.yml:62`
- `.github/workflows/release.yml:73`
- `.github/workflows/release.yml:93`
- `.github/workflows/release.yml:96`
- `.github/workflows/release.yml:99`
- `.github/workflows/release.yml:109`

### github-env-injection (severity: high)

The 'Calculate new version' step writes `echo "new_version=$new_version" >> $GITHUB_OUTPUT` where `$new_version` is computed from `${{ steps.current_version.outputs.current }}` (a steps.*.outputs.* value) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Similarly, the 'Get current version from Git tags' step writes `echo "current=$current" >> $GITHUB_OUTPUT` where `$current` is derived from git tag output. Both writes to $GITHUB_OUTPUT lack newline sanitization, allowing a malicious value containing newlines to inject additional key=value pairs into the output context.

Locations:

- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:80`

### missing-permissions (severity: medium)

build.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions. A minimal permissions block (e.g. `permissions: read-all` or specific scopes) should be added.

Locations:

- `.github/workflows/build.yml:1`

### missing-permissions (severity: medium)

codeql-analysis.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default token permissions. CodeQL analysis workflows typically need `security-events: write` and `contents: read` — these should be declared explicitly.

Locations:

- `.github/workflows/codeql-analysis.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 findings across 3 workflow files:

1. release.yml - script-injection: Moved all ${{ }} expressions from run: shell commands into env: blocks. Specifically: (a) 'Calculate new version' step now uses CURRENT_VERSION and VERSION_BUMP env vars instead of direct interpolation; (b) 'Create release branch and tag' step uses NEW_VERSION env var for git commit message, tag, and push; (c) 'Update major version tag' step uses NEW_VERSION env var.

2. release.yml - github-env-injection: Added newline sanitization using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT in both the 'Get current version from Git tags' step (safe_current) and the 'Calculate new version' step (safe_new_version). Also properly quoted $GITHUB_OUTPUT.

3. build.yml - missing-permissions: Added top-level `permissions: contents: read` block.

4. codeql-analysis.yml - missing-permissions: Added top-level `permissions:` block with `contents: read` and `security-events: write` (required for CodeQL to upload analysis results).

