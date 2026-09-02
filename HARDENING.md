<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates multiple GitHub Actions expressions inside shell command strings (rule a), and also uses unquoted shell variable expansions of workflow-controllable values (rule b).

Rule (a) violations — direct ${{ }} interpolation inside run: shell commands:
- `if [ "${{ github.event_name }}" == "push" ]` (line ~72)
- `HEAD=${{ github.event.after }}` (line ~77, also unquoted)
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]` (line ~78, unquoted)
- `elif [ "${{ github.event_name }}" == "workflow_dispatch" ] || [ "${{ github.event_name }}" == "schedule" ]` (line ~82)
- `elif [ "${{ github.event_name }}" == "pull_request" ]` (line ~84)
- `BASE=${{github.event.pull_request.base.sha}}` (line ~85, unquoted)
- `HEAD=${{github.event.pull_request.head.sha}}` (line ~86, unquoted)

Rule (b) violations — unquoted shell variable expansions of workflow-controllable env vars in the docker run command:
- `${BASE:-''}` (holds inputs.base, unquoted positional arg)
- `${HEAD:-''}` (holds inputs.head, unquoted positional arg)
- `${ARGS:-''}` (holds inputs.extra_args, unquoted positional arg)

An attacker controlling github.event.after, github.event.before, github.event.pull_request.base.sha, or inputs.extra_args could inject arbitrary shell commands.

Locations:

- `action.yml:72`
- `action.yml:77`
- `action.yml:78`
- `action.yml:82`
- `action.yml:84`
- `action.yml:85`
- `action.yml:86`
- `action.yml:97`
- `action.yml:99`
- `action.yml:101`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection vulnerabilities in action.yml:
1. Moved all ${{ github.* }} expressions (github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, github.event.pull_request.head.sha) into the step's env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, GH_PR_HEAD_SHA.
2. Replaced all direct ${{ }} interpolations inside the run: shell script with plain env var references ($GH_EVENT_NAME, etc.).
3. Fixed unquoted ${BASE:-''} and ${HEAD:-''} positional args with ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} (safe single-optional-value pattern).
4. Fixed unquoted ${ARGS:-''} (inputs.extra_args — a whitespace-separated argument list) by tokenizing it with xargs into a bash array (extra_args) and expanding it as "${extra_args[@]}" — preserving quote-aware tokenization without injection risk.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed 4 findings across 4 workflow files: (1) release-bot.yml: moved `${{ steps.auth.outputs.credentials_file_path }}` out of the run: block into the step's env: block as CREDENTIALS_FILE_PATH, eliminating the script-injection risk; (2) smoke.yml: added top-level `permissions: contents: read`; (3) performance.yml: added top-level `permissions: contents: read`; (4) secrets.yml: added top-level `permissions: contents: read`. The `contents: read` permission is the minimum needed for the `actions/checkout` steps present in each workflow.

