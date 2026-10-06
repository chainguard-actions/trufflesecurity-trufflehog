<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates GitHub Actions expressions inside shell commands, violating rule (a). Multiple ${{ github.* }} expressions are embedded directly in the shell script:
- `if [ "${{ github.event_name }}" == "push" ]` — github.event_name interpolated directly into shell
- `HEAD=${{ github.event.after }}` — github.event.after interpolated directly into shell variable assignment
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]` — github.event.before interpolated directly into shell condition (also unquoted)
- `BASE=${{ github.event.before }}` — github.event.before interpolated directly
- `elif [ "${{ github.event_name }}" == "workflow_dispatch" ]` — repeated interpolation
- `elif [ "${{ github.event_name }}" == "pull_request" ]` — repeated interpolation
- `BASE=${{github.event.pull_request.base.sha}}` — pull request base SHA interpolated directly
- `HEAD=${{github.event.pull_request.head.sha}}` — pull request head SHA interpolated directly

Additionally, rule (b) is violated: `${BASE:-''}` and `${HEAD:-''}` are used unquoted in the docker run command, where BASE and HEAD were assigned from the above tainted ${{ github.* }} expressions. An attacker controlling these values (e.g. via a crafted branch name or commit SHA) could inject shell metacharacters.

Locations:

- `action.yml:76`
- `action.yml:81`
- `action.yml:82`
- `action.yml:84`
- `action.yml:87`
- `action.yml:89`
- `action.yml:90`
- `action.yml:91`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection findings in hardened/action/action.yml by moving all ${{ github.* }} expressions from the run: block into the env: block. Added five new environment variables: GH_EVENT_NAME (${{ github.event_name }}), GH_EVENT_AFTER (${{ github.event.after }}), GH_EVENT_BEFORE (${{ github.event.before }}), GH_PR_BASE_SHA (${{ github.event.pull_request.base.sha }}), and GH_PR_HEAD_SHA (${{ github.event.pull_request.head.sha }}). Updated the shell script to reference these as properly quoted environment variables ($GH_EVENT_NAME, $GH_EVENT_AFTER, etc.) instead of inline GitHub expressions, eliminating the risk of shell injection via attacker-controlled values like branch names or commit SHAs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed four unquoted shell variable expansions in action.yml:
1. Quoted `$COMMIT_LENGTH` in the `if` test condition: `if [ "$COMMIT_LENGTH" == "0" ]`
2. Double-quoted `${BASE:-''}` → `"${BASE:-}"` in the docker run command
3. Double-quoted `${HEAD:-''}` → `"${HEAD:-}"` in the docker run command
4. Replaced unquoted `${ARGS:-''}` (a list-type input) with xargs-based tokenization into a bash array `extra_args`, then expanded as `"${extra_args[@]}"` to preserve argument boundaries while preventing shell metacharacter injection

