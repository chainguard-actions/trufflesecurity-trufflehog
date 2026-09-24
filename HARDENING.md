<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.9** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates multiple ${{ github.* }} expressions into shell command strings (sub-rule a). The following expressions appear unquoted or directly substituted inside shell commands:
- `${{ github.event_name }}` used in `if [ "${{ github.event_name }}" == "push" ]` and similar comparisons
- `HEAD=${{ github.event.after }}` — direct assignment without quoting
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]` — unquoted in a test expression
- `BASE=${{ github.event.before }}` — direct assignment without quoting
- `BASE=${{github.event.pull_request.base.sha}}` — direct assignment without quoting
- `HEAD=${{github.event.pull_request.head.sha}}` — direct assignment without quoting

Any ${{ ... }} expression interpolated directly inside a run: shell script is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, allowing shell metacharacters to be injected. These values should be passed via env: variables and then referenced as quoted shell variables (e.g., "$VAR").

Locations:

- `action.yml:55`
- `action.yml:57`
- `action.yml:58`
- `action.yml:61`
- `action.yml:63`
- `action.yml:64`
- `action.yml:65`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved all github.* context expressions from the run: shell block into the env: block. Added five new env vars: EVENT_NAME (${{ github.event_name }}), EVENT_AFTER (${{ github.event.after }}), EVENT_BEFORE (${{ github.event.before }}), PR_BASE_SHA (${{ github.event.pull_request.base.sha }}), PR_HEAD_SHA (${{ github.event.pull_request.head.sha }}). Updated the shell script to reference these as quoted shell variables ($EVENT_NAME, $EVENT_AFTER, $EVENT_BEFORE, $PR_BASE_SHA, $PR_HEAD_SHA) instead of inline template expressions. This eliminates the script injection risk from all identified locations (lines 55, 57, 58, 61, 63, 64, 65, and 95).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted variable expansions in the docker run command:
1. `${BASE:-''}` → `"${BASE:-}"` (double-quoted single value)
2. `${HEAD:-''}` → `"${HEAD:-}"` (double-quoted single value)
3. `${ARGS:-''}` → tokenized via xargs into bash array `extra_args`, expanded as `"${extra_args[@]}"` (quote-aware tokenization for list input)

The xargs tokenization uses the standard guard pattern (`if [ -n "$ARGS" ]`) to prevent empty-token emission, and uses `printf '%s' | xargs printf '%s\0'` with a NUL-delimited read loop for correct argument boundary preservation. The step uses `shell: bash` so bash arrays and process substitution are available.

