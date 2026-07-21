<!-- markdownlint-disable -->

# Hardening Report: actions--setup-python/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-python/v7.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple workflow files contain ${{ }} expressions interpolated directly inside run: shell command strings (sub-rule a). This allows expression values to be interpreted as shell commands before the shell ever sees them.

Examples of offending lines:
- e2e-tests.yml: `run: pipx run --python '${{ steps.cp314.outputs.python-path }}' nox --version`
- test-graalpy.yml: `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'`, `run: ${{ steps.setup-python.outputs.python-path }} --version`, `run: ${{ steps.setup-python.outputs.python-path }} -c ...`, `EXECUTABLE=${{ matrix.graalpy }}` inside a run block, `EXECUTABLE='${{ steps.graalpy.outputs.python-version }}'`
- test-pypy.yml: `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'`, `run: ${{ steps.setup-python.outputs.python-path }} --version`, `run: ${{ steps.setup-python.outputs.python-path }} -c ...`, `EXECUTABLE=${{ matrix.pypy }}`
- test-python.yml: `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'`, `run: echo ${{ matrix.python }} > .python-version`, `run: ${{ steps.setup-python.outputs.python-path }} -VVV`, `run: ${{ startsWith(steps.setup-python.outputs.python-version, '3.15.') }}`, and `${{ matrix.python }}` interpolated inside PowerShell run blocks
- test-python-freethreaded.yml: same patterns as test-python.yml across many jobs

Locations:

- `.github/workflows/e2e-tests.yml:67`
- `.github/workflows/test-graalpy.yml:37`
- `.github/workflows/test-graalpy.yml:52`
- `.github/workflows/test-graalpy.yml:80`
- `.github/workflows/test-graalpy.yml:83`
- `.github/workflows/test-graalpy.yml:107`
- `.github/workflows/test-pypy.yml:41`
- `.github/workflows/test-pypy.yml:52`
- `.github/workflows/test-pypy.yml:88`
- `.github/workflows/test-pypy.yml:99`
- `.github/workflows/test-pypy.yml:118`
- `.github/workflows/test-pypy.yml:119`
- `.github/workflows/test-python.yml:40`
- `.github/workflows/test-python.yml:68`
- `.github/workflows/test-python.yml:75`
- `.github/workflows/test-python.yml:101`
- `.github/workflows/test-python.yml:108`
- `.github/workflows/test-python-freethreaded.yml:36`
- `.github/workflows/test-python-freethreaded.yml:43`
- `.github/workflows/test-python-freethreaded.yml:65`
- `.github/workflows/test-python-freethreaded.yml:73`
- `.github/workflows/test-python-freethreaded.yml:95`

### unpinned-uses (severity: high)

Several workflow files reference reusable workflows or actions using mutable branch/tag refs instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced repository is compromised.

Offending references:
- basic-validation.yml: `uses: actions/reusable-workflows/.github/workflows/basic-validation.yml@main`
- check-dist.yml: `uses: actions/reusable-workflows/.github/workflows/check-dist.yml@main`
- codeql-analysis.yml: `uses: actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`
- licensed.yml: `uses: actions/reusable-workflows/.github/workflows/licensed.yml@main`
- release-new-action-version.yml: `uses: actions/publish-action@v0.4.0`
- update-config-files.yml: `uses: actions/reusable-workflows/.github/workflows/update-config-files.yml@main`

Locations:

- `.github/workflows/basic-validation.yml:13`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:11`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/update-config-files.yml:11`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (which may be broad), violating the principle of least privilege.

Affected files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-graalpy.yml, test-pypy.yml, test-python-freethreaded.yml, test-python.yml, update-config-files.yml

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-graalpy.yml:1`
- `.github/workflows/test-pypy.yml:1`
- `.github/workflows/test-python-freethreaded.yml:1`
- `.github/workflows/test-python.yml:1`
- `.github/workflows/update-config-files.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across the workflow files:

1. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks in e2e-tests.yml, test-graalpy.yml, test-pypy.yml, test-python.yml, and test-python-freethreaded.yml. The startsWith() expressions used as shell commands were replaced with proper bash conditionals. Python path outputs used as executables are now referenced via $PYTHON_PATH env var. Matrix version values used in echo/heredoc commands are now passed via $PYTHON_VERSION env var.

2. unpinned-uses: Pinned all 6 mutable references to full SHA hashes: actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 (basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml) and actions/publish-action@v0.4.0 → @23f4c6f12633a2da8f44938b71fde9afec138fb4 (release-new-action-version.yml).

3. missing-permissions: Added permissions: {} to all 10 flagged workflow files. The release-new-action-version.yml already had permissions: contents: write which was preserved.

### Iteration 2

**Fixes applied:** unsafe-shell, script-injection

**Notes:**

Fixed both findings in two jobs (python-pipenv-dependencies-caching and python-pipenv-dependencies-caching-path) in .github/workflows/e2e-cache.yml:

1. unsafe-shell (lines 87, 195): Replaced `curl https://... | python` with a safe pattern: download to a temp file with `curl -sSL -o get-pipenv.py`, execute separately with `python get-pipenv.py`, then remove the file. This avoids executing unverified remote content.

2. script-injection (lines 98, 101, 208, 211): Moved `${{ matrix.python-version }}` out of PowerShell `run:` blocks into an `env:` block as `PYTHON_VERSION`. Updated PowerShell scripts to reference `$env:PYTHON_VERSION` instead of the direct template expression, preventing shell command injection via attacker-controlled matrix values.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted ${EXECUTABLE} --version shell expansions in .github/workflows/test-graalpy.yml (2 locations: setup-graalpy job line ~72 and check-latest job line ~119) and .github/workflows/test-pypy.yml (5 locations: setup-pypy job line ~62, check-non-eol job line ~107, check-latest job, and two in setup-pypy-multiple-versions job). Changed `${EXECUTABLE} --version` to `"${EXECUTABLE}" --version` in all cases to prevent shell metacharacter injection from workflow-controllable matrix values.

