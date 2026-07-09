<!-- markdownlint-disable -->

# Hardening Report: actions--setup-python/v5.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--setup-python/v5.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions, violating rule (a). Examples include: `run: pipx run --python '${{ steps.cp313.outputs.python-path }}' nox --version` (e2e-tests.yml:76), `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'` (test-graalpy.yml:39, test-pypy.yml, test-python.yml), `run: ${{ steps.setup-python.outputs.python-path }} --version` (test-graalpy.yml:86, test-pypy.yml, test-python.yml), `EXECUTABLE=${{ matrix.graalpy }}` inside a run block (test-graalpy.yml:56), `run: echo ${{ matrix.python }} > .python-version` (test-python.yml:79), `pipenv install --python ${{ matrix.python-version }}` inside a run block (e2e-cache.yml), and `run: ${{ startsWith(steps.setup-python.outputs.python-version, '3.14.') }}` (test-python.yml). Any ${{ }} expression interpolated directly into a run: shell command is a script-injection risk.

Locations:

- `.github/workflows/e2e-tests.yml:76`
- `.github/workflows/e2e-cache.yml:55`
- `.github/workflows/e2e-cache.yml:111`
- `.github/workflows/test-graalpy.yml:39`
- `.github/workflows/test-graalpy.yml:56`
- `.github/workflows/test-graalpy.yml:86`
- `.github/workflows/test-graalpy.yml:89`
- `.github/workflows/test-graalpy.yml:102`
- `.github/workflows/test-pypy.yml:36`
- `.github/workflows/test-pypy.yml:47`
- `.github/workflows/test-pypy.yml:63`
- `.github/workflows/test-pypy.yml:65`
- `.github/workflows/test-python.yml:44`
- `.github/workflows/test-python.yml:50`
- `.github/workflows/test-python.yml:79`
- `.github/workflows/test-python.yml:88`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or branch names instead of pinned full-length SHA commits. Failing references include: `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`, `actions/reusable-workflows/.github/workflows/check-dist.yml@main`, `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`, `actions/checkout@v4`, `actions/reusable-workflows/.github/workflows/licensed.yml@main`, `actions/publish-immutable-action@v0.0.4`, `actions/publish-action@v0.3.0`, `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`.

Locations:

- `.github/workflows/basic-validation.yml:13`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/e2e-cache.yml:26`
- `.github/workflows/e2e-tests.yml:31`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/release-new-action-version.yml:19`
- `.github/workflows/test-graalpy.yml:30`
- `.github/workflows/test-pypy.yml:30`
- `.github/workflows/test-python.yml:35`
- `.github/workflows/update-config-files.yml:12`

### missing-permissions (severity: medium)

Multiple workflow files have no top-level `permissions:` key and no job-level `permissions:` keys. This means workflows run with the default (potentially broad) token permissions. Affected files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-graalpy.yml, test-pypy.yml, test-python.yml, update-config-files.yml.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-graalpy.yml:1`
- `.github/workflows/test-pypy.yml:1`
- `.github/workflows/test-python.yml:1`
- `.github/workflows/update-config-files.yml:1`

### unsafe-shell (severity: high)

Two steps in e2e-cache.yml pipe remote content directly to a Python interpreter without first saving to a file: `run: curl https://raw.githubusercontent.com/pypa/pipenv/master/get-pipenv.py | python`. This allows arbitrary code execution from a remote URL.

Locations:

- `.github/workflows/e2e-cache.yml:44`
- `.github/workflows/e2e-cache.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, unsafe-shell

**Notes:**

Fixed all four finding types across 12 workflow files:

1. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as environment variables. The `run: ${{ startsWith(...) }}` pattern (using an expression as a shell command) was replaced with a proper bash case statement.

2. unpinned-uses: Pinned all mutable references to full commit SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 # v4
   - actions/reusable-workflows/...@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 # main
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978 # v0.0.4
   - actions/publish-action@v0.3.0 → @f784495ce78a41bac4ed7e34a73f0034015764bb # v0.3.0

3. missing-permissions: Added `permissions: contents: read` to all 9 workflow files that lacked permissions blocks.

4. unsafe-shell: Replaced both `curl ... | python` patterns in e2e-cache.yml with a safe two-step approach: download script to /tmp/get-pipenv.py first, then execute it separately.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion of untrusted data in two workflow files:
1. hardened/action/.github/workflows/test-graalpy.yml: Quoted `${EXECUTABLE}` as `"${EXECUTABLE}"` in both the setup-graalpy job (line 58) and the check-latest job (line 108).
2. hardened/action/.github/workflows/test-pypy.yml: Quoted `${EXECUTABLE}` as `"${EXECUTABLE}"` in the setup-pypy job (line 60), and also fixed additional unquoted instances in the check-latest and setup-pypy-multiple-versions jobs for consistency.
All instances of `${EXECUTABLE} --version` have been replaced with `"${EXECUTABLE}" --version` to prevent command injection via attacker-controlled matrix values containing shell metacharacters.

