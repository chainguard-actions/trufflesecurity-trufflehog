<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.100.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.100.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the composite step's run: block puts GitHub expressions straight into the shell script. Offending lines: `if [ "${{ github.event_name }}" == "push" ]; then` (line 78), `HEAD=${{ github.event.after }}` (84), `if [ ${{ github.event.before }} == "0000..." ]` (85), `BASE=${{ github.event.before }}` (88), the `${{ github.event_name }}` comparisons for workflow_dispatch/schedule/pull_request (90, 93), `BASE=${{github.event.pull_request.base.sha}}` (94) and `HEAD=${{github.event.pull_request.head.sha}}` (95). Fix: map github.event_name, github.event.before/after and pull_request.base/head.sha into env: variables (or read the built-in $GITHUB_EVENT_NAME), then reference them double-quoted.

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

Rule (b): in the docker run command, env vars holding workflow-controlled input are expanded without quotes: `${BASE:-''}` (line 105, from inputs.base), `${HEAD:-''}` (line 107, from inputs.head) and `${ARGS:-''}` (line 112, from inputs.extra_args). None of these use the guarded `${VAR:+"$VAR"}` form, and none is an array assignment with a shellcheck directive. Fix: use `${BASE:+"$BASE"}` and `${HEAD:+"$HEAD"}`. For ARGS, put it in an array on its own line, e.g. `# shellcheck disable=SC2206` then `ARGS_ARR=($ARGS)`, and pass `"${ARGS_ARR[@]}"`.

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

Moved github.event_name, event.before/after and pull_request base/head sha into the step's env: block and replaced the inline ${{ }} expressions in run: with double-quoted variables. In the docker run command, BASE and HEAD are now passed as "${BASE:-}" and "${HEAD:-}". Upstream's ${BASE:-''} gave one empty argument when the value was unset. Keeping that empty argument stops --since-commit and --branch from taking the next flag as their value, which is why I didn't use the suggested ${VAR:+"$VAR"} form. extra_args is split with an unquoted array assignment marked shellcheck SC2206, so it splits and globs on whitespace the same way upstream did. When extra_args is empty, a single empty argument is still passed, as before. The result is passed as "${ARGS_ARR[@]}". I also quoted $COMMIT_LENGTH in its comparison.

