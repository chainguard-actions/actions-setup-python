<!-- markdownlint-disable -->

# Hardening Report: actions--setup-python/v6.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-python/v6.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or branch names instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks. Unpinned refs found:
- basic-validation.yml: `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`
- check-dist.yml: `actions/reusable-workflows/.github/workflows/check-dist.yml@main`
- codeql-analysis.yml: `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`
- e2e-cache-freethreaded.yml: `actions/checkout@v6` (multiple)
- e2e-cache.yml: `actions/checkout@v6` (multiple)
- e2e-tests.yml: `actions/checkout@v6`
- licensed.yml: `actions/reusable-workflows/.github/workflows/licensed.yml@main`
- publish-immutable-actions.yml: `actions/checkout@v6`, `actions/publish-immutable-action@v0.0.4`
- release-new-action-version.yml: `actions/publish-action@v0.4.0`
- test-graalpy.yml: `actions/checkout@v6` (multiple)
- test-pypy.yml: `actions/checkout@v6` (multiple)
- test-python-freethreaded.yml: `actions/checkout@v6` (multiple)
- test-python.yml: `actions/checkout@v6` (multiple)
- update-config-files.yml: `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:15`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/e2e-cache-freethreaded.yml:22`
- `.github/workflows/e2e-cache.yml:22`
- `.github/workflows/e2e-tests.yml:30`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release-new-action-version.yml:19`
- `.github/workflows/test-graalpy.yml:37`
- `.github/workflows/test-pypy.yml:40`
- `.github/workflows/test-python-freethreaded.yml:37`
- `.github/workflows/test-python.yml:37`
- `.github/workflows/update-config-files.yml:10`

### permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions.

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

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions (rule a), allowing expression values to be parsed by the shell before quoting can occur. Affected patterns include:
- `run: pipx run --python '${{ steps.cp313.outputs.python-path }}' nox --version` (e2e-tests.yml)
- `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'` (test-graalpy.yml, test-pypy.yml, test-python-freethreaded.yml, test-python.yml)
- `run: ${{ steps.setup-python.outputs.python-path }} --version` (test-graalpy.yml, test-pypy.yml, test-python-freethreaded.yml, test-python.yml)
- `run: ${{ steps.setup-python.outputs.python-path }} -c '...'` (test-graalpy.yml, test-pypy.yml, test-python-freethreaded.yml, test-python.yml)
- `run: EXECUTABLE=${{ matrix.graalpy }}` inside a multiline run block (test-graalpy.yml)
- `run: echo ${{ matrix.python }} > .python-version` (test-python.yml, test-python-freethreaded.yml)
- `run: echo "python ${{ matrix.python }}" > .tool-versions` (test-python-freethreaded.yml)
- `run: ${{ startsWith(steps.setup-python.outputs.python-version, '3.14.') }}` (test-python.yml, test-python-freethreaded.yml)
- Multiline run blocks embedding `${{ matrix.python }}` inside heredoc-style echo strings (test-python.yml, test-python-freethreaded.yml, test-pypy.yml, e2e-cache.yml)

Locations:

- `.github/workflows/e2e-tests.yml:67`
- `.github/workflows/test-graalpy.yml:44`
- `.github/workflows/test-graalpy.yml:54`
- `.github/workflows/test-graalpy.yml:79`
- `.github/workflows/test-graalpy.yml:80`
- `.github/workflows/test-graalpy.yml:97`
- `.github/workflows/test-pypy.yml:47`
- `.github/workflows/test-pypy.yml:100`
- `.github/workflows/test-pypy.yml:115`
- `.github/workflows/test-pypy.yml:116`
- `.github/workflows/test-python-freethreaded.yml:37`
- `.github/workflows/test-python-freethreaded.yml:43`
- `.github/workflows/test-python-freethreaded.yml:47`
- `.github/workflows/test-python-freethreaded.yml:75`
- `.github/workflows/test-python-freethreaded.yml:81`
- `.github/workflows/test-python.yml:37`
- `.github/workflows/test-python.yml:57`
- `.github/workflows/test-python.yml:63`
- `.github/workflows/test-python.yml:97`
- `.github/workflows/test-python.yml:103`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three finding categories across 14 workflow files:

1. unpinned-uses: Pinned all mutable action references to full SHA digests:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10 (all files)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978
   - actions/publish-action@v0.4.0 → @23f4c6f12633a2da8f44938b71fde9afec138fb4
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 (4 files)

2. permissions: Added permissions blocks to 10 workflow files that were missing them. Used permissions: {} for reusable workflow callers and permissions: contents: read for test workflows.

3. script-injection: Moved all ${{ ... }} expressions from run: blocks into env: blocks, referencing them as plain environment variables. The startsWith() expression patterns were replaced with bash case statements. The EXECUTABLE=${{ matrix.graalpy }} pattern was moved to env: GRAALPY_VERSION. PowerShell validate steps now use $env:MATRIX_PYTHON instead of ${{ matrix.python }}. Heredoc-style echo strings with ${{ matrix.python }} were replaced with shell variable interpolation using env vars.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed two occurrences of `curl ... | python` unsafe shell patterns in `.github/workflows/e2e-cache.yml` (lines 82 and 195, in the `python-pipenv-dependencies-caching` and `python-pipenv-dependencies-caching-path` jobs). Each was replaced with a two-step approach: (1) download the script to `/tmp/get-pipenv.py` using `curl -sSL -o`, then (2) execute it separately with `python /tmp/get-pipenv.py`. This prevents a compromised remote server from executing arbitrary code by piping directly into the interpreter.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted shell/PowerShell variable expansions of untrusted data:
1. hardened/action/.github/workflows/test-pypy.yml: Changed `${EXECUTABLE} --version` to `"${EXECUTABLE}" --version` in all 5 occurrences (setup-pypy job, check-non-eol job, check-latest job, and two in setup-pypy-multiple-versions job).
2. hardened/action/.github/workflows/test-graalpy.yml: Changed `${EXECUTABLE} --version` to `"${EXECUTABLE}" --version` in both the setup-graalpy job and check-latest job.
3. hardened/action/.github/workflows/e2e-cache.yml: Changed `pipenv install --python $env:MATRIX_PYTHON_VERSION` to `pipenv install --python "$env:MATRIX_PYTHON_VERSION"` in both the python-pipenv-dependencies-caching job and python-pipenv-dependencies-caching-path job.

