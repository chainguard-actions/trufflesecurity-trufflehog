<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates multiple ${{ github.* }} expressions inside shell commands (rule a), and also uses unquoted shell variable expansions of env vars that hold untrusted input values (rule b).

Rule (a) violations — ${{ }} expressions directly in shell:
- Line 73: `if [ "${{ github.event_name }}" == "push" ]`
- Line 79: `HEAD=${{ github.event.after }}`
- Line 80: `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]`
- Line 83: `BASE=${{ github.event.before }}`
- Line 85: `elif [ "${{ github.event_name }}" == "workflow_dispatch" ] || [ "${{ github.event_name }}" == "schedule" ]`
- Line 88: `elif [ "${{ github.event_name }}" == "pull_request" ]`
- Line 89: `BASE=${{github.event.pull_request.base.sha}}`
- Line 90: `HEAD=${{github.event.pull_request.head.sha}}`

Rule (b) violations — unquoted shell variable expansions of env vars holding untrusted inputs (inputs.base, inputs.head, inputs.extra_args):
- Lines 97-103: `${BASE:-''}`, `${HEAD:-''}`, `${ARGS:-''}` passed unquoted to docker run command.

An attacker controlling github.event.after, github.event.before, or the pull_request SHAs could inject shell metacharacters. Similarly, unquoted $ARGS (from inputs.extra_args) allows word splitting and glob expansion.

Locations:

- `action.yml:73`
- `action.yml:79`
- `action.yml:80`
- `action.yml:83`
- `action.yml:85`
- `action.yml:88`
- `action.yml:89`
- `action.yml:90`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in hardened/action/action.yml:
1. Moved all ${{ github.* }} expressions from the shell run: block to the step's env: block: GH_EVENT_NAME (${{ github.event_name }}), GH_EVENT_AFTER (${{ github.event.after }}), GH_EVENT_BEFORE (${{ github.event.before }}), GH_PR_BASE_SHA (${{ github.event.pull_request.base.sha }}), GH_PR_HEAD_SHA (${{ github.event.pull_request.head.sha }}).
2. Replaced all inline ${{ }} references in the shell script with properly quoted "$VAR_NAME" references.
3. Fixed unquoted ${BASE:-''} and ${HEAD:-''} to use ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} — drops the argument when empty, properly quoted when present.
4. Fixed unquoted ${ARGS:-''} (inputs.extra_args, a list of arguments) by tokenizing via xargs into a bash array extra_args and expanding as "${extra_args[@]}" with a guard for empty input.

