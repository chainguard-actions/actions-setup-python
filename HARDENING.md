<!-- markdownlint-disable -->

# Hardening Report: actions--setup-python/v6.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-python/v6.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks.

Failing references:
- basic-validation.yml: `uses: actions/reusable-workflows/.github/workflows/basic-validation.yml@main`
- check-dist.yml: `uses: actions/reusable-workflows/.github/workflows/check-dist.yml@main`
- codeql-analysis.yml: `uses: actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`
- licensed.yml: `uses: actions/reusable-workflows/.github/workflows/licensed.yml@main`
- update-config-files.yml: `uses: actions/reusable-workflows/.github/workflows/update-config-files.yml@main`
- publish-immutable-actions.yml: `uses: actions/checkout@v5`, `uses: actions/publish-immutable-action@v0.0.4`
- release-new-action-version.yml: `uses: actions/publish-action@v0.4.0`
- e2e-cache-freethreaded.yml: `uses: actions/checkout@v5`
- e2e-cache.yml: `uses: actions/checkout@v5`
- e2e-tests.yml: `uses: actions/checkout@v5`
- test-graalpy.yml: `uses: actions/checkout@v5`
- test-pypy.yml: `uses: actions/checkout@v5`
- test-python-freethreaded.yml: `uses: actions/checkout@v5`
- test-python.yml: `uses: actions/checkout@v5`

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/update-config-files.yml:11`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/e2e-cache-freethreaded.yml:22`
- `.github/workflows/e2e-cache.yml:22`
- `.github/workflows/e2e-tests.yml:22`
- `.github/workflows/test-graalpy.yml:27`
- `.github/workflows/test-pypy.yml:33`
- `.github/workflows/test-python-freethreaded.yml:33`
- `.github/workflows/test-python.yml:33`

### missing-permissions (severity: medium)

Multiple workflow files have no top-level `permissions:` key and no job-level `permissions:` keys, meaning jobs run with the default (potentially broad) GITHUB_TOKEN permissions.

Affected files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-graalpy.yml, test-pypy.yml, test-python-freethreaded.yml, test-python.yml, update-config-files.yml.

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

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions (sub-rule a), allowing an attacker to inject arbitrary shell commands. The affected contexts include `matrix.*` values (which can be influenced via workflow_dispatch or pull_request triggers) and `steps.*.outputs.*` values.

Key violations:

**test-python.yml:**
- `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'` (multiple jobs)
- `run: echo ${{ matrix.python }} > .python-version` (multiple jobs)
- `run: ${{ steps.setup-python.outputs.python-path }} --version`
- `run: ${{ steps.setup-python.outputs.python-path }} -c '...'`
- `run: ${{ startsWith(steps.setup-python.outputs.python-version, '3.14.') }}` (multiple)
- `echo "python ${{ matrix.python }}" > .tool-versions` inside run block
- `echo '[project]\n  requires-python = "${{ matrix.python }}"' > pyproject.toml` inside run block

**test-python-freethreaded.yml:**
- `run: ${{ steps.setup-python.outputs.python-path }} -VVV`
- `run: echo ${{ matrix.python }} > .python-version` (multiple jobs)
- `echo "python ${{ matrix.python }}" > .tool-versions` inside run block
- `run: ${{ steps.setup-python.outputs.python-path }} --version` (multiple)

**test-graalpy.yml:**
- `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'`
- `EXECUTABLE=${{ matrix.graalpy }}` inside run block
- `run: ${{ steps.setup-python.outputs.python-path }} --version`
- `run: ${{ steps.setup-python.outputs.python-path }} -c '...'`
- `EXECUTABLE='${{ steps.graalpy.outputs.python-version }}'` inside run block

**test-pypy.yml:**
- `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'` (multiple jobs)
- `EXECUTABLE=${{ matrix.pypy }}` inside run block (multiple jobs)
- `run: ${{ steps.setup-python.outputs.python-path }} --version`
- `run: ${{ steps.setup-python.outputs.python-path }} -c '...'`

**e2e-tests.yml:**
- `run: pipx run --python '${{ steps.cp313.outputs.python-path }}' nox --version`

**e2e-cache.yml:**
- `if ("${{ matrix.python-version }}" -Match "pypy")` inside run blocks (two jobs)

Locations:

- `.github/workflows/test-python.yml:42`
- `.github/workflows/test-python.yml:68`
- `.github/workflows/test-python.yml:88`
- `.github/workflows/test-python.yml:113`
- `.github/workflows/test-python.yml:133`
- `.github/workflows/test-python-freethreaded.yml:42`
- `.github/workflows/test-python-freethreaded.yml:44`
- `.github/workflows/test-python-freethreaded.yml:68`
- `.github/workflows/test-python-freethreaded.yml:88`
- `.github/workflows/test-graalpy.yml:38`
- `.github/workflows/test-graalpy.yml:48`
- `.github/workflows/test-graalpy.yml:75`
- `.github/workflows/test-graalpy.yml:77`
- `.github/workflows/test-graalpy.yml:104`
- `.github/workflows/test-pypy.yml:46`
- `.github/workflows/test-pypy.yml:56`
- `.github/workflows/test-pypy.yml:83`
- `.github/workflows/test-pypy.yml:100`
- `.github/workflows/e2e-tests.yml:57`
- `.github/workflows/e2e-cache.yml:88`
- `.github/workflows/e2e-cache.yml:214`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 14 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs: actions/reusable-workflows@main→4735e71..., actions/checkout@v5→93cb6ef..., actions/publish-immutable-action@v0.0.4→4bc8754..., actions/publish-action@v0.4.0→23f4c6f...

2. missing-permissions: Added 'permissions: contents: read' top-level block to basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-graalpy.yml, test-pypy.yml, test-python-freethreaded.yml, test-python.yml, update-config-files.yml.

3. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks. Key fixes: python-path outputs → PYTHON_PATH env var; python-version outputs → PYTHON_VERSION env var; matrix values (python, pypy, graalpy) → env vars; startsWith() expressions replaced with bash [[ ]] conditionals; PowerShell -Match expressions use $env:PYTHON_VERSION; e2e-cache.yml pipenv install blocks use $env:PYTHON_VERSION in pwsh.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed all 4 occurrences of `curl https://raw.githubusercontent.com/pypa/pipenv/master/get-pipenv.py | python` across two workflow files:
- `.github/workflows/e2e-cache-freethreaded.yml` lines 71 and 130 (jobs: python-pipenv-dependencies-caching, python-pipenv-dependencies-caching-path)
- `.github/workflows/e2e-cache.yml` lines 99 and 195 (jobs: python-pipenv-dependencies-caching, python-pipenv-dependencies-caching-path)

Each unsafe pipe was replaced with a three-step safe pattern:
1. `curl -sSfL <url> -o get-pipenv.py` — download to a local file
2. `python get-pipenv.py` — execute the local file
3. `rm get-pipenv.py` — clean up the downloaded file

This prevents arbitrary code execution from a compromised remote URL or MITM network interception.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in 4 locations across 2 workflow files:
1. hardened/action/.github/workflows/test-graalpy.yml (setup-graalpy job, line ~74): Changed `${EXECUTABLE} --version` to `"${EXECUTABLE}" --version`
2. hardened/action/.github/workflows/test-graalpy.yml (check-latest job, line ~120): Changed `${EXECUTABLE} --version` to `"${EXECUTABLE}" --version`
3. hardened/action/.github/workflows/test-pypy.yml (setup-pypy job, line ~85): Changed `${EXECUTABLE} --version` to `"${EXECUTABLE}" --version`
4. hardened/action/.github/workflows/test-pypy.yml (check-non-eol job, line ~141): Changed `${EXECUTABLE} --version` to `"${EXECUTABLE}" --version`

In all cases, EXECUTABLE was derived from workflow-controllable matrix values (matrix.graalpy or matrix.pypy) via env vars. Quoting the variable prevents shell metacharacter injection while preserving the intended behavior.

