<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.96.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.96.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml composite run step (lines 45-111) breaks both sub-rules. (a) github.* expressions are placed directly in the run script: line 78 `if [ "${{ github.event_name }}" == "push" ]`, line 84 `HEAD=${{ github.event.after }}`, line 85 `if [ ${{ github.event.before }} == ...`, line 88 `BASE=${{ github.event.before }}`, lines 90/93 `${{ github.event_name }}` comparisons, line 94 `BASE=${{github.event.pull_request.base.sha}}`, line 95 `HEAD=${{github.event.pull_request.head.sha}}`. (b) Variables that hold the base/head/extra_args inputs are expanded unquoted in the docker run command: line 105 `${BASE:-''}`, line 107 `${HEAD:-''}`, line 111 `${ARGS:-''}`, which lets the shell split and expand them. Fix: move the github.* values into env: variables and quote them, and use the guarded form ${VAR:+"$VAR"} (or a bash array for ARGS if it must carry several flags).

Locations:

- `action.yml:78`
- `action.yml:84`
- `action.yml:85`
- `action.yml:88`
- `action.yml:90`
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

I fixed both findings in action.yml. Nothing has been run or tested yet.

- **Expressions moved out of the script:** the five `github.*` expressions (event_name, event.after, event.before, pull_request.base.sha and pull_request.head.sha) are now in the step's env: block as GH_EVENT_NAME, GH_EVENT_AFTER, GH_EVENT_BEFORE, GH_PR_BASE_SHA and GH_PR_HEAD_SHA. The script only uses them as quoted variables. I also quoted `$COMMIT_LENGTH`.
- **BASE and HEAD:** I used `"${BASE:-}"` and `"${HEAD:-}"`, not the guarded `${VAR:+"$VAR"}` form the finding suggests. They follow `--since-commit` and `--branch`, so they always have to fill that slot. Upstream's `''` passed an empty argument when no value was set, and the quoted form does the same. With the guarded form, an empty BASE would make `--since-commit` take `--branch` as its value.
- **extra_args:** upstream split it on whitespace as an unquoted variable. It is now read into an array with `read -r -d '' -a ... || true`, only when it isn't empty, and passed as `"${extra_args[@]}"`. Words still split the same way, but nothing is evaluated or glob-expanded.

One behaviour change: when extra_args is empty, upstream passed one trailing empty argument to trufflehog and the new code passes none. That should be harmless, but I haven't checked it.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Rewrote action.yml cleanly. The first attempt, using targeted edits, left the file still garbled. Changes: (1) Fixed the broken '::error::' line and put back upstream's 'exit 1 / fi / else' structure, so the event-based BASE/HEAD selection only runs when no base or head input is given. (2) Deleted the leftover docker run that expanded ${BASE:-''}, ${HEAD:-''} and ${ARGS:-''} unquoted. The one docker run left quotes "${BASE:-}" and "${HEAD:-}". It passes extra_args as an array built with 'read -r -d "" -a', which splits on whitespace the same way upstream's unquoted $ARGS did. One small difference from upstream: when extra_args is empty, no empty argument is passed (upstream passed ''). Every ${{ }} expression is now in env:.

