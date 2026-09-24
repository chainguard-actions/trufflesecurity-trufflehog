<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.97.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.97.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates GitHub Actions expressions (${{ ... }}) inside the shell script, violating rule (a). Multiple github.* context values are interpolated directly:
- Line ~62: `if [ "${{ github.event_name }}" == "push" ]`
- Line ~65: `HEAD=${{ github.event.after }}`
- Line ~66: `if [ ${{ github.event.before }} == "0000000000000000000000000000000000000000" ]`
- Line ~68: `BASE=${{ github.event.before }}`
- Line ~67/70: `elif [ "${{ github.event_name }}" == "workflow_dispatch" ]` / `elif [ "${{ github.event_name }}" == "pull_request" ]`
- Line ~71: `BASE=${{github.event.pull_request.base.sha}}`
- Line ~72: `HEAD=${{github.event.pull_request.head.sha}}`

Any of these values are substituted into the shell script before the shell parses it, allowing an attacker to inject shell metacharacters.

Additionally, rule (b) is violated: the docker run command uses unquoted expansions of untrusted env vars: `${BASE:-''}`, `${HEAD:-''}`, and `${ARGS:-''}` (where ARGS comes from inputs.extra_args). These should be double-quoted: `"${BASE:-}"`, `"${HEAD:-}"`, `"${ARGS:-}"` to prevent shell word-splitting and glob expansion.

Locations:

- `action.yml:62`
- `action.yml:65`
- `action.yml:66`
- `action.yml:68`
- `action.yml:70`
- `action.yml:71`
- `action.yml:72`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed both findings in action.yml:
1. Moved all github.* context expressions from the run: block into the env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA, and GH_PR_HEAD_SHA. The shell script now references these as plain environment variables, eliminating shell injection risk.
2. Fixed unquoted expansions: ${BASE:-''} and ${HEAD:-''} are now "${BASE:-}" and "${HEAD:-}". The ARGS (extra_args) input is now properly tokenized using xargs into a bash array (extra_args) and expanded as "${extra_args[@]}" to correctly handle multi-argument inputs while preserving argument boundaries.

