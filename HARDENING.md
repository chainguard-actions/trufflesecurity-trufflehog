<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.95.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **trufflesecurity--trufflehog/v3.95.9** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The run: block in action.yml directly interpolates GitHub Actions expressions inside shell commands. Specifically: `${{ github.event_name }}` (lines ~63, 75, 78), `${{ github.event.after }}` (line ~69), `${{ github.event.before }}` (lines ~70, 73), `${{ github.event.pull_request.base.sha }}` (line ~79), and `${{ github.event.pull_request.head.sha }}` (line ~80) are all expanded by the YAML template engine before the shell sees them, enabling script injection. These values should be passed via env: variables and referenced as $ENV_VAR in the shell.

Locations:

- `action.yml:63`
- `action.yml:69`
- `action.yml:70`
- `action.yml:73`
- `action.yml:75`
- `action.yml:78`
- `action.yml:79`
- `action.yml:80`

### script-injection (severity: high)

Sub-rule (a): The run: block in release-bot.yml directly interpolates `${{ steps.auth.outputs.credentials_file_path }}` inside a shell command (docker -v mount argument). Any ${{ ... }} expression inside a run: block is a script-injection risk as it is expanded by the YAML template engine before the shell processes it. The value should be passed via an env: variable instead.

Locations:

- `.github/workflows/release-bot.yml:30`

### missing-permissions (severity: medium)

The workflow file performance.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/performance.yml:1`

### missing-permissions (severity: medium)

The workflow file smoke.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file secrets.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/secrets.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, missing-permissions

**Notes:**

Fixed script injection in action.yml by moving all github context expressions (${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{ github.event.pull_request.base.sha }}, ${{ github.event.pull_request.head.sha }}) into the env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, GH_PR_HEAD_SHA, and updated shell references accordingly. Fixed script injection in release-bot.yml by moving ${{ steps.auth.outputs.credentials_file_path }} into an env: variable CREDENTIALS_FILE_PATH. Added `permissions: contents: read` to performance.yml, smoke.yml, and secrets.yml which had no permissions blocks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed four unquoted shell variable expansions in action.yml that could allow shell injection via attacker-controlled inputs:
1. `if [ $COMMIT_LENGTH == "0" ]` → `if [ "$COMMIT_LENGTH" == "0" ]` (line ~91)
2. `${BASE:-''}` → `"${BASE:-}"` in docker run command (line ~97)
3. `${HEAD:-''}` → `"${HEAD:-}"` in docker run command (line ~99)
4. `${ARGS:-''}` → `"${ARGS:-}"` in docker run command (line ~103)
All variables are already sourced from the env: block (not directly from ${{ }} expressions in the run: block), so double-quoting them is sufficient to prevent shell metacharacter injection.

