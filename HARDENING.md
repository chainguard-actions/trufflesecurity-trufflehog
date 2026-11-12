<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.9** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violated. The composite step's run: block puts GitHub expressions straight into the shell script. Examples: `if [ "${{ github.event_name }}" == "push" ]; then`, `HEAD=${{ github.event.after }}`, `if [ ${{ github.event.before }} == "0000..." ]`, `BASE=${{ github.event.before }}`, `BASE=${{github.event.pull_request.base.sha}}` and `HEAD=${{github.event.pull_request.head.sha}}`. Move these values into env: (for example EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA) and reference them as quoted shell variables. Rule (b) is also violated: workflow-controlled env vars are expanded without quotes in the docker run command: `${BASE:-''}`, `${HEAD:-''}` and `${ARGS:-''}` (ARGS comes from inputs.extra_args). The unquoted `$COMMIT_LENGTH` in `if [ $COMMIT_LENGTH == "0" ]` should also be quoted. Use the guarded quoted form, e.g. ${BASE:+"$BASE"}. For extra_args, split it into an array (ARGS_ARR=($ARGS) with a shellcheck SC2206 directive) and pass "${ARGS_ARR[@]}".

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

I moved github.event_name, event.after, event.before, pull_request.base.sha and pull_request.head.sha into env (EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA). The script now uses them as quoted variables, and $COMMIT_LENGTH is quoted too. In the docker run command, BASE and HEAD are now passed as "${BASE:-}" and "${HEAD:-}". When empty, this still gives one empty argument, the same as upstream's ${BASE:-''}. I didn't use the suggested ${BASE:+"$BASE"} form because it drops the argument entirely, which changes the argument count. extra_args is split with an unquoted array assignment (ARGS_ARR=($ARGS), with a shellcheck directive). That splits and globs exactly as upstream's unquoted expansion did, and an empty value still gives one empty argument as before. Nothing was run to test these changes.

