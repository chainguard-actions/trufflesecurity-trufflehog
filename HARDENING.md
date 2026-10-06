<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.98.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.98.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ github.* }} expressions are directly interpolated inside the run: shell script in action.yml. The following lines embed GitHub context values directly into shell commands without routing through env: variables: (1) `if [ "${{ github.event_name }}" == "push" ]` — github.event_name interpolated directly; (2) `HEAD=${{ github.event.after }}` — github.event.after interpolated directly; (3) `if [ ${{ github.event.before }} == "0000..." ]` — github.event.before interpolated directly and unquoted; (4) `BASE=${{ github.event.before }}` — github.event.before interpolated directly; (5) `elif [ "${{ github.event_name }}" == "workflow_dispatch" ] || [ "${{ github.event_name }}" == "schedule" ]` — github.event_name interpolated directly; (6) `elif [ "${{ github.event_name }}" == "pull_request" ]` — github.event_name interpolated directly; (7) `BASE=${{github.event.pull_request.base.sha}}` — interpolated directly; (8) `HEAD=${{github.event.pull_request.head.sha}}` — interpolated directly. These values flow through YAML template substitution before the shell processes them, enabling script injection.

Locations:

- `action.yml:72`
- `action.yml:77`
- `action.yml:78`
- `action.yml:81`
- `action.yml:83`
- `action.yml:86`
- `action.yml:87`
- `action.yml:88`

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansions of untrusted data in the docker run command. The env vars BASE (from inputs.base), HEAD (from inputs.head), and ARGS (from inputs.extra_args) are expanded without double-quoting: `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` are passed as unquoted positional arguments to docker run. An attacker-controlled value containing shell metacharacters (spaces, semicolons, pipes, glob characters, etc.) could alter the command structure. These should be `"${BASE:-}"`, `"${HEAD:-}"`, and `"${ARGS:-}"` or use the guarded form.

Locations:

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

Fixed all script injection findings in hardened/action/action.yml:
1. Moved all ${{ github.* }} expressions (github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, github.event.pull_request.head.sha) from inline run: shell strings to the step's env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, GH_PR_HEAD_SHA. All references in the shell script now use plain $VAR_NAME syntax.
2. Replaced unquoted ${BASE:-''}, ${HEAD:-''}, ${ARGS:-''} in the docker run command with properly quoted constructs: BASE and HEAD use a guarded bash array (args) with double-quoted values; ARGS (a list input) uses the xargs-based quote-aware tokenization pattern into extra_args array, both expanded with "${args[@]}" and "${extra_args[@]}".

