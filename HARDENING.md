<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.96.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.96.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ github.* }} expressions are directly interpolated inside the run: shell script in action.yml, allowing an attacker to inject arbitrary shell commands. Offending lines include:
- `if [ "${{ github.event_name }}" == "push" ]` — github.event_name interpolated directly into shell
- `HEAD=${{ github.event.after }}` — github.event.after interpolated directly, unquoted
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]` — github.event.before interpolated directly, unquoted
- `elif [ "${{ github.event_name }}" == "workflow_dispatch" ]` — repeated github.event_name interpolation
- `elif [ "${{ github.event_name }}" == "pull_request" ]` — repeated github.event_name interpolation
- `BASE=${{github.event.pull_request.base.sha}}` — pull_request.base.sha interpolated directly, unquoted
- `HEAD=${{github.event.pull_request.head.sha}}` — pull_request.head.sha interpolated directly, unquoted

Sub-rule (b): Unquoted shell variable expansions of workflow-controllable data:
- `${BASE:-''}`, `${HEAD:-''}`, `${ARGS:-''}` passed unquoted to `docker run` — shell metacharacters in these values (sourced from inputs.*) would be parsed by the shell
- `[ $COMMIT_LENGTH == "0" ]` — $COMMIT_LENGTH is unquoted in the test expression

Locations:

- `action.yml:76`
- `action.yml:80`
- `action.yml:81`
- `action.yml:83`
- `action.yml:84`
- `action.yml:87`
- `action.yml:88`
- `action.yml:89`
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

Fixed all script injection findings in action.yml by moving github.* expressions out of the run: shell script and into the step's env: block: github.event_name → GH_EVENT_NAME, github.event.after → GH_EVENT_AFTER, github.event.before → GH_EVENT_BEFORE, github.event.pull_request.base.sha → GH_PR_BASE_SHA, github.event.pull_request.head.sha → GH_PR_HEAD_SHA. All shell references were updated to use the safe $VAR_NAME form with proper double-quoting. Also fixed unquoted variable expansions: $COMMIT_LENGTH in test expression now quoted as "$COMMIT_LENGTH"; ${BASE:-''} and ${HEAD:-''} now quoted as "${BASE:-}" and "${HEAD:-}". The ARGS variable (extra_args input, a list of CLI flags) remains unquoted for proper word-splitting as it is an argument list.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced the unquoted `${ARGS:-}` expansion in the `docker run` command with a proper xargs-based array tokenization. The `extra_args` input is an argument list, so the fix initializes a bash array, guards against empty input, uses `printf '%s' "$ARGS" | xargs -r printf '%s\0'` for quote-aware tokenization into NUL-delimited tokens, reads them into the array, and expands as `"${extra_args[@]}"` in the docker run invocation. This prevents shell injection while correctly handling multiple arguments including quoted ones.

