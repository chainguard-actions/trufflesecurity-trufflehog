<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple GitHub Actions expressions are directly interpolated inside the run: shell script in action.yml. The expressions ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{github.event.pull_request.base.sha}}, and ${{github.event.pull_request.head.sha}} are substituted directly into shell commands before the shell parses them, enabling script injection. For example: `if [ "${{ github.event_name }}" == "push" ]`, `HEAD=${{ github.event.after }}`, `if [ ${{ github.event.before }} == "0000..."`, `BASE=${{github.event.pull_request.base.sha}}`, `HEAD=${{github.event.pull_request.head.sha}}`. Rule (b): The env vars $BASE, $HEAD, and $ARGS (sourced from inputs.base, inputs.head, inputs.extra_args) are expanded unquoted in the docker run command: `${BASE:-''}`, `${HEAD:-''}`, `${ARGS:-''}` — allowing shell metacharacter injection.

Locations:

- `action.yml:34`

### script-injection (severity: high)

Rule (a): In release-bot.yml, the expression ${{ steps.auth.outputs.credentials_file_path }} is directly interpolated inside a run: shell command: `-v ${{ steps.auth.outputs.credentials_file_path }}:/tmp/keys/GCP_SA_TRUFFLE_RELEASE_BOT.json:ro`. The steps.*.outputs.* context is listed as an untrusted source and its value is substituted into the shell command string before the shell parses it.

Locations:

- `.github/workflows/release-bot.yml:31`

### missing-permissions (severity: medium)

The workflow file secrets.yml has no top-level permissions: key and no job-level permissions: key on its only job. Without explicit permissions, the workflow inherits the repository default (which may be write-all for private repos or read-all for public repos), granting broader access than necessary.

Locations:

- `.github/workflows/secrets.yml:1`

### missing-permissions (severity: medium)

The workflow file smoke.yml has no top-level permissions: key and neither of its two jobs (smoke, zombies) has a job-level permissions: key. Without explicit permissions, the workflow inherits the repository default, granting broader access than necessary.

Locations:

- `.github/workflows/smoke.yml:1`

### missing-permissions (severity: medium)

The workflow file performance.yml has no top-level permissions: key and its only job (speed) has no job-level permissions: key. Without explicit permissions, the workflow inherits the repository default, granting broader access than necessary.

Locations:

- `.github/workflows/performance.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 6 findings:

1. action.yml script-injection (lines 34 & 95): Moved all 5 GitHub context expressions (github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, github.event.pull_request.head.sha) into the step's env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, GH_PR_HEAD_SHA. Fixed unquoted ${BASE:-''} and ${HEAD:-''} to use ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} for safe optional single-value expansion. Fixed ${ARGS:-''} by tokenizing the list input with xargs into a bash array (extra_args) to properly handle whitespace-separated arguments.

2. release-bot.yml script-injection (line 31): Moved ${{ steps.auth.outputs.credentials_file_path }} to the env: block as CREDENTIALS_FILE_PATH and referenced it as "$CREDENTIALS_FILE_PATH" in the docker run command.

3. secrets.yml missing-permissions: Added top-level `permissions: contents: read`.

4. smoke.yml missing-permissions: Added top-level `permissions: contents: read`.

5. performance.yml missing-permissions: Added top-level `permissions: contents: read`.

