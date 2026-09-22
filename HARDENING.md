<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ github.* }} expressions are directly interpolated inside the run: shell block in action.yml. These values flow through YAML template substitution before the shell processes them, enabling script injection. Offending lines include:
- `if [ "${{ github.event_name }}" == "push" ]`
- `HEAD=${{ github.event.after }}`
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]`
- `BASE=${{ github.event.before }}`
- `elif [ "${{ github.event_name }}" == "workflow_dispatch" ]`
- `elif [ "${{ github.event_name }}" == "pull_request" ]`
- `BASE=${{github.event.pull_request.base.sha}}`
- `HEAD=${{github.event.pull_request.head.sha}}`

Rule (b): The shell variables `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` are expanded unquoted in the docker run command. These variables hold values sourced from `inputs.*` and `github.*` contexts (set in the env: block), so unquoted expansion allows shell metacharacter injection.

Locations:

- `action.yml:57`
- `action.yml:62`
- `action.yml:63`
- `action.yml:65`
- `action.yml:68`
- `action.yml:70`
- `action.yml:72`
- `action.yml:73`
- `action.yml:82`
- `action.yml:84`
- `action.yml:88`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in action.yml:
1. Moved all ${{ github.* }} expressions (github.event_name, github.event.after, github.event.before, github.event.pull_request.base.sha, github.event.pull_request.head.sha) from the run: shell block into the env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, and GH_PR_HEAD_SHA respectively.
2. Updated all references in the run: block to use the environment variable names instead of template expressions.
3. Replaced unquoted ${BASE:-''}, ${HEAD:-''}, and ${ARGS:-''} expansions in the docker run command with properly quoted bash arrays: docker_args (built conditionally with --since-commit "$BASE" and --branch "$HEAD" only when non-empty) and extra_args (populated via xargs-based quote-aware tokenization for the list-style ARGS input), both expanded with "${array[@]}" syntax.

