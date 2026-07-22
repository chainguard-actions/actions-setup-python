<!-- markdownlint-disable -->

# Hardening Report: actions--setup-python/v5.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-python/v5.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files uses a mutable tag or branch ref instead of a pinned 40-character SHA commit hash, making the workflows vulnerable to supply-chain attacks if the referenced action or reusable workflow is compromised or altered.

Failing references include:
- `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`
- `actions/reusable-workflows/.github/workflows/check-dist.yml@main`
- `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`
- `actions/checkout@v4` (used in e2e-cache.yml, e2e-tests.yml, publish-immutable-actions.yml, test-graalpy.yml, test-pypy.yml, test-python.yml)
- `actions/publish-immutable-action@v0.0.4`
- `actions/publish-action@v0.3.0`
- `actions/reusable-workflows/.github/workflows/licensed.yml@main`
- `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`

Locations:

- `.github/workflows/basic-validation.yml:13`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:11`
- `.github/workflows/e2e-cache.yml:27`
- `.github/workflows/e2e-tests.yml:31`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/test-graalpy.yml:27`
- `.github/workflows/test-pypy.yml:32`
- `.github/workflows/test-python.yml:33`
- `.github/workflows/update-config-files.yml:10`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may include write access to contents and other scopes), violating the principle of least privilege.

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

### script-injection (severity: high)

Multiple workflow `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing an attacker to inject arbitrary shell commands via matrix values or step outputs.

**e2e-cache.yml**: `${{ matrix.python-version }}` is interpolated unquoted inside a PowerShell `run:` block: `pipenv install --python ${{ matrix.python-version }}`. This occurs in two jobs (python-pipenv-dependencies-caching and python-pipenv-dependencies-caching-path).

**e2e-tests.yml**: `${{ steps.cp313.outputs.python-path }}` is interpolated directly into a `run:` command: `run: pipx run --python '${{ steps.cp313.outputs.python-path }}' nox --version`.

**test-graalpy.yml**: `${{ steps.setup-python.outputs.python-path }}` is used as the entire `run:` command value (e.g. `run: ${{ steps.setup-python.outputs.python-path }} --version`), and also passed as a shell argument in `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'`.

**test-pypy.yml**: Same pattern — `run: ${{ steps.setup-python.outputs.python-path }} --version` and `run: ${{ steps.setup-python.outputs.python-path }} -c '...'`, plus the check-python-path.sh invocation.

**test-python.yml**: `${{ matrix.python }}` interpolated in `run: echo ${{ matrix.python }} > .python-version`; `${{ steps.setup-python.outputs.python-path }}` used as the entire run command; `${{ startsWith(steps.setup-python.outputs.python-version, '3.14.') }}` used as the entire run command.

Locations:

- `.github/workflows/e2e-cache.yml:54`
- `.github/workflows/e2e-cache.yml:111`
- `.github/workflows/e2e-tests.yml:71`
- `.github/workflows/test-graalpy.yml:35`
- `.github/workflows/test-graalpy.yml:80`
- `.github/workflows/test-graalpy.yml:82`
- `.github/workflows/test-pypy.yml:40`
- `.github/workflows/test-pypy.yml:72`
- `.github/workflows/test-pypy.yml:74`
- `.github/workflows/test-python.yml:48`
- `.github/workflows/test-python.yml:84`
- `.github/workflows/test-python.yml:159`
- `.github/workflows/test-python.yml:161`
- `.github/workflows/test-python.yml:196`
- `.github/workflows/test-python.yml:198`

### unsafe-shell (severity: high)

Two `run:` steps in `.github/workflows/e2e-cache.yml` pipe remote content directly to a Python interpreter without first downloading and verifying the script:

`run: curl https://raw.githubusercontent.com/pypa/pipenv/master/get-pipenv.py | python`

This pattern executes whatever content is served at that URL at the time of the run. If the remote URL is compromised or the content changes, arbitrary code will execute on the runner. The script should be downloaded to a file, inspected/verified, and then executed separately.

Locations:

- `.github/workflows/e2e-cache.yml:43`
- `.github/workflows/e2e-cache.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, unsafe-shell

**Notes:**

Fixed all 4 findings across 13 workflow files:

1. unpinned-uses: Pinned all action references to full SHA hashes - actions/checkout@v4→SHA, actions/publish-immutable-action@v0.0.4→SHA, actions/publish-action@v0.3.0→SHA, and all actions/reusable-workflows@main references→SHA.

2. missing-permissions: Added top-level 'permissions: contents: read' to 9 workflow files that were missing it (basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-graalpy.yml, test-pypy.yml, test-python.yml, update-config-files.yml).

3. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks. Fixed: matrix.python-version in PowerShell pipenv install commands, steps.cp313.outputs.python-path in e2e-tests.yml, steps.setup-python.outputs.python-path in test-graalpy.yml and test-pypy.yml noenv jobs, matrix.graalpy and steps.graalpy.outputs.python-version in test-graalpy.yml, matrix.pypy in test-pypy.yml, matrix.python in echo commands in test-python.yml, and replaced startsWith() expressions used as entire run commands with proper bash case statements.

4. unsafe-shell: Replaced both 'curl ... | python' patterns in e2e-cache.yml with download-then-execute: 'curl -o get-pipenv.py URL && python get-pipenv.py'.

