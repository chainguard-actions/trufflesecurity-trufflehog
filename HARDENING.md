<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the composite step in action.yml puts GitHub expressions straight into the run: script. Offending lines: `if [ "${{ github.event_name }}" == "push" ]`, `HEAD=${{ github.event.after }}`, `if [ ${{ github.event.before }} == "0000..." ]`, `BASE=${{ github.event.before }}`, `elif [ "${{ github.event_name }}" == "workflow_dispatch" ] || [ "${{ github.event_name }}" == "schedule" ]`, `elif [ "${{ github.event_name }}" == "pull_request" ]`, `BASE=${{github.event.pull_request.base.sha}}`, `HEAD=${{github.event.pull_request.head.sha}}`. Rule (b): variables that come from inputs are expanded without quotes: `${BASE:-''}`, `${HEAD:-''}` and `${ARGS:-''}` (ARGS is inputs.extra_args), plus `[ $COMMIT_LENGTH == "0" ]`. Fix: move the github.* values into env: variables and double-quote every expansion. For optional values, use the guarded form ${VAR:+"$VAR"}.

Locations:

- `action.yml:78`
- `action.yml:84`
- `action.yml:85`
- `action.yml:88`
- `action.yml:90`
- `action.yml:93`
- `action.yml:94`
- `action.yml:95`
- `action.yml:105`
- `action.yml:107`
- `action.yml:111`

### static-inline-injection (severity: high)

shell injection: expression "${{github.event.pull_request.head.sha}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved github.event_name, event.after, event.before, pull_request.base.sha and pull_request.head.sha into env: (EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA) and changed the script to use quoted $VAR references. Quoted $COMMIT_LENGTH. The --since-commit/--branch values are now "${BASE:-}" and "${HEAD:-}", which still passes an empty argument when the value is empty, as upstream's ${BASE:-''} did. extra_args is a whitespace-separated list that upstream expanded unquoted, so it is now split into a bash array with read -r -d '' -a (no quote handling, same as upstream). One empty argument is still passed when it is empty, to match upstream's ${ARGS:-''}.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Replaced every ${{ }} in the run script with existing env vars (EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA). BASE/HEAD are now passed as "$BASE"/"$HEAD" rather than ${BASE:+...}, because upstream's ${BASE:-''} always passed an argument (an empty one when unset). Dropping it would make --since-commit take '--branch' as its value. extra_args is split on whitespace into a bash array with read -d '' (same as upstream's unquoted expansion, which did not evaluate quotes). When it is empty, one empty argument is passed, as upstream did.

