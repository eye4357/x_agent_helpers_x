# Productionization Playbook For Future Repos

This playbook defines the standard for turning a small local utility repo into a professional, repeatable, release-ready project.

It is based on the productionization pattern already used for:

- `x_get_win_py_x`
- `x_create_github_repos_x`

Use this document before productionizing `x_create_cv_x` and every future repo.

## Productionized Means

A repo is productionized when a new user or future maintainer can clone it, understand it, install its development tools, run automated checks, trust the tests, and see a clear change-control record without needing private context from the original workspace.

Minimum definition:

- The repo has a clear name, purpose, and public entry point.
- The README explains setup, usage, development, validation, and troubleshooting.
- The tool exposes `--version` or an equivalent version command.
- Project metadata lives in `pyproject.toml`.
- Development dependencies are pinned and installable through `requirements-dev.txt`.
- Formatting, linting, typing, tests, and CI are configured.
- Unit tests mock all dangerous, external, paid, private, network, filesystem-destructive, or machine-mutating boundaries.
- GitHub Actions runs the same quality gate expected locally.
- When repeated unauthenticated GitHub REST checks hit rate limits, confirm public CI status from the workflow run page before recording the checkpoint; if the public page serves a stale in-progress snapshot, refresh with a cache-busting query; do not infer success from an in-progress API response.
- VS Code settings encourage strict Python analysis and pytest discovery.
- `.gitignore` protects generated files, environments, caches, secrets, and private data.
- `CHANGE_CONTROL_PACKET.md` records scope, controlled files, changes, validation, rollback, and operator notes.
- Private data, tokens, machine-specific state, and scratch extraction outputs are not committed.

## Standard Repo Shape

For a single-file Python CLI utility:

```text
repo_name/
  .github/
    workflows/
      ci.yml
  .vscode/
    settings.json
  .gitignore
  CHANGE_CONTROL_PACKET.md
  README.md
  pyproject.toml
  requirements-dev.txt
  tests/
    test_repo_name.py
  repo_name.py
```

For a utility with helper scripts:

```text
repo_name/
  .github/
    workflows/
      ci.yml
  .vscode/
    settings.json
  .gitignore
  CHANGE_CONTROL_PACKET.md
  README.md
  pyproject.toml
  requirements-dev.txt
  tests/
    test_repo_name.py
  repo_name.py
  helper_script.ps1
  another_helper.ps1
```

For a larger app or service:

```text
repo_name/
  .github/
    workflows/
      ci.yml
  .vscode/
    settings.json
  docs/
    architecture.md
    privacy.md
    operations.md
  examples/
    fake_input.json
    fake_output.json
  src_or_package_name/
    __init__.py
    cli.py
    models.py
    services.py
    storage.py
  tests/
    fixtures/
    test_cli.py
    test_models.py
    test_services.py
  .gitignore
  CHANGELOG.md
  CHANGE_CONTROL_PACKET.md
  README.md
  SECURITY.md
  pyproject.toml
  requirements-dev.txt
```

Use the single-file shape first unless the repo has real module boundaries.

## Phase 0: Preflight

Before editing, answer these questions in writing or in the working notes:

- What is the repo's exact public name?
- Does the GitHub repo name match the tool name?
- What is the main entry point?
- Is it a CLI, library, service, app, script collection, or mixed tool?
- What operating systems must it support?
- Does it mutate the local machine?
- Does it call a network API?
- Does it need tokens, passwords, credentials, or local secrets?
- Does it process PII, private files, or generated artifacts?
- Which files are source code and which are scratch/generated/private outputs?
- Which external boundaries must be mocked in tests?
- What is the minimal quality gate that proves it works?

Do not productionize until private data and generated scratch files are clearly separated from public source.

## Phase 1: Repo Identity

Required:

- GitHub repository name matches the tool or product name.
- Local folder name matches the GitHub repository name.
- Main script/package name is predictable from the repo name.
- README title matches the product name, not an old working name.
- Remote origin points to the correct GitHub URL.
- Default branch is `main`.
- Repo visibility is intentional: public only if no secrets or PII are committed.

Checks:

```powershell
git remote -v
git status --short --branch
git branch --show-current
```

Search for stale names:

```powershell
Select-String -Path .\* -Pattern "old_repo_name" -Recurse
```

If GitHub repo name is wrong, rename it before release. A wrong name leaks into clone URLs, docs, package metadata, and user trust.

## Phase 2: Source Cleanup

Required:

- Keep only current source, docs, tests, examples, and config.
- Remove `__pycache__`, `.pytest_cache`, `.ruff_cache`, `.mypy_cache`, build output, scratch output, temp extraction folders, local logs, and generated archives.
- Keep private files only under ignored paths.
- Backup before deleting old local data.
- Verify the backup before pruning.
- If a backup fails, stop immediately and repair before deleting.

Private-data rule:

- Public repo gets fake fixtures and schemas.
- Private repo/local ignored folder gets real data.
- CI must never require real private data.

For projects like `x_create_cv_x`, public source must not include real CV data, raw extraction files, private seed scripts, private zips, private JSON, addresses, emails, phone numbers, or personal history details.

## Phase 3: `.gitignore`

Start from the standard Python `.gitignore` and add project-specific private or generated paths.

Required categories:

- Python bytecode: `__pycache__/`, `*.py[codz]`
- Build artifacts: `build/`, `dist/`, `*.egg-info/`
- Virtual environments: `.venv/`, `venv/`, `env/`
- Test and coverage output: `.pytest_cache/`, `.coverage`, `htmlcov/`
- Type/lint caches: `.mypy_cache/`, `.ruff_cache/`, `.pyre/`
- Tool secrets: `.env`, `.envrc`, `.pypirc`
- Editor/runtime scratch: `tempCodeRunnerFile.py`
- Project private data: for example `data/private/`
- Project scratch extraction data: for example `temp data/`

Validation:

```powershell
git status --short --ignored
git check-ignore -v path\to\private_file
```

Acceptance criteria:

- Private files show under ignored status.
- Source files and docs remain visible to Git.
- No important source file is accidentally ignored.

## Phase 4: Versioning

Every productionized tool needs a version.

Required:

- Set initial version to `0.0.1` unless there is a reason to start higher.
- Add a module constant such as `VERSION = "0.0.1"` or `__version__ = "0.0.1"`.
- Add `--version` for Python CLIs.
- Add `-Version` for PowerShell scripts.
- Show the version in README.
- Show the version in `CHANGE_CONTROL_PACKET.md`.
- Match `pyproject.toml` version.

Validation examples:

```powershell
python .\tool_name.py --version
powershell -ExecutionPolicy Bypass -File .\helper.ps1 -Version
```

## Phase 5: `pyproject.toml`

Every Python repo gets project metadata and tool config.

Required sections:

- `[project]`
- `[project.optional-dependencies]`
- `[tool.black]`
- `[tool.ruff]`
- `[tool.ruff.lint]`
- `[tool.mypy]`
- `[tool.pytest.ini_options]`

Baseline template for a single-file CLI:

```toml
[project]
name = "repo-name-with-dashes"
version = "0.0.1"
description = "Short plain-English description."
requires-python = ">=3.10"
dependencies = []

[project.optional-dependencies]
dev = [
    "black==24.10.0",
    "mypy==1.14.1",
    "pytest==8.3.4",
    "ruff==0.8.6",
]

[tool.black]
line-length = 120
target-version = ["py310"]

[tool.ruff]
line-length = 120
target-version = "py310"

[tool.ruff.lint]
select = ["B", "C4", "E", "F", "I", "RUF", "SIM", "UP"]

[tool.mypy]
python_version = "3.10"
strict = true
files = ["tool_name.py", "tests"]

[tool.pytest.ini_options]
testpaths = ["tests"]
```

Windows-specific tools can add:

```toml
[tool.mypy]
platform = "win32"
```

For package repos, point mypy at the package and tests:

```toml
files = ["package_name", "tests"]
```

Rules:

- Prefer stdlib dependencies when reasonable.
- Add runtime dependencies only when they remove real complexity.
- Pin dev dependencies.
- Keep line length at 120 unless a project has a stronger local reason.
- Keep mypy strict.

## Phase 6: `requirements-dev.txt`

Use one line:

```text
-e .[dev]
```

This keeps dev tooling in one canonical place: `pyproject.toml`.

Validation:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
```

## Phase 7: VS Code Settings

Add `.vscode/settings.json` for consistent local behavior.

Baseline:

```json
{
    "python.analysis.typeCheckingMode": "strict",
    "python.analysis.diagnosticMode": "workspace",
    "python.testing.pytestEnabled": true,
    "python.testing.unittestEnabled": false,
    "python.testing.pytestArgs": [
        "tests"
    ],
    "python.defaultInterpreterPath": "${workspaceFolder}\\.venv\\Scripts\\python.exe",
    "[python]": {
        "editor.defaultFormatter": "ms-python.black-formatter",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.organizeImports": "explicit",
            "source.fixAll.ruff": "explicit"
        }
    }
}
```

Do not commit personal machine paths except the conventional workspace `.venv` path.

## Phase 8: CLI Standards

Required for command-line tools:

- Use `argparse` unless a framework is clearly needed.
- Provide `--help`.
- Provide `--version`.
- Return integer process codes from `main()`.
- Use `if __name__ == "__main__": raise SystemExit(main())`.
- Print normal results to stdout.
- Print errors to stderr.
- Use exit code `0` for success.
- Use exit code `2` for usage/configuration errors when appropriate.
- Avoid doing work at import time.
- Keep side-effect boundaries injectable or mockable.

For scripts that call APIs:

- Never print tokens.
- Explain missing-token errors clearly.
- Convert common API failures into actionable messages.
- Add endpoint-specific troubleshooting if a real user will hit permission errors.

For scripts that mutate a machine:

- Add dry-run behavior if practical.
- Print a summary of what changed.
- Keep destructive operations behind explicit flags.
- Test PATH, environment, registry, filesystem, process, and network changes through mocks.

## Phase 9: PowerShell Standards

Required if the repo includes `.ps1` files:

- Add `-Version`.
- Add comment-based help if the script has more than a few flags.
- Set `$ErrorActionPreference = "Stop"` near the top.
- Avoid leaking secrets.
- Use explicit parameter types.
- Support non-interactive operation.
- CI must parse every `.ps1` file with the PowerShell parser.
- README must show `powershell -ExecutionPolicy Bypass -File .\script.ps1` when execution policy may block the script.

CI parser check:

```yaml
      - name: PowerShell syntax check
        shell: pwsh
        run: |
          $ErrorActionPreference = "Stop"
          foreach ($script in Get-ChildItem -Filter *.ps1) {
            $tokens = $null
            $errors = $null
            [System.Management.Automation.Language.Parser]::ParseFile($script.FullName, [ref]$tokens, [ref]$errors) | Out-Null
            if ($errors.Count -gt 0) {
              $errors | ForEach-Object { Write-Error $_.Message }
            }
          }
```

Use `windows-latest` for CI when PowerShell behavior is central to the project.

## Phase 10: Tests

Create `tests/` and add focused unit tests.

Required test characteristics:

- Tests import the tool module directly.
- Tests avoid live network calls.
- Tests avoid real installs, uninstalls, account changes, file deletion outside temp dirs, or PATH mutation.
- Tests use `pytest.MonkeyPatch`, `tmp_path`, and `capsys` where useful.
- Tests cover argument parsing for important CLI paths.
- Tests cover happy path and at least the main failure path.
- Tests cover external boundary adapters through fakes.
- Tests prove error messages contain the actionable guidance users need.

Coverage targets by project type:

- API tool: endpoint selection, request payloads, auth checks, 403/401/404 guidance, response validation.
- Installer/machine tool: discovery, fallback behavior, path rewriting, installed-state detection, subprocess failure handling.
- Data factory: schema creation, record insertion, validation, deterministic JSON output, private-data exclusion, fake fixture rebuilds.
- Renderer/exporter: stable text output, formatting edge cases, missing data handling, export errors.
- Web service: route tests, validation errors, auth boundaries, storage isolation, no private data in logs.

Renderer conformance hardening:

- Use public fake fixtures that exercise the same renderer paths as private evidence.
- Add exact generated artifact assertions only when inputs, default package flags, serializers, ordering, IDs, timestamps, and relationship targets are deterministic.
- Prefer one package part or one renderer surface per slice, then run the narrowest behavior-scoped test before making adjacent edits.
- Assert exact package manifests only when the enabled package flags and emitted part set are deterministic and already covered by public fake fixtures.
- Assert content type omissions only when disabled package flags make the absent overrides deterministic and already covered by the package manifest.
- Assert renderer summary counters from the summary API's semantics and the exact local fixture, not nearby visual intuition or a richer neighboring fixture; for example, body child counts and paragraph element counts may intentionally differ.
- Assert renderer summary metadata projections only when they are derived from an exact generated local part already fixed by the same public fake fixture; changing the metadata itself is a design decision.
- Assert audit or comparison report byte, normalized-text, and structure parity when the public fake source artifact is intentionally copied from the generated artifact; pin representative summary keys from the exact local fixture, not assumed default counts.
- Assert human-readable audit report rows only after checking the rendered scalar format; Markdown may render booleans, paths, and dictionaries differently than Python literals.
- Probe each sibling artifact section before copying an exact row assertion across them; the same metric can intentionally differ between fixtures, such as DOCX page margins for different resume variants.
- Assert report metadata paths by deriving them from the same path constants or normalization helpers as the reporter; do not assume relative paths when the reporter records resolved paths.
- Treat generated package sidecar metadata as assertable public coverage only when IDs, schema references, and payloads are fixed and contain no private content.
- Treat generated theme or styling package parts as assertable public coverage only when the serializer output is fixed; selecting or changing the visual theme is a design decision.
- Treat template choices, default behavior changes, schema changes, privacy boundaries, and compatibility strategy as design decisions, not medium assertion-hardening work.

Example test style:

```python
from __future__ import annotations

import pytest

import tool_name as app


def test_main_requires_token(monkeypatch: pytest.MonkeyPatch, capsys: pytest.CaptureFixture[str]) -> None:
    monkeypatch.delenv("GITHUB_TOKEN", raising=False)

    assert app.main(["repo-name"]) == 2

    captured = capsys.readouterr()
    assert "Set GITHUB_TOKEN" in captured.err
```

Acceptance criteria:

- `python -m pytest` passes.
- Tests do not require secrets.
- Tests do not require live services.
- Tests are deterministic.

## Phase 11: Linting, Formatting, Typing

Required local gate:

```powershell
python -m pytest
ruff check .
black --check .
mypy
```

For first-time setup:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
```

Before final release, also run:

```powershell
python -m py_compile .\tool_name.py
python .\tool_name.py --help
python .\tool_name.py --version
```

If the repo has multiple Python files, compile them all:

```powershell
python -m compileall .
```

Do not relax mypy strict mode unless the repo has an explicit documented reason.

## Phase 12: GitHub Actions

Add `.github/workflows/ci.yml`.

Baseline Ubuntu workflow for pure Python utilities:

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  quality:
    name: Quality Gates
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - name: Install development dependencies
        run: python -m pip install --upgrade pip && python -m pip install -r requirements-dev.txt

      - name: Ruff lint
        run: ruff check .

      - name: Black format check
        run: black --check .

      - name: Mypy strict type check
        run: mypy

      - name: Pytest
        run: pytest
```

Use Windows workflow when the tool is Windows-specific or includes PowerShell helpers:

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  quality:
    name: Quality Gates
    runs-on: windows-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - name: Install development dependencies
        run: python -m pip install --upgrade pip && python -m pip install -r requirements-dev.txt

      - name: Ruff lint
        run: ruff check .

      - name: Black format check
        run: black --check .

      - name: Mypy strict type check
        run: mypy

      - name: Pytest
        run: pytest

      - name: PowerShell syntax check
        shell: pwsh
        run: |
          $ErrorActionPreference = "Stop"
          foreach ($script in Get-ChildItem -Filter *.ps1) {
            $tokens = $null
            $errors = $null
            [System.Management.Automation.Language.Parser]::ParseFile($script.FullName, [ref]$tokens, [ref]$errors) | Out-Null
            if ($errors.Count -gt 0) {
              $errors | ForEach-Object { Write-Error $_.Message }
            }
          }
```

Rules:

- CI must not require secrets for normal checks.
- CI must not create GitHub repos, install software, mutate PATH, or use private data.
- CI should use fake fixtures and mocks.
- CI should run on push and pull request.
- CI should match the local validation gate.

## Phase 13: README

The README must be useful to someone who has never seen the workspace.

Required sections:

- Title
- One-paragraph purpose
- Version
- Requirements
- Setup or prerequisites
- Usage examples
- Options or commands
- Development and release checks
- Testing notes
- Troubleshooting
- Link to `CHANGE_CONTROL_PACKET.md`

For token/API projects, include:

- Token type recommendation.
- Minimal permissions.
- Where to create the token.
- How to set it in PowerShell.
- Common authentication and authorization errors.
- Reminder not to commit the token.

For Windows/PowerShell projects, include:

- Execution policy example.
- How to run with `powershell -ExecutionPolicy Bypass -File`.
- What to do if PATH changes are not visible in VS Code.

For private-data projects, include:

- Public repo contains fake examples only.
- Real data lives under ignored paths.
- How to validate using fake fixtures.
- How private/local validation works without exposing PII.

README development block:

````markdown
## Development And Release Checks

Install the local development toolchain:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
```

Run the full local quality gate before release:

```powershell
python -m pytest
ruff check .
black --check .
mypy
```
````

Do not include private values, personal data, raw logs, local absolute paths, or one-off scratch instructions in public README docs.

## Phase 14: Change Control Packet

Every productionized repo gets `CHANGE_CONTROL_PACKET.md`.

Use this for the current controlled release record. It is more detailed than a normal changelog entry and should include validation and rollback.

Template:

```markdown
# Change Control Packet

Tool: `tool_name`
Version: `0.0.1`
Packet ID: `tool_name-0.0.1-YYYY-MM-DD`
Date: `YYYY-MM-DD`
Status: Updated

## Scope

One or two sentences describing what this controlled version establishes.

## Controlled Files

- `tool_name.py`
- `README.md`
- `CHANGE_CONTROL_PACKET.md`
- `pyproject.toml`
- `requirements-dev.txt`
- `.github/workflows/ci.yml`
- `.vscode/settings.json`
- `tests/test_tool_name.py`

## Change Summary

- Added explicit tool version `0.0.1`.
- Added `--version` CLI output.
- Documented setup, usage, validation, and troubleshooting in the README.
- Added pytest coverage for important behavior and failure paths.
- Added pinned Ruff, Black, mypy, pytest, Pylance strict settings, and GitHub Actions CI.

## Validation

- Run `python .\tool_name.py --version`.
- Run `python .\tool_name.py --help`.
- Run `python -m py_compile .\tool_name.py`.
- Run `python -m pytest`.
- Run `ruff check .`.
- Run `black --check .`.
- Run `mypy`.

## Rollback

Restore the previous script and README versions, then remove this packet if reverting before release.

## Operator Notes

Mention tokens, OS requirements, private-data rules, side effects, or operational caveats.
```

Rules:

- Keep the packet current with the release being prepared.
- Update the controlled files list when files are added.
- Validation must list the commands actually run.
- Operator notes must mention dangerous side effects and private-data boundaries.

## Phase 15: Changelog

For small internal utilities, `CHANGE_CONTROL_PACKET.md` is the primary release record.

Add `CHANGELOG.md` when:

- The repo will have multiple public releases.
- Users need a historical release log.
- The project becomes a package, app, service, or public showcase.

Recommended format:

```markdown
# Changelog

## 0.0.1 - YYYY-MM-DD

### Added

- Initial productionized release.
- CLI version output.
- README setup and validation docs.
- Tests and CI quality gate.

### Changed

- N/A

### Fixed

- N/A

### Security

- Documented secret/private-data handling.
```

Rules:

- Changelog records user-facing changes over time.
- Change-control packet records the current controlled release details, validation, and rollback.
- Use both for serious public repos.

## Phase 16: Security And Secrets

Required:

- No tokens in source, tests, README, logs, examples, fixtures, or screenshots.
- No private PII in public files.
- No real email, phone, address, resume, customer, account, or token values in fake examples.
- `.env` and private data folders are ignored.
- README tells users how to provide secrets safely.
- Tests use fake tokens like `token` or `example-token` only.
- Error messages do not echo secrets.
- Logs avoid request bodies when they may contain sensitive data.

For public repos, consider adding `SECURITY.md` when:

- The tool handles credentials.
- The tool mutates machines.
- The tool processes private data.
- The tool has a hosted service path.

`SECURITY.md` should cover:

- Supported versions.
- How to report vulnerabilities.
- Secret handling.
- Private-data boundaries.
- Known non-goals.

## Phase 17: Examples And Fixtures

Examples must be fake but realistic enough to exercise the product.

Required for data-heavy repos:

- `examples/` or `tests/fixtures/` with synthetic input.
- Expected output fixtures.
- A test proving synthetic seeds rebuild expected output.
- Documentation stating examples are fake.

Forbidden:

- Real PII.
- Real customer data.
- Real tokens.
- Real private archive contents.
- Generated private outputs copied into public fixtures.

For `x_create_cv_x`, create fake CV fixtures before CI. The private golden zip can remain local, but CI must validate fake data only.

## Phase 18: Documentation Set

Minimum for small utilities:

- `README.md`
- `CHANGE_CONTROL_PACKET.md`

Recommended for larger projects:

- `CHANGELOG.md`
- `SECURITY.md`
- `docs/architecture.md`
- `docs/privacy.md`
- `docs/operations.md`
- `docs/testing.md`

For showcase projects, also include:

- `docs/roadmap.md`
- `docs/demo-data.md`
- Screenshots or terminal examples that use fake data only.

## Phase 19: GitHub Repo Settings

Recommended GitHub settings:

- Description set.
- Website/demo URL set if available.
- Topics added.
- Default branch is `main`.
- Issues enabled if you want feedback.
- Wiki disabled unless intentionally used.
- Projects disabled unless intentionally used.
- Actions enabled.
- Branch protection added once CI is stable.

Suggested topics for Python utilities:

- `python`
- `cli`
- `automation`
- project-specific topic such as `github-api`, `windows`, `resume-builder`, or `local-first`

Branch protection once stable:

- Require pull request before merging.
- Require status checks to pass.
- Require the CI workflow.
- Prevent force pushes to `main`.

Do not enable strict protection until the repo can reliably pass CI.

## Phase 20: Release Validation

Run the release gate locally before pushing.

Standard Python utility gate:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
python .\tool_name.py --version
python .\tool_name.py --help
python -m py_compile .\tool_name.py
python -m pytest
ruff check .
black --check .
mypy
git status --short
```

PowerShell helper gate:

```powershell
powershell -ExecutionPolicy Bypass -File .\helper.ps1 -Version
```

Parser gate:

```powershell
$ErrorActionPreference = "Stop"
foreach ($script in Get-ChildItem -Filter *.ps1) {
    $tokens = $null
    $errors = $null
    [System.Management.Automation.Language.Parser]::ParseFile($script.FullName, [ref]$tokens, [ref]$errors) | Out-Null
    if ($errors.Count -gt 0) {
        $errors | ForEach-Object { Write-Error $_.Message }
    }
}
```

Data factory gate:

```powershell
python .\tool_name.py validate --db-dir .\path\to\fake_output --expected-zip .\path\to\fake_expected.zip --zip-prefix expected
```

Rules:

- If validation fails, fix root cause before pushing.
- If a failure is unrelated and pre-existing, document it clearly before proceeding.
- Do not publish a repo where CI depends on private local files.

## Phase 21: Commit And Push

Before commit:

```powershell
git status --short
git diff -- .
```

Confirm:

- No private data is staged.
- No cache/build output is staged.
- Docs mention only public-safe paths and examples.
- Version numbers match.
- `CHANGE_CONTROL_PACKET.md` controlled files list is accurate.
- Tests pass locally.

Commit message examples:

```text
Productionize Python CLI release
Add CI and change-control packet
Add tests and release validation docs
```

Push:

```powershell
git push origin main
```

After push:

- Confirm GitHub Actions starts.
- Confirm GitHub Actions passes.
- Fix failed CI before calling the repo productionized.

## Phase 22: Productionization Acceptance Checklist

Use this checklist for every repo.

Identity:

- [ ] Repo name matches tool/product name.
- [ ] Local folder name matches GitHub repo name.
- [ ] Remote origin points to the correct GitHub URL.
- [ ] Default branch is `main`.
- [ ] Stale old-name references are removed.

Source hygiene:

- [ ] Only current source, docs, tests, examples, and config are committed.
- [ ] Private data is ignored.
- [ ] Generated scratch data is ignored or removed.
- [ ] Caches and build outputs are absent.
- [ ] Backup exists before pruning old material.

Project config:

- [ ] `pyproject.toml` exists.
- [ ] `requirements-dev.txt` exists and contains `-e .[dev]`.
- [ ] `.vscode/settings.json` exists.
- [ ] `.github/workflows/ci.yml` exists.
- [ ] `.gitignore` covers Python, environments, caches, secrets, and project private paths.

Code quality:

- [ ] `--help` works.
- [ ] `--version` works.
- [ ] Main script compiles.
- [ ] Runtime side effects are isolated behind functions.
- [ ] Errors are actionable.
- [ ] Secrets are never printed.

Testing:

- [ ] Tests exist under `tests/`.
- [ ] Tests mock external side effects.
- [ ] Tests cover success and failure paths.
- [ ] Tests do not require secrets.
- [ ] Tests do not require private data.
- [ ] `python -m pytest` passes.

Quality gates:

- [ ] `ruff check .` passes.
- [ ] `black --check .` passes.
- [ ] `mypy` passes.
- [ ] PowerShell parser checks pass if applicable.
- [ ] CI passes on GitHub.

Docs:

- [ ] `README.md` explains purpose, version, setup, usage, development checks, and troubleshooting.
- [ ] `CHANGE_CONTROL_PACKET.md` exists and is current.
- [ ] `CHANGELOG.md` exists if the project needs release history.
- [ ] `SECURITY.md` exists if the project handles secrets, PII, machine mutation, or hosted service behavior.
- [ ] Docs contain no private values.

Release:

- [ ] Version is consistent across code, README, `pyproject.toml`, and change-control packet.
- [ ] Validation commands were run locally.
- [ ] Git status reviewed before commit.
- [ ] GitHub Actions pass after push.

## Applying This To `x_create_cv_x`

`x_create_cv_x` needs the standard productionization path plus private-data hardening.

Required next steps:

- Keep `data/private/` ignored.
- Keep `private.zip` out of Git.
- Keep real seed scripts out of Git.
- Add fake fixtures for CI.
- Add `pyproject.toml`.
- Add `requirements-dev.txt`.
- Add `.vscode/settings.json`.
- Add `.github/workflows/ci.yml`.
- Add tests for factory commands, JSON writing, zip validation, resume record references, and fake fixture rebuilds.
- Add `CHANGE_CONTROL_PACKET.md`.
- Add `SECURITY.md` because the project handles private CV data.
- Consider `CHANGELOG.md` because this is intended as a showcase project.
- Update README to describe public fake-data flow and private local flow separately.
- Split the current CLI into modules only if tests and future web/API work benefit from it.

Special `x_create_cv_x` checks:

- No real CV values in public files.
- No private archive contents in public tests.
- CI uses fake CV data only.
- Validation with real private data remains local and ignored.
- Public examples demonstrate the model without exposing the real person.
- README clearly says private seed scripts are local-only.

Recommended initial `x_create_cv_x` test plan:

- `test_init_database_creates_master_profile`
- `test_add_user_adds_person_record`
- `test_add_job_adds_job_record`
- `test_create_resume_writes_resume_file`
- `test_add_resume_section_orders_sections`
- `test_add_resume_item_references_master_records`
- `test_validate_against_zip_passes_for_fake_fixture`
- `test_validate_against_zip_reports_mismatch`
- `test_cli_rejects_invalid_json_payload`
- `test_cli_typed_flags_build_expected_payload`

## Standard Done Statement

When a repo is productionized, the closeout should say:

```text
Productionized repo_name with pyproject metadata, pinned dev tooling, strict VS Code settings, GitHub Actions CI, tests, README development/release docs, and a current change-control packet. Local validation passed: pytest, Ruff, Black, mypy, compile, help/version checks, and any project-specific safety checks.
```

If anything could not be done, say exactly what is missing and why.