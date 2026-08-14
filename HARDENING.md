<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple GitHub Actions expressions are directly interpolated inside the run: shell script in action.yml. The following expressions appear verbatim in shell commands: `${{ github.event_name }}` (used in if-conditions), `${{ github.event.after }}` (assigned to HEAD), `${{ github.event.before }}` (used in if-condition and assigned to BASE), `${{ github.event.pull_request.base.sha }}` (assigned to BASE), and `${{ github.event.pull_request.head.sha }}` (assigned to HEAD). These values flow through YAML template substitution before the shell processes them, enabling script injection. For example: `if [ "${{ github.event_name }}" == "push" ]`, `HEAD=${{ github.event.after }}`, `BASE=${{github.event.pull_request.base.sha}}`. These should be moved into env: variables and referenced as $GITHUB_EVENT_NAME etc.

Rule (b): The docker run command at the end of the run: block uses unquoted shell variable expansions: `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}`. BASE and HEAD hold values from inputs.base/inputs.head (workflow-controllable), and ARGS holds inputs.extra_args (directly attacker-controlled). Unquoted expansions allow shell metacharacter injection (semicolons, pipes, backticks, etc.). These must be double-quoted: `"${BASE:-}"`, `"${HEAD:-}"`, `"${ARGS:-}"`.

Offending lines include:
- `if [ "${{ github.event_name }}" == "push" ]`
- `HEAD=${{ github.event.after }}`
- `if [ ${{ github.event.before }} == "0000..." ]`
- `BASE=${{ github.event.before }}`
- `BASE=${{github.event.pull_request.base.sha}}`
- `HEAD=${{github.event.pull_request.head.sha}}`
- `${BASE:-''}`, `${HEAD:-''}`, `${ARGS:-''}`

Locations:

- `action.yml:71`
- `action.yml:76`
- `action.yml:77`
- `action.yml:80`
- `action.yml:82`
- `action.yml:83`
- `action.yml:84`
- `action.yml:85`
- `action.yml:86`
- `action.yml:93`
- `action.yml:95`
- `action.yml:98`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in action.yml:

1. Moved all GitHub context expressions from the run: block into the env: block as environment variables: GH_EVENT_NAME (${{ github.event_name }}), GH_EVENT_AFTER (${{ github.event.after }}), GH_EVENT_BEFORE (${{ github.event.before }}), GH_PR_BASE_SHA (${{ github.event.pull_request.base.sha }}), GH_PR_HEAD_SHA (${{ github.event.pull_request.head.sha }}). These are now referenced as plain shell variables in the script.

2. Fixed unquoted shell variable expansions in the docker run command: ${BASE:-''} and ${HEAD:-''} are now double-quoted as "${BASE:-}" and "${HEAD:-}". For ARGS (inputs.extra_args, a list of arguments), used the xargs tokenization pattern to safely split into an array and expand as "${extra_args[@]}" to handle quoted arguments correctly without shell injection.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed 4 findings across 4 workflow files: (1) release-bot.yml: moved `${{ steps.auth.outputs.credentials_file_path }}` out of the `run:` shell string into the step's `env:` block as `CREDENTIALS_FILE_PATH`, referenced as `"$CREDENTIALS_FILE_PATH"` in the docker run command to prevent script injection; (2) performance.yml: added `permissions: {}` top-level block; (3) secrets.yml: added `permissions: {}` top-level block; (4) smoke.yml: added `permissions: {}` top-level block.

