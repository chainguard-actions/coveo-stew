<!-- markdownlint-disable -->

# Hardening Report: coveo--stew/5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **coveo--stew/5.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action references actions by mutable tags instead of 40-character commit SHAs: 'uses: actions/checkout@v7' and 'uses: actions/setup-python@v7'.

Locations:

- `action.yml:53`
- `action.yml:54`

### script-injection (severity: high)

Rule (b): in the 'Upgrade python tools and install pipx' step, the workflow-controlled env var PYTHON_EXEC (from inputs.python-exec) is expanded without quotes: '$PYTHON_EXEC -m pip install --upgrade pip wheel setuptools pipx --user --disable-pip-version-check'.

Locations:

- `action.yml:67`

### script-injection (severity: high)

Rule (b): in the 'poetry and stew' step, PYTHON_EXEC (from inputs.python-exec) is expanded without quotes on three lines, e.g. '$PYTHON_EXEC -m pipx install "poetry$POETRY_VERSION"', '$PYTHON_EXEC -m pipx inject poetry poetry-plugin-export' and '$PYTHON_EXEC -m pipx install "coveo-stew$COVEO_STEW_VERSION"'.

Locations:

- `action.yml:77`
- `action.yml:78`
- `action.yml:79`

### script-injection (severity: high)

Rule (a): in the 'Run stew ci' step, an expression is placed straight into the run script: 'run: stew ci ${{ env.STEW_CI_ARGS }}'. STEW_CI_ARGS comes from inputs.project-name, so an attacker who controls that input can run shell commands.

Locations:

- `action.yml:104`

### github-env-injection (severity: high)

In the 'Target specific project by name' step, PROJECT_NAME (from inputs.project-name) is written to $GITHUB_ENV without first stripping newlines: 'echo "STEW_CI_ARGS=$PROJECT_NAME --exact-match" >> $GITHUB_ENV'. A newline in the value would let it set any environment variable.

Locations:

- `action.yml:91`

### github-env-injection (severity: high)

In the 'Target the project at the root of the repository' step, PROJECT_PATH (from inputs.project-name) is written to $GITHUB_ENV without first stripping newlines: 'echo "STEW_CI_ARGS=$PROJECT_PATH" >> $GITHUB_ENV'.

Locations:

- `action.yml:98`

### github-env-injection (severity: high)

In the 'Prepare pipx path' step, the inherited process env var HOME is written to $GITHUB_PATH without the printf '%s' | tr -d '\n\r' sanitization step: 'echo "$HOME/.local/bin" >> $GITHUB_PATH'. A composite action inherits its environment from the calling workflow, so HOME counts as workflow-controlled under this check.

Locations:

- `action.yml:61`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Pinned actions/checkout and actions/setup-python to their v7 commit SHAs. Double-quoted every $PYTHON_EXEC use. In 'Run stew ci', the inline ${{ env.STEW_CI_ARGS }} is replaced by reading the STEW_CI_ARGS env var, which the job already has, and splitting it into a bash array with the quote-aware xargs pattern. This gives the same tokens as upstream's inline expression. Newlines are now stripped (printf | tr -d '\n\r') before the project name or path is written to GITHUB_ENV, and before HOME is written to GITHUB_PATH. The changes were not run.

