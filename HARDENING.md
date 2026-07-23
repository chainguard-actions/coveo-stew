<!-- markdownlint-disable -->

# Hardening Report: coveo--stew/4.1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **coveo--stew/4.1.6** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple `run:` blocks directly interpolate `${{ }}` expressions inside shell commands, allowing expression values to be parsed as shell code before the shell ever sees them.

1. `action.yml`: `run: stew ci ${{ env.STEW_CI_ARGS }}` — the env context value is interpolated directly into the shell command.
2. `.github/workflows/actions/build/action.yml`: `run: echo "minimum-version=$(${{ github.action_path }}/get-minimum-version.sh)"` — `github.action_path` interpolated inside command substitution.
3. `.github/workflows/actions/build/action.yml`: `RELEASE="$(pypi next-version coveo-stew --minimum-version ${{ steps.get-minimum-version.outputs.minimum-version }})"` — step output interpolated directly.
4. `.github/workflows/actions/build/action.yml`: `if [[ ${{ inputs.pre-release }} == true ]]` — input interpolated into shell conditional.
5. `.github/workflows/actions/build/action.yml`: `echo "version=${{ steps.get-versions.outputs.prerelease }}" >> $GITHUB_OUTPUT` — step output interpolated in echo.
6. `.github/workflows/actions/build/action.yml`: `poetry version ${{ steps.get-next-version.outputs.version }}` — step output interpolated as CLI argument.
7. `.github/workflows/actions/post-publish/action.yml`: `TAG_NAME=${{ inputs.next-version }}` — input interpolated as shell assignment.
8. `.github/workflows/actions/post-publish/action.yml`: `git tag ... $TAG_NAME ${{ github.sha }}` — github context interpolated as CLI argument.
9. `.github/workflows/actions/post-publish/action.yml`: `if [[ ${{ inputs.dry-run }} == false ]]` — input interpolated into shell conditional.
10. `.github/workflows/actions/setup-python-and-tools/action.yml`: `pipx install "poetry${{ inputs.poetry-version }}"` — input interpolated inside quoted string in shell command.

Locations:

- `action.yml:82`
- `.github/workflows/actions/build/action.yml:21`
- `.github/workflows/actions/build/action.yml:27`
- `.github/workflows/actions/build/action.yml:28`
- `.github/workflows/actions/build/action.yml:36`
- `.github/workflows/actions/build/action.yml:38`
- `.github/workflows/actions/build/action.yml:44`
- `.github/workflows/actions/build/action.yml:45`
- `.github/workflows/actions/post-publish/action.yml:18`
- `.github/workflows/actions/post-publish/action.yml:23`
- `.github/workflows/actions/post-publish/action.yml:27`
- `.github/workflows/actions/setup-python-and-tools/action.yml:27`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_ENV` or `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. `action.yml` — Step 'Target specific project by name': `echo "STEW_CI_ARGS=$PROJECT_NAME --exact-match" >> $GITHUB_ENV` where `PROJECT_NAME` comes from `inputs.project-name`. An attacker-controlled newline in the input can inject arbitrary environment variables.
2. `action.yml` — Step 'Target the project at the root of the repository': `echo "STEW_CI_ARGS=$PROJECT_PATH" >> $GITHUB_ENV` where `PROJECT_PATH` comes from `inputs.project-name`. Same injection risk.
3. `.github/workflows/actions/build/action.yml` — Step 'Compute the release and prerelease versions': `echo "release=$RELEASE" >> $GITHUB_OUTPUT` and `echo "prerelease=$PRERELEASE" >> $GITHUB_OUTPUT` where `$RELEASE`/`$PRERELEASE` are derived from `${{ steps.get-minimum-version.outputs.minimum-version }}` (a step output, which is workflow-controllable). No sanitization applied.
4. `.github/workflows/actions/build/action.yml` — Step 'Determine the version to publish': `echo "version=${{ steps.get-versions.outputs.prerelease }}" >> $GITHUB_OUTPUT` and `echo "version=${{ steps.get-versions.outputs.release }}" >> $GITHUB_OUTPUT` — step outputs written directly to GITHUB_OUTPUT without sanitization.
5. `.github/workflows/actions/post-publish/action.yml` — Step 'Tag repository': `echo "tag-name=$TAG_NAME" >> $GITHUB_OUTPUT` where `TAG_NAME=${{ inputs.next-version }}` — input written to GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:68`
- `action.yml:75`
- `.github/workflows/actions/build/action.yml:29`
- `.github/workflows/actions/build/action.yml:31`
- `.github/workflows/actions/build/action.yml:36`
- `.github/workflows/actions/build/action.yml:38`
- `.github/workflows/actions/post-publish/action.yml:19`

### unpinned-uses (severity: high)

Multiple `uses:` references use mutable version tags or branch names instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag or branch is moved or overwritten.

Failing references:
- `action.yml`: `uses: actions/checkout@v6` and `uses: actions/setup-python@v6`
- `.github/workflows/coveo-stew.yml`: `uses: actions/checkout@v6` (appears 3 times)
- `.github/workflows/dependency-review.yml`: `uses: coveo/public-actions/.github/workflows/dependency-review-v3.yml@main` (branch reference)
- `.github/workflows/actions/setup-python-and-tools/action.yml`: `uses: actions/setup-python@v6`

Locations:

- `action.yml:48`
- `action.yml:49`
- `.github/workflows/coveo-stew.yml:30`
- `.github/workflows/coveo-stew.yml:55`
- `.github/workflows/coveo-stew.yml:97`
- `.github/workflows/dependency-review.yml:11`
- `.github/workflows/actions/setup-python-and-tools/action.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding types across 6 files:

1. unpinned-uses: Pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@v6 to SHA ece7cb06caefa5fff74198d8649806c4678c61a1, and coveo/public-actions@main to SHA 5a3c9311bf6a65f17f02c21d72cccd1a0ad48a3a in action.yml, coveo-stew.yml (3 occurrences), dependency-review.yml, and setup-python-and-tools/action.yml.

2. script-injection: Moved all ${{ }} expressions from run: shell strings into env: blocks in action.yml (STEW_CI_ARGS), build/action.yml (ACTION_PATH, MINIMUM_VERSION, PRE_RELEASE, PRERELEASE_VERSION, RELEASE_VERSION, NEXT_VERSION), post-publish/action.yml (NEXT_VERSION, GIT_SHA, DRY_RUN), and setup-python-and-tools/action.yml (POETRY_VERSION).

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before all writes to $GITHUB_ENV and $GITHUB_OUTPUT in action.yml (PROJECT_NAME, PROJECT_PATH), build/action.yml (minimum-version, release, prerelease, version), and post-publish/action.yml (tag-name).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings in hardened/action/action.yml:
1. 'Upgrade python tools and install pipx' step (line 70): Quoted `$PYTHON_EXEC` → `"$PYTHON_EXEC"` to prevent shell metacharacter injection.
2. 'poetry and stew' step (lines 80-82): Quoted `$PYTHON_EXEC`, `$POETRY_VERSION`, and `$COVEO_STEW_VERSION` using double-quotes around each variable expansion.
3. 'Run stew ci' step (line 105): Replaced unquoted `stew ci $STEW_CI_ARGS` with a bash array approach (`IFS=' ' read -ra args <<< "$STEW_CI_ARGS"` then `stew ci "${args[@]}"`) so each argument token is properly quoted while still allowing the space-separated args to be passed as separate arguments to stew.

### Iteration 3

**Fixes applied:** invalid-yaml

**Notes:**

Fixed YAML parsing error at line 62 in action.yml. The `run:` value started with a quoted string `"$PYTHON_EXEC"` which YAML parsed as a complete quoted scalar and then rejected the trailing ` -m pip install --upgrade pip wheel setuptools pipx --user --disable-pip-version-check`. Converted the single-line `run:` to a block scalar using `run: |` so the entire command is treated as a literal string.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/action.yml (lines 78/80): In the 'poetry and stew' step, sanitized POETRY_VERSION and COVEO_STEW_VERSION by stripping backticks, dollar signs, and backslashes via `printf '%s' "$VAR" | tr -d '`$\\'` before interpolating into double-quoted pipx install commands.
2. hardened/action/.github/workflows/actions/setup-python-and-tools/action.yml (line 33): In the 'Install poetry' step, sanitized POETRY_VERSION the same way before using it in the pipx install command.
Both fixes prevent command substitution injection where attacker-controlled version strings like `$(malicious_command)` could be executed by bash inside double-quoted strings.

