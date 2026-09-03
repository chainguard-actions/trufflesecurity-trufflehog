<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.3** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ ... }} expressions are directly interpolated inside the run: shell script in action.yml, violating rule (a). Specifically: `${{ github.event_name }}` (lines ~68, 77, 79), `${{ github.event.after }}` (line ~73), `${{ github.event.before }}` (line ~74), `${{github.event.pull_request.base.sha}}` (line ~80), and `${{github.event.pull_request.head.sha}}` (line ~81) are all substituted directly into shell code before the shell parses it. An attacker who can influence these values (e.g. via a crafted event payload) could inject arbitrary shell commands. Additionally, rule (b) is violated: the env vars $BASE, $HEAD, and $ARGS (sourced from inputs.*) are expanded unquoted in the docker run command as `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}`, allowing shell metacharacter injection.

Locations:

- `action.yml:68`
- `action.yml:73`
- `action.yml:74`
- `action.yml:77`
- `action.yml:79`
- `action.yml:80`
- `action.yml:81`
- `action.yml:91`
- `action.yml:93`
- `action.yml:95`

### missing-permissions (severity: medium)

The workflow file performance.yml has no top-level `permissions:` key and the job 'speed' also has no job-level `permissions:` key, leaving the workflow with default (potentially broad) permissions.

Locations:

- `.github/workflows/performance.yml:1`

### missing-permissions (severity: medium)

The workflow file smoke.yml has no top-level `permissions:` key and neither job ('smoke' nor 'zombies') has a job-level `permissions:` key, leaving the workflow with default (potentially broad) permissions.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file secrets.yml has no top-level `permissions:` key and the job 'test' also has no job-level `permissions:` key, leaving the workflow with default (potentially broad) permissions.

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

Fixed action.yml script injection by moving all ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{github.event.pull_request.base.sha}}, and ${{github.event.pull_request.head.sha}} expressions into the env: block as EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA. Fixed unquoted ${BASE:-''} and ${HEAD:-''} to use ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} (safe conditional expansion). Fixed ${ARGS:-''} (a list-style extra_args input) using xargs-based tokenization into a bash array. Added permissions: {} to performance.yml, smoke.yml, and secrets.yml workflow files.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In .github/workflows/release-bot.yml, moved `${{ steps.auth.outputs.credentials_file_path }}` out of the `run:` shell command and into the step's `env:` block as `CREDENTIALS_FILE_PATH`. The docker `-v` flag now references `"$CREDENTIALS_FILE_PATH"` as a quoted shell variable, preventing shell metacharacter injection.

