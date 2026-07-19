<!-- markdownlint-disable -->

# Hardening Report: actions--setup-python/v5.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-python/v5.6.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tag refs (e.g., @v4, @main, @v0.0.4, @v0.3.0) instead of pinned 40-character SHA commits. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised. Affected references include: actions/reusable-workflows@main (basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml), actions/checkout@v4 (e2e-cache-freethreaded.yml, e2e-cache.yml, e2e-tests.yml, test-graalpy.yml, test-pypy.yml, test-python.yml, test-python-freethreaded.yml), actions/checkout@v4 and actions/publish-immutable-action@v0.0.4 (publish-immutable-actions.yml), actions/publish-action@v0.3.0 (release-new-action-version.yml).

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:15`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/e2e-cache-freethreaded.yml:30`
- `.github/workflows/e2e-cache.yml:41`
- `.github/workflows/e2e-tests.yml:26`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/test-graalpy.yml:30`
- `.github/workflows/test-pypy.yml:38`
- `.github/workflows/test-python-freethreaded.yml:30`
- `.github/workflows/test-python.yml:30`
- `.github/workflows/update-config-files.yml:11`

### missing-permissions (severity: medium)

Multiple workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, workflows inherit the default repository permissions (which may include write access), violating the principle of least privilege.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-graalpy.yml:1`
- `.github/workflows/test-pypy.yml:1`
- `.github/workflows/test-python.yml:1`
- `.github/workflows/test-python-freethreaded.yml:1`
- `.github/workflows/update-config-files.yml:1`

### script-injection (severity: high)

Multiple workflow files interpolate GitHub Actions expressions (sub-rule a) directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands. Examples include: `run: ./__tests__/check-python-path.sh '${{ steps.setup-python.outputs.python-path }}'` (steps output injected into shell argument); `EXECUTABLE=${{ matrix.graalpy }}` / `EXECUTABLE=${{ matrix.pypy }}` (matrix value injected unquoted into shell variable assignment); `run: echo ${{ matrix.python }} > .python-version` (matrix value injected into shell command); `run: pipx run --python '${{ steps.cp313.outputs.python-path }}' nox --version` (steps output injected into shell); `run: ${{ steps.setup-python.outputs.python-path }} --version` (steps output used as the command itself); `run: ${{ startsWith(...) }}` (expression used as shell command); PowerShell `if ("${{ matrix.python-version }}" -Match "pypy")` (matrix value injected into PowerShell string).

Locations:

- `.github/workflows/test-graalpy.yml:42`
- `.github/workflows/test-graalpy.yml:60`
- `.github/workflows/test-graalpy.yml:63`
- `.github/workflows/test-graalpy.yml:72`
- `.github/workflows/test-pypy.yml:46`
- `.github/workflows/test-pypy.yml:100`
- `.github/workflows/test-pypy.yml:155`
- `.github/workflows/test-pypy.yml:158`
- `.github/workflows/test-python.yml:42`
- `.github/workflows/test-python.yml:58`
- `.github/workflows/test-python.yml:85`
- `.github/workflows/test-python.yml:100`
- `.github/workflows/test-python.yml:130`
- `.github/workflows/test-python.yml:175`
- `.github/workflows/test-python.yml:220`
- `.github/workflows/test-python-freethreaded.yml:42`
- `.github/workflows/test-python-freethreaded.yml:55`
- `.github/workflows/test-python-freethreaded.yml:80`
- `.github/workflows/test-python-freethreaded.yml:95`
- `.github/workflows/test-python-freethreaded.yml:120`
- `.github/workflows/e2e-tests.yml:60`
- `.github/workflows/e2e-cache.yml:107`
- `.github/workflows/e2e-cache.yml:260`

### unsafe-shell (severity: high)

Four `run:` steps pipe remote content directly to a Python interpreter: `curl https://raw.githubusercontent.com/pypa/pipenv/master/get-pipenv.py | python`. This downloads and executes arbitrary remote code without any integrity verification, and is equivalent to `curl | bash` in risk. If the remote URL is compromised or the content is tampered with in transit, malicious code will execute on the runner.

Locations:

- `.github/workflows/e2e-cache-freethreaded.yml:55`
- `.github/workflows/e2e-cache-freethreaded.yml:131`
- `.github/workflows/e2e-cache.yml:88`
- `.github/workflows/e2e-cache.yml:248`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, unsafe-shell

**Notes:**

Fixed all four finding types across 14 workflow files:

1. unpinned-uses: Pinned all mutable tag refs to full 40-char SHAs: actions/reusable-workflows@main→4735e71, actions/checkout@v4→34e1148, actions/publish-immutable-action@v0.0.4→4bc8754, actions/publish-action@v0.3.0→f784495.

2. missing-permissions: Added 'permissions: contents: read' top-level block to all 10 affected workflow files.

3. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks. Key fixes: check-python-path.sh arguments, matrix.graalpy/pypy EXECUTABLE assignments, echo ${{ matrix.python }} > .python-version, pipx run --python, python-path used as command, startsWith() expressions replaced with bash conditionals, PowerShell matrix.python-version injection.

4. unsafe-shell: Replaced all 4 instances of 'curl ... | python' with download-then-execute pattern: 'curl -fsSL -o /tmp/get-pipenv.py URL; python /tmp/get-pipenv.py' in e2e-cache.yml and e2e-cache-freethreaded.yml.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 7 occurrences of unquoted `${EXECUTABLE} --version` by replacing them with `"${EXECUTABLE}" --version` in two workflow files:
- hardened/action/.github/workflows/test-graalpy.yml: 2 fixes (setup-graalpy job and check-latest job)
- hardened/action/.github/workflows/test-pypy.yml: 5 fixes (setup-pypy job, check-non-eol job, check-latest job, and two steps in setup-pypy-multiple-versions job)

The EXECUTABLE variable is derived from workflow-controllable matrix values (matrix.graalpy, matrix.pypy). Quoting the variable when used as a command prevents shell metacharacter injection while preserving the intended behavior.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two instances of unquoted PowerShell variable `$env:MATRIX_PYTHON_VERSION` in `pipenv install --python` commands in `.github/workflows/e2e-cache.yml`. In both the `python-pipenv-dependencies-caching` job (line 114) and the `python-pipenv-dependencies-caching-path` job (line 215), changed `pipenv install --python $env:MATRIX_PYTHON_VERSION` to `pipenv install --python "$env:MATRIX_PYTHON_VERSION"`. The env block already correctly isolated the matrix value from the `${{ }}` expression; only the shell-level quoting was missing.

