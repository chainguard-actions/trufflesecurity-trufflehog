<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ github.event.* }} expressions are directly interpolated inside the run: shell script in action.yml. This allows an attacker to inject arbitrary shell commands via crafted commit SHAs or event metadata. Offending lines include:
- `if [ "${{ github.event_name }}" == "push" ];` (line ~65)
- `HEAD=${{ github.event.after }}` (line ~70)
- `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ];` (line ~71)
- `BASE=${{ github.event.before }}` (line ~74)
- `elif [ "${{ github.event_name }}" == "workflow_dispatch" ]` (line ~76)
- `elif [ "${{ github.event_name }}" == "pull_request" ];` (line ~79)
- `BASE=${{github.event.pull_request.base.sha}}` (line ~80)
- `HEAD=${{github.event.pull_request.head.sha}}` (line ~81)

Rule (b): The variables `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` are used unquoted in the docker run command. These hold values sourced from `inputs.*` (via the env: block) and from direct ${{ }} interpolation. Unquoted expansion allows shell metacharacter injection (e.g., spaces, semicolons, glob characters). Offending lines in the docker run command block (~lines 95-107).

Locations:

- `action.yml:65`
- `action.yml:70`
- `action.yml:71`
- `action.yml:74`
- `action.yml:76`
- `action.yml:79`
- `action.yml:80`
- `action.yml:81`
- `action.yml:101`
- `action.yml:103`
- `action.yml:105`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in action.yml:
1. Moved all ${{ github.event.* }} expressions (event_name, event.after, event.before, event.pull_request.base.sha, event.pull_request.head.sha) from the run: shell script to the env: block as EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, EVENT_PR_BASE_SHA, EVENT_PR_HEAD_SHA. Shell script now references these as properly quoted environment variables.
2. Fixed unquoted variable expansions in the docker run command: ${BASE:-''} and ${HEAD:-''} replaced with ${BASE:+"$BASE"} and ${HEAD:+"$HEAD"} (properly quoted, drops argument when empty to avoid passing empty positional args). ${ARGS:-''} replaced with a properly tokenized bash array using xargs (handles quoted arguments correctly, guards against empty input).

