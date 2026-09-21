<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates GitHub Actions expressions (`${{ }}`) inside shell commands (rule a). Multiple `github.*` context values are interpolated directly: `${{ github.event_name }}` (used in string comparisons), `${{ github.event.after }}` (assigned to HEAD=), `${{ github.event.before }}` (used in a comparison and assigned to BASE=), `${{github.event.pull_request.base.sha}}` (assigned to BASE=), and `${{github.event.pull_request.head.sha}}` (assigned to HEAD=). Any `${{ ... }}` expression inside a `run:` shell script is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Additionally (rule b), the docker run command at the end of the script passes `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` unquoted as positional arguments. BASE and HEAD are derived from attacker-influenced `github.event.*` values and `inputs.*`; ARGS comes from `inputs.extra_args`. Unquoted shell variable expansions allow shell metacharacter injection (`;`, `|`, `&`, `$(...)`, etc.).

Locations:

- `action.yml:72`
- `action.yml:77`
- `action.yml:78`
- `action.yml:80`
- `action.yml:82`
- `action.yml:85`
- `action.yml:86`
- `action.yml:87`
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

Fixed all script injection issues in action.yml:
1. Moved all ${{ github.* }} expressions from the run: block to the env: block as named environment variables: GH_EVENT_NAME (${{ github.event_name }}), GH_EVENT_AFTER (${{ github.event.after }}), GH_EVENT_BEFORE (${{ github.event.before }}), GH_PR_BASE_SHA (${{ github.event.pull_request.base.sha }}), GH_PR_HEAD_SHA (${{ github.event.pull_request.head.sha }}). The shell script now references these as plain $VAR_NAME environment variables.
2. Fixed unquoted variable expansions in the docker run command: ${BASE:-''} and ${HEAD:-''} replaced with ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} (properly quoted, argument dropped when empty). ${ARGS:-''} replaced with a bash array (extra_args) populated via the xargs+read-loop pattern to handle quoted arguments correctly, expanded as "${extra_args[@]}".

