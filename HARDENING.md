<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.4** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates GitHub Actions expressions inside shell commands (sub-rule a). The following expressions are embedded directly in the shell script without going through env: variables or any quoting protection: `${{ github.event_name }}` (used in if-conditions), `${{ github.event.after }}` (assigned to HEAD), `${{ github.event.before }}` (used in if-condition and assigned to BASE), `${{github.event.pull_request.base.sha}}` (assigned to BASE), and `${{github.event.pull_request.head.sha}}` (assigned to HEAD). Any ${{ ... }} expression interpolated directly into a run: block is a script-injection risk because the value is substituted into the shell script before the shell parses it, allowing an attacker to inject shell metacharacters.

Locations:

- `action.yml:68`
- `action.yml:74`
- `action.yml:75`
- `action.yml:78`
- `action.yml:80`
- `action.yml:83`
- `action.yml:84`
- `action.yml:85`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents and other scopes). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/performance.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Explicit minimal permissions should be declared.

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

Fixed script injection in action.yml by moving all ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{github.event.pull_request.base.sha}}, and ${{github.event.pull_request.head.sha}} expressions from the run: block into the env: block as EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, and PR_HEAD_SHA respectively. The shell script now references these as properly-quoted environment variables. Added `permissions: contents: read` to .github/workflows/performance.yml, .github/workflows/smoke.yml, and .github/workflows/secrets.yml to address the missing-permissions findings.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection findings:

1. action.yml: Replaced unquoted `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` in the docker run command. BASE and HEAD (single optional values) now use the safe `${BASE:+"$BASE"}` / `${HEAD:+"$HEAD"}` form. ARGS (a list of arguments) is now tokenized via xargs into a bash array `extra_args` and expanded as `"${extra_args[@]}"`.

2. release-bot.yml: Moved `${{ steps.auth.outputs.credentials_file_path }}` out of the `run:` block into an `env:` variable `CREDENTIALS_FILE_PATH`, then referenced it as `"$CREDENTIALS_FILE_PATH"` in the shell script to prevent template injection.

