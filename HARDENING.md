<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.95.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.95.8** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ ... }} expressions are directly interpolated inside the run: shell block in action.yml. Specifically: `${{ github.event_name }}` is used in string comparisons, `${{ github.event.after }}` and `${{ github.event.before }}` are assigned to shell variables HEAD and BASE, and `${{github.event.pull_request.base.sha}}` and `${{github.event.pull_request.head.sha}}` are assigned to BASE and HEAD. Any of these values flowing through YAML template substitution before the shell parses them could allow command injection.

Locations:

- `action.yml:60`
- `action.yml:65`
- `action.yml:66`
- `action.yml:72`
- `action.yml:73`

### script-injection (severity: high)

Rule (a): `${{ steps.auth.outputs.credentials_file_path }}` is directly interpolated inside a run: shell command in release-bot.yml (used as the source path in a docker -v volume mount flag). The steps.*.outputs.* context flows through YAML template substitution before the shell parses it.

Locations:

- `.github/workflows/release-bot.yml:36`

### missing-permissions (severity: medium)

smoke.yml has no top-level `permissions:` key and none of its jobs (smoke, zombies) define job-level permissions. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

performance.yml has no top-level `permissions:` key and its only job (speed) has no job-level permissions block. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/performance.yml:1`

### missing-permissions (severity: medium)

secrets.yml has no top-level `permissions:` key and its only job (test) has no job-level permissions block. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/secrets.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, static-inline-injection

**Notes:**

Fixed all 6 findings:
1. action.yml script-injection (lines 60, 65, 66, 72, 73, 90): Moved github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, and github.event.pull_request.head.sha from inline ${{ }} expressions in the run: block to the step's env: block as EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA. Shell script updated to reference plain env vars.
2. release-bot.yml script-injection (line 36): Moved ${{ steps.auth.outputs.credentials_file_path }} to env: block as CREDENTIALS_FILE_PATH and used "$CREDENTIALS_FILE_PATH" in the docker -v flag.
3. smoke.yml missing-permissions: Added `permissions: {}` at top level.
4. performance.yml missing-permissions: Added `permissions: {}` at top level.
5. secrets.yml missing-permissions: Added `permissions: {}` at top level.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed four script injection vulnerabilities in action.yml at the docker run command (lines 103, 106, 108, 112):
1. VERSION: changed `${VERSION}` to `"${VERSION}"` (double-quoted) in the image reference
2. BASE: changed `${BASE:-''}` to `${BASE:+"$BASE"}` (guarded form — drops out when empty, double-quoted when present)
3. HEAD: changed `${HEAD:-''}` to `${HEAD:+"$HEAD"}` (same guarded form)
4. ARGS: changed `${ARGS:-''}` to `${ARGS:+"$ARGS"}` (same guarded form)

All inputs were already moved to the step's env: block (no ${{ }} expressions in the run: block), so only the shell variable expansion quoting needed to be fixed.

