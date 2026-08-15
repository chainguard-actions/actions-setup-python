<!-- markdownlint-disable -->

# Hardening Report: actions--setup-python/v6.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-python/v6.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple workflow run: blocks directly interpolate ${{ }} expressions, violating rule (a). Examples include: `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'` (steps output injected into shell command), `EXECUTABLE=${{ matrix.graalpy }}` (matrix value injected into shell variable assignment), `run: ${{ steps.setup-python.outputs.python-path }} --version` (step output used as the entire command), `run: pipx run --python '${{ steps.cp313.outputs.python-path }}'` (step output injected into shell), `pipenv install --python ${{ matrix.python-version }}` (matrix value injected into shell), and `run: ${{ startsWith(steps.setup-python.outputs.python-version, '3.14.') }}` (expression used as entire command). All of these allow an attacker-controlled value to be interpreted by the shell before quoting can protect it.

Locations:

- `.github/workflows/e2e-tests.yml:56`
- `.github/workflows/test-graalpy.yml:38`
- `.github/workflows/test-graalpy.yml:57`
- `.github/workflows/test-graalpy.yml:84`
- `.github/workflows/test-graalpy.yml:113`
- `.github/workflows/test-pypy.yml:40`
- `.github/workflows/test-pypy.yml:59`
- `.github/workflows/test-pypy.yml:113`
- `.github/workflows/test-pypy.yml:132`
- `.github/workflows/test-pypy.yml:165`
- `.github/workflows/test-pypy.yml:184`
- `.github/workflows/test-python.yml:38`
- `.github/workflows/test-python.yml:55`
- `.github/workflows/test-python.yml:80`
- `.github/workflows/test-python.yml:107`
- `.github/workflows/test-python.yml:134`
- `.github/workflows/test-python.yml:163`
- `.github/workflows/test-python.yml:193`
- `.github/workflows/test-python.yml:222`
- `.github/workflows/test-python.yml:253`
- `.github/workflows/test-python.yml:280`
- `.github/workflows/test-python.yml:310`
- `.github/workflows/test-python.yml:336`
- `.github/workflows/test-python.yml:362`
- `.github/workflows/test-python-freethreaded.yml:38`
- `.github/workflows/test-python-freethreaded.yml:55`
- `.github/workflows/test-python-freethreaded.yml:80`
- `.github/workflows/test-python-freethreaded.yml:107`
- `.github/workflows/test-python-freethreaded.yml:134`
- `.github/workflows/test-python-freethreaded.yml:163`
- `.github/workflows/test-python-freethreaded.yml:193`
- `.github/workflows/test-python-freethreaded.yml:222`
- `.github/workflows/test-python-freethreaded.yml:253`
- `.github/workflows/test-python-freethreaded.yml:280`
- `.github/workflows/test-python-freethreaded.yml:310`
- `.github/workflows/test-python-freethreaded.yml:336`
- `.github/workflows/e2e-cache.yml:96`
- `.github/workflows/e2e-cache.yml:99`
- `.github/workflows/e2e-cache.yml:196`
- `.github/workflows/e2e-cache.yml:199`

### unpinned-uses (severity: high)

All uses: references in workflow files use mutable tags or branch names instead of full 40-character SHA commit pins. Failing references include: `actions/checkout@v6` (used in nearly every workflow), `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`, `actions/reusable-workflows/.github/workflows/check-dist.yml@main`, `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`, `actions/reusable-workflows/.github/workflows/licensed.yml@main`, `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`, `actions/publish-action@v0.4.0`, `actions/publish-immutable-action@v0.0.4`. None of these are pinned to a full SHA digest.

Locations:

- `.github/workflows/basic-validation.yml:13`
- `.github/workflows/check-dist.yml:13`
- `.github/workflows/codeql-analysis.yml:11`
- `.github/workflows/e2e-cache-freethreaded.yml:20`
- `.github/workflows/e2e-cache.yml:20`
- `.github/workflows/e2e-tests.yml:26`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release-new-action-version.yml:21`
- `.github/workflows/test-graalpy.yml:37`
- `.github/workflows/test-graalpy.yml:74`
- `.github/workflows/test-graalpy.yml:100`
- `.github/workflows/test-pypy.yml:38`
- `.github/workflows/test-pypy.yml:99`
- `.github/workflows/test-pypy.yml:155`
- `.github/workflows/test-pypy.yml:175`
- `.github/workflows/test-pypy.yml:207`
- `.github/workflows/test-python-freethreaded.yml:37`
- `.github/workflows/test-python.yml:37`
- `.github/workflows/update-config-files.yml:10`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. This means the GITHUB_TOKEN is granted default (potentially write) permissions. Affected files: e2e-tests.yml, test-graalpy.yml, test-pypy.yml, test-python.yml, test-python-freethreaded.yml.

Locations:

- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/test-graalpy.yml:1`
- `.github/workflows/test-pypy.yml:1`
- `.github/workflows/test-python.yml:1`
- `.github/workflows/test-python-freethreaded.yml:1`

### unsafe-shell (severity: high)

Two run: steps in e2e-cache.yml pipe remote content directly to a Python interpreter: `run: curl https://raw.githubusercontent.com/pypa/pipenv/master/get-pipenv.py | python`. This fetches and executes arbitrary remote code without any integrity verification, and the URL references a mutable branch (`master`) so the content can change at any time.

Locations:

- `.github/workflows/e2e-cache.yml:72`
- `.github/workflows/e2e-cache.yml:172`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, unsafe-shell, script-injection

**Notes:**

Fixed all four security findings across 14 workflow files:

1. unpinned-uses: Pinned all action references to full SHA commits - actions/checkout@v6 → SHA df4cb1c, reusable-workflows@main → SHA 4735e71, publish-action@v0.4.0 → SHA 23f4c6f, publish-immutable-action@v0.0.4 → SHA 4bc8754.

2. missing-permissions: Added 'permissions: contents: read' top-level block to e2e-tests.yml, test-graalpy.yml, test-pypy.yml, test-python.yml, and test-python-freethreaded.yml.

3. unsafe-shell: Fixed both curl|python patterns in e2e-cache.yml by downloading the script to /tmp/get-pipenv.py first, then executing it separately.

4. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks. Fixed: check-python-path.sh with python-path output, EXECUTABLE=${{ matrix.graalpy/pypy }}, python-path output used as command, pipx run --python with step output, pipenv install --python with matrix value, startsWith() expression used as entire command (replaced with proper bash version check), and echo ${{ matrix.python }} > file patterns.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed 10 script injection instances by moving ${{ matrix.python }} from run: shell commands into env: blocks as MATRIX_PYTHON, then referencing $MATRIX_PYTHON in the shell scripts. The echo commands that embed the value in quoted TOML/INI strings use shell quoting (single-quote/double-quote switching) to safely expand the env var. Added top-level permissions blocks to 5 workflow files: basic-validation.yml (contents: read), check-dist.yml (contents: read), codeql-analysis.yml (contents: read + security-events: write), licensed.yml (contents: read), update-config-files.yml (contents: write).

