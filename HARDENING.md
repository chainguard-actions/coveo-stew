<!-- markdownlint-disable -->

# Hardening Report: coveo--stew/4.1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **coveo--stew/4.1.6** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references: `actions/checkout@v6` (line 60) and `actions/setup-python@v6` (line 61).

Locations:

- `action.yml:60`
- `action.yml:61`

### script-injection (severity: high)

Rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. At line 111: `run: stew ci ${{ env.STEW_CI_ARGS }}` — the `env.STEW_CI_ARGS` value is set from attacker-controlled inputs (inputs.project-name) in earlier steps and is injected directly into the shell command without quoting or sanitization, enabling command injection.

Locations:

- `action.yml:111`

### script-injection (severity: high)

Rule (b): Multiple `run:` blocks expand env vars sourced from workflow inputs without double-quoting, allowing shell metacharacter injection. (1) Line 74: `run: $PYTHON_EXEC -m pip install ...` — `$PYTHON_EXEC` is unquoted and sourced from `inputs.python-exec`. (2) Lines 84–86: `$PYTHON_EXEC`, `$POETRY_VERSION`, and `$COVEO_STEW_VERSION` are all unquoted inside the run block and sourced from `inputs.python-exec`, `inputs.poetry-version`, and `inputs.coveo-stew-version` respectively. An attacker-controlled input value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) would be executed by the shell.

Locations:

- `action.yml:74`
- `action.yml:84`
- `action.yml:85`
- `action.yml:86`

### github-env-injection (severity: high)

Two `run:` steps write values derived from the untrusted input `inputs.project-name` to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). (1) Line 98: `echo "STEW_CI_ARGS=$PROJECT_NAME --exact-match" >> $GITHUB_ENV` — `$PROJECT_NAME` is set from `inputs.project-name`. (2) Line 105: `echo "STEW_CI_ARGS=$PROJECT_PATH" >> $GITHUB_ENV` — `$PROJECT_PATH` is set from `inputs.project-name`. A newline character in the input value can inject arbitrary environment variables into `$GITHUB_ENV`, which are then available to all subsequent steps.

Locations:

- `action.yml:98`
- `action.yml:105`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all four findings in action.yml:
1. unpinned-uses: Pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and actions/setup-python@v6 to SHA ece7cb06caefa5fff74198d8649806c4678c61a1.
2. script-injection (a): Moved env.STEW_CI_ARGS from inline ${{ }} in run: to an env: block, referenced as $STEW_CI_ARGS in the shell.
3. script-injection (b): Added double-quotes around $PYTHON_EXEC in both run blocks; $POETRY_VERSION and $COVEO_STEW_VERSION are already inside double-quoted strings.
4. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing PROJECT_NAME and PROJECT_PATH to $GITHUB_ENV in both affected steps.

### Iteration 2

**Fixes applied:** invalid-yaml, script-injection

**Notes:**

Fixed two issues in hardened/action/action.yml: (1) Converted the single-line `run: "$PYTHON_EXEC" -m pip install ...` to a block scalar (`run: |`) to fix the YAML parse error at line 62 — YAML was treating the quoted `"$PYTHON_EXEC"` as a complete scalar and rejecting the trailing text. (2) Quoted the `$STEW_CI_ARGS` variable in the 'Run stew ci' step (`stew ci "$STEW_CI_ARGS"`) to prevent word-splitting and glob expansion on the attacker-controlled input.

