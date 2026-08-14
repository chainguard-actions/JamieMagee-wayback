<!-- markdownlint-disable -->

# Hardening Report: JamieMagee--wayback/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamieMagee--wayback/v2.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Calculate new version' step in release.yml directly interpolates GitHub Actions expressions inside a `run:` shell command string. The line `current="${{ steps.current_version.outputs.current }}"` injects a step output (derived from git tags) directly into the shell before quoting, and `case "${{ inputs.version_bump }}"` injects a workflow input directly. Any expression inside `${{ }}` is substituted by the YAML template engine before the shell ever sees the string, allowing shell metacharacters to escape quoting. These values should be passed via `env:` variables and then referenced as `"$VAR"` in the shell.

Locations:

- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:86`

### script-injection (severity: high)

Sub-rule (a): The 'Create release branch and tag' step in release.yml directly interpolates `${{ steps.version_bump.outputs.new_version }}` inside `run:` shell commands: in a `git commit -m` message, a `git tag` command, and a `git push` command. Step outputs are workflow-controllable data and must not be interpolated directly into shell via `${{ }}`; they should be passed through `env:` and referenced as quoted shell variables.

Locations:

- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:110`
- `.github/workflows/release.yml:114`

### script-injection (severity: high)

Sub-rule (a): The 'Update major version tag' step in release.yml directly interpolates `${{ steps.version_bump.outputs.new_version }}` inside a `run:` shell command: `version="${{ steps.version_bump.outputs.new_version }}"`. Step outputs are workflow-controllable data and must not be interpolated directly into shell via `${{ }}`; they should be passed through `env:` and referenced as quoted shell variables.

Locations:

- `.github/workflows/release.yml:124`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all three script-injection findings in hardened/action/.github/workflows/release.yml:

1. 'Calculate new version' step: Added `env:` block with CURRENT_VERSION (${{ steps.current_version.outputs.current }}) and VERSION_BUMP (${{ inputs.version_bump }}). Replaced direct ${{ }} interpolations in the run: shell with "$CURRENT_VERSION" and "$VERSION_BUMP".

2. 'Create release branch and tag' step: Added `env:` block with NEW_VERSION (${{ steps.version_bump.outputs.new_version }}). Replaced all three ${{ steps.version_bump.outputs.new_version }} interpolations in git commit -m, git tag, and git push commands with "$NEW_VERSION".

3. 'Update major version tag' step: Added `env:` block with NEW_VERSION (${{ steps.version_bump.outputs.new_version }}). Replaced the direct interpolation with "$NEW_VERSION" and also fixed an unquoted variable in the echo pipeline.

The ${{ }} references in the 'Create GitHub Release' step's `with:` block (tag_name and name) are YAML template values passed as action inputs, not shell commands, so they are not shell injection risks and were left unchanged.

