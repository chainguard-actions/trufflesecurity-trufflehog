<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.8** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): the composite run step in action.yml puts GitHub expressions directly into the shell script. Offending lines: `if [ "${{ github.event_name }}" == "push" ]; then` (line 78), `HEAD=${{ github.event.after }}` (84), `if [ ${{ github.event.before }} == "0000..." ]` (85), `BASE=${{ github.event.before }}` (88), the `${{ github.event_name }}` comparisons for workflow_dispatch, schedule and pull_request (90, 93), `BASE=${{github.event.pull_request.base.sha}}` (94) and `HEAD=${{github.event.pull_request.head.sha}}` (95). Fix: pass these values in through env: variables and reference them double-quoted, or use the default variable $GITHUB_EVENT_NAME.

Locations:

- `action.yml:78`
- `action.yml:84`
- `action.yml:85`
- `action.yml:88`
- `action.yml:90`
- `action.yml:93`
- `action.yml:94`
- `action.yml:95`

### script-injection (severity: high)

Sub-rule (b): variables built from inputs are expanded without double quotes in the docker run command. Offending lines: `${BASE:-''}` (line 105, from inputs.base), `${HEAD:-''}` (line 107, from inputs.head) and `${ARGS:-''}` (line 112, from inputs.extra_args). Unquoted expansion allows word splitting and glob expansion of values the caller controls. Use the guarded form `${BASE:+"$BASE"}` / `${HEAD:+"$HEAD"}`, and parse ARGS safely, for example into a bash array with `read -ra`, then expand it quoted as `"${args[@]}"`.

Locations:

- `action.yml:105`
- `action.yml:107`
- `action.yml:112`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved github.event_name, event.before, event.after and the pull_request base/head SHAs into step env: variables (EVENT_NAME, EVENT_BEFORE, EVENT_AFTER, PR_BASE_SHA, PR_HEAD_SHA). The script now uses them double-quoted. BASE and HEAD are passed as "${BASE:-}" and "${HEAD:-}". Upstream's ${BASE:-''} gave an empty argument when the value was unset, so this keeps the same argument count for --since-commit and --branch. extra_args is now split on whitespace into a bash array with read -r -d '' -a, the same way upstream's unquoted $ARGS was split, and expanded as "${args[@]}". When extra_args is empty, the array holds one empty argument, which matches what upstream's ${ARGS:-''} passed.

