<!-- markdownlint-disable -->

# Hardening Report: trufflesecurity--trufflehog/v3.100.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **trufflesecurity--trufflehog/v3.100.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Composite step run: block in action.yml breaks both script-injection sub-rules. (a) GitHub expressions are put straight into the shell script: `if [ "${{ github.event_name }}" == "push" ]` (lines 78, 90, 93), `HEAD=${{ github.event.after }}` (line 84), `if [ ${{ github.event.before }} == ...` (line 85), `BASE=${{ github.event.before }}` (line 88), `BASE=${{github.event.pull_request.base.sha}}` (line 94) and `HEAD=${{github.event.pull_request.head.sha}}` (line 95). Pass these through env: variables and use them double-quoted. (b) Shell variables holding workflow-controlled data are expanded without quotes in the docker run command: `${BASE:-''}` (line 105, from inputs.base), `${HEAD:-''}` (line 107, from inputs.head) and `${ARGS:-''}` (line 111, from inputs.extra_args). `$COMMIT_LENGTH` is also unquoted in `if [ $COMMIT_LENGTH == "0" ]` (line 80). It holds jq output computed from event data. Quote these expansions, or use the guarded form `${VAR:+"$VAR"}` for optional positional arguments.

Locations:

- `action.yml:78`
- `action.yml:84`
- `action.yml:85`
- `action.yml:88`
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

I moved every ${{ }} expression out of the run: script in action.yml and into the step's env: block. That covers event_name, event.after, event.before, and the pull request base and head SHAs. The script now reads them as double-quoted variables. $COMMIT_LENGTH is now quoted. BASE and HEAD are passed to docker run as "${BASE}" and "${HEAD}". The old ${BASE:-''} and ${HEAD:-''} passed one empty argument when the value was empty, and the quoted form does the same. extra_args is a whitespace-separated list, so it is not passed as one quoted word. Instead it goes into an array assigned unquoted, which splits and globs it the same way upstream's unquoted ${ARGS:-''} did. When it is empty, the array holds one empty string, matching upstream. Nothing was run or tested.

