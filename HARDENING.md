<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub expressions are pasted straight into the run: script in action.yml. Examples: `if [ "${{ github.event_name }}" == "push" ]; then` (line 78), `HEAD=${{ github.event.after }}` (line 84), `if [ ${{ github.event.before }} == "0000..." ]` (line 85), `BASE=${{ github.event.before }}` (line 88), the event_name comparisons (lines 90, 93), and `BASE=${{github.event.pull_request.base.sha}}` / `HEAD=${{github.event.pull_request.head.sha}}` (lines 94-95). Fix: pass these values in through env: variables (or use $GITHUB_EVENT_NAME) and double-quote them.

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

Sub-rule (b): env vars that come from inputs are expanded without quotes in the docker run command: `${BASE:-''}` (line 105), `${HEAD:-''}` (line 107) and `${ARGS:-''}` (line 111). BASE, HEAD and ARGS come from inputs.base, inputs.head and inputs.extra_args. They are also reassigned from event data, and `$COMMIT_LENGTH` is unquoted on line 80. Without quotes the shell splits these values into words and expands glob characters. Fix: quote them, or use the guarded form `${VAR:+"$VAR"}` for optional positional arguments. ARGS should be parsed safely, for example into a bash array.

Locations:

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

Changes are in action.yml only; nothing was run or tested. github.event_name, event.after, event.before and pull_request.base.sha/head.sha are now passed in through env: as EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA and PR_HEAD_SHA, and are always double-quoted where used. COMMIT_LENGTH is quoted too. In the docker run command, ${BASE:-''} and ${HEAD:-''} became "${BASE}" and "${HEAD}". An empty value still gives one empty argument, as upstream did. One small difference: a non-empty value is now always a single argument, where upstream would have split it on spaces. That only matters for a value containing spaces, and a SHA or branch name does not. extra_args is a list, so it is split on whitespace into a bash array with read -r -d '' -a. That matches upstream's unquoted $ARGS, except that globs are no longer expanded, and it runs nothing inside the value. When extra_args is empty it still passes one empty argument, as upstream did.

