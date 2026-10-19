<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.3** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple GitHub Actions expressions are interpolated directly inside the run: shell script, bypassing shell quoting. The following ${{ }} expressions appear literally in shell command strings:
- Line 72: `if [ "${{ github.event_name }}" == "push" ]` — github.event_name is attacker-influenced via event payloads
- Line 79: `HEAD=${{ github.event.after }}` — unquoted assignment from event data
- Line 80: `if [ ${{ github.event.before }} == "0000...` — unquoted comparison
- Line 83: `BASE=${{ github.event.before }}` — unquoted assignment
- Line 85: `elif [ "${{ github.event_name }}" == "workflow_dispatch" ] || [ "${{ github.event_name }}" == "schedule" ]`
- Line 87: `elif [ "${{ github.event_name }}" == "pull_request" ]`
- Line 88: `BASE=${{github.event.pull_request.base.sha}}`
- Line 89: `HEAD=${{github.event.pull_request.head.sha}}`

Any of these values could contain shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) injected via a crafted push/PR event, leading to arbitrary command execution on the runner.

Rule (b): The docker run command at lines ~95–101 expands `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` without double-quoting. BASE and HEAD are populated from workflow-controllable inputs or from the ${{ github.event.* }} expressions above; ARGS comes from inputs.extra_args. Unquoted expansions allow the shell to parse metacharacters out of these values.

Locations:

- `action.yml:72`
- `action.yml:79`
- `action.yml:80`
- `action.yml:83`
- `action.yml:85`
- `action.yml:87`
- `action.yml:88`
- `action.yml:89`
- `action.yml:96`
- `action.yml:98`
- `action.yml:101`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection findings in action.yml:
1. Moved all ${{ github.event_name }}, ${{ github.event.after }}, ${{ github.event.before }}, ${{ github.event.pull_request.base.sha }}, and ${{ github.event.pull_request.head.sha }} expressions from the run: shell script into the env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, and GH_PR_HEAD_SHA respectively.
2. Updated all references in the shell script to use the plain environment variable names (e.g., "$GH_EVENT_NAME" instead of "${{ github.event_name }}").
3. Fixed unquoted ${BASE:-''} and ${HEAD:-''} expansions to use ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} which properly quote the values and drop the argument entirely when empty.
4. Fixed unquoted ${ARGS:-''} by tokenizing ARGS into a bash array using 'read -r -d '' -a args <<< "$ARGS" || true' (whitespace split matching upstream's behavior) and expanding as "${args[@]}".

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in action.yml line 63: changed `[ $COMMIT_LENGTH == "0" ]` to `[ "$COMMIT_LENGTH" == "0" ]`. The variable COMMIT_LENGTH is derived from COMMIT_IDS which is set from the workflow-controllable expression `${{ toJson(github.event.commits.*.id) }}`, making it transitively untrusted. Double-quoting prevents word-splitting or metacharacter injection.

