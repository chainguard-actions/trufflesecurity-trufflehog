<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.99.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.99.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Composite step run block in action.yml. (a) GitHub expressions are put straight into the shell script: `if [ "${{ github.event_name }}" == "push" ]` (line 78, also lines 90 and 93), `HEAD=${{ github.event.after }}` (line 84), `if [ ${{ github.event.before }} == ...` (line 85), `BASE=${{ github.event.before }}` (line 88), `BASE=${{github.event.pull_request.base.sha}}` (line 94) and `HEAD=${{github.event.pull_request.head.sha}}` (line 95). These should be passed in through env: variables and read as quoted shell variables. (b) Env vars that hold untrusted values (inputs.base, inputs.head, inputs.extra_args) are expanded without quotes as docker arguments: `${BASE:-''}` (line 105), `${HEAD:-''}` (line 107) and `${ARGS:-''}` (line 112). `$COMMIT_LENGTH`, which comes from github.event.commits, is also unquoted on line 80. Use quoted or guarded forms such as `${BASE:+"$BASE"}`. ARGS should be parsed into an array safely, not word-split.

Locations:

- `action.yml:78`
- `action.yml:80`
- `action.yml:84`
- `action.yml:85`
- `action.yml:88`
- `action.yml:90`
- `action.yml:93`
- `action.yml:94`
- `action.yml:95`
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

Moved github.event_name, event.after, event.before and pull_request.base.sha/head.sha out of the script and into env: variables (EVENT_NAME, EVENT_AFTER, EVENT_BEFORE, PR_BASE_SHA, PR_HEAD_SHA). The script now reads them as quoted shell variables. $COMMIT_LENGTH is quoted. BASE and HEAD reach docker as "${BASE}" and "${HEAD}". When they are empty this still passes one empty argument, as upstream's ${X:-''} did. ARGS is split on whitespace into an array with 'read -r -d "" -a' (the same split as upstream's unquoted env var, minus globbing). An empty ARGS still passes one empty argument, as upstream did.

