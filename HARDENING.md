<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.98.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.98.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ github.* }} expressions are directly interpolated inside the run: shell script in action.yml. Specifically:
- `if [ "${{ github.event_name }}" == "push" ]` (and similar comparisons) — github.event_name is interpolated directly into the shell string.
- `HEAD=${{ github.event.after }}` — github.event.after is interpolated directly into a shell variable assignment.
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]` — github.event.before is interpolated directly into a shell comparison.
- `BASE=${{ github.event.before }}` — github.event.before is interpolated directly into a shell variable assignment.
- `BASE=${{github.event.pull_request.base.sha}}` — pull_request.base.sha is interpolated directly.
- `HEAD=${{github.event.pull_request.head.sha}}` — pull_request.head.sha is interpolated directly.
All of these values flow through YAML template substitution before the shell processes them, making them potential script injection vectors. They should be passed via env: variables and referenced as $ENV_VAR instead.

Sub-rule (b): The shell variables `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` are used unquoted in the `docker run` command at the end of the run: block. `ARGS` is sourced from `inputs.extra_args` (attacker-controlled), and BASE/HEAD can be set from attacker-controlled github context values. Unquoted expansions allow shell metacharacter injection.

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in action.yml:
1. Moved all github.* context expressions from inline run: shell strings to the step's env: block: GH_EVENT_NAME (${{ github.event_name }}), GH_EVENT_AFTER (${{ github.event.after }}), GH_EVENT_BEFORE (${{ github.event.before }}), GH_PR_BASE_SHA (${{ github.event.pull_request.base.sha }}), GH_PR_HEAD_SHA (${{ github.event.pull_request.head.sha }}).
2. Replaced all inline ${{ }} references in the run: block with plain $ENV_VAR references.
3. Fixed unquoted ${BASE:-''} and ${HEAD:-''} in docker run command to use ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} (properly quoted, drops argument when empty).
4. Replaced unquoted ${ARGS:-''} with a bash array (extra_args) populated via xargs tokenization to correctly handle quoted arguments in extra_args input, expanded as "${extra_args[@]}".

