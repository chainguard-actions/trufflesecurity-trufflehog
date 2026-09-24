<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.7** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ github.* }} expressions are directly interpolated inside the run: shell script, enabling script injection. Offending lines include:
- `if [ "${{ github.event_name }}" == "push" ]`
- `HEAD=${{ github.event.after }}`
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]`
- `BASE=${{ github.event.before }}`
- `elif [ "${{ github.event_name }}" == "workflow_dispatch" ]`
- `elif [ "${{ github.event_name }}" == "pull_request" ]`
- `BASE=${{github.event.pull_request.base.sha}}`
- `HEAD=${{github.event.pull_request.head.sha}}`

These values are substituted by the GitHub Actions template engine before the shell ever sees them, allowing an attacker to inject arbitrary shell commands via crafted event payloads (e.g. a branch name or commit SHA containing shell metacharacters). All github.* context values should be passed through env: variables and then referenced as quoted shell variables instead.

Rule (b): The docker run command at the end of the script uses unquoted shell variable expansions for attacker-controlled values:
- `${BASE:-''}` (from inputs.base)
- `${HEAD:-''}` (from inputs.head)
- `${ARGS:-''}` (from inputs.extra_args)

These should be double-quoted: `"${BASE:-}"`, `"${HEAD:-}"`, `"${ARGS:-}"`.

Locations:

- `action.yml:76`
- `action.yml:83`
- `action.yml:84`
- `action.yml:87`
- `action.yml:89`
- `action.yml:91`
- `action.yml:92`
- `action.yml:93`
- `action.yml:104`
- `action.yml:106`
- `action.yml:108`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in action.yml:
1. Moved all ${{ github.* }} expressions from the run: block to the env: block: GH_EVENT_NAME (${{ github.event_name }}), GH_EVENT_AFTER (${{ github.event.after }}), GH_EVENT_BEFORE (${{ github.event.before }}), GH_PR_BASE_SHA (${{ github.event.pull_request.base.sha }}), GH_PR_HEAD_SHA (${{ github.event.pull_request.head.sha }}).
2. Replaced all inline ${{ github.* }} references in the shell script with properly double-quoted shell variable references ($GH_EVENT_NAME, $GH_EVENT_AFTER, etc.).
3. Fixed unquoted ${BASE:-''} and ${HEAD:-''} in the docker run command to properly double-quoted "${BASE:-}" and "${HEAD:-}".
4. Replaced the unquoted ${ARGS:-''} (extra_args list input) with xargs-based tokenization into a bash array (extra_args), then expanded as "${extra_args[@]}" to correctly handle multi-token arguments while preventing injection.

