# Rebrand to CMI Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Complete rebrand of `claude_monitor` package to `cmi` (Claude Monitor Indicators) across all code, configuration, documentation, and deployment files.

**Architecture:** Systematic search-and-replace approach executed in logical phases: (1) package structure, (2) Python imports, (3) configuration, (4) entry points, (5) documentation, (6) CI/CD, (7) cleanup, (8) verification. All changes committed atomically in a single commit.

**Tech Stack:** Python 3.9+, pytest, Pydantic v2, Rich, pyproject.toml-based configuration

---

## Task 1: Create New Package Structure

**Files:**
- Create: `src/cmi/` (copy of `src/claude_monitor/`)
- Keep: `src/claude_monitor/` (temporary, will delete later)

**Step 1: Copy package directory**

```bash
cp -r src/claude_monitor src/cmi
```

**Step 2: Verify structure is intact**

```bash
ls -la src/cmi/
# Expected: __init__.py, _version.py, cli/, core/, data/, monitoring/, ui/, terminal/, utils/
```

**Step 3: Verify files exist**

```bash
find src/cmi -name "*.py" | head -5
# Expected: src/cmi/__init__.py, src/cmi/_version.py, src/cmi/cli/main.py, etc.
```

---

## Task 2: Update Imports in cmi Package

**Files:**
- Modify: All `.py` files in `src/cmi/`

**Step 1: Find all import references**

```bash
grep -r "from claude_monitor" src/cmi/
grep -r "import claude_monitor" src/cmi/
```

**Step 2: Replace all absolute imports**

```bash
find src/cmi -name "*.py" -type f -exec sed -i 's/from claude_monitor/from cmi/g' {} +
find src/cmi -name "*.py" -type f -exec sed -i 's/import claude_monitor/import cmi/g' {} +
```

**Step 3: Verify replacements**

```bash
grep -r "from claude_monitor" src/cmi/
# Expected: (no output - all replaced)
grep -r "import cmi" src/cmi/ | head -3
# Expected: shows updated imports
```

---

## Task 3: Update Imports in Tests

**Files:**
- Modify: All `.py` files in `src/tests/`

**Step 1: Replace test imports**

```bash
find src/tests -name "*.py" -type f -exec sed -i 's/from claude_monitor/from cmi/g' {} +
find src/tests -name "*.py" -type f -exec sed -i 's/import claude_monitor/import cmi/g' {} +
```

**Step 2: Replace test fixture paths**

```bash
find src/tests -name "*.py" -type f -exec sed -i 's/\.claude-monitor/.cmi/g' {} +
find src/tests -name "*.py" -type f -exec sed -i 's/claude-monitor/cmi/g' {} +
```

**Step 3: Verify**

```bash
grep -r "from claude_monitor" src/tests/
# Expected: (no output)
grep -r "\.claude-monitor" src/tests/
# Expected: (no output)
```

---

## Task 4: Update Configuration Paths in Source Code

**Files:**
- Modify: `src/cmi/monitoring/data_manager.py` (config path)
- Modify: `src/cmi/core/settings.py` (config path references)
- Modify: Any other files with hardcoded `~/.claude-monitor` paths

**Step 1: Find all config path references**

```bash
grep -r "claude-monitor" src/cmi/
grep -r "claude_monitor" src/cmi/
```

**Step 2: Replace in data_manager.py**

First, read the file to understand current implementation:
```bash
grep -n "claude-monitor" src/cmi/monitoring/data_manager.py
```

Then replace:
```bash
sed -i 's/\.claude-monitor/.cmi/g' src/cmi/monitoring/data_manager.py
sed -i "s/'claude-monitor'/'cmi'/g" src/cmi/monitoring/data_manager.py
sed -i 's/"claude-monitor"/"cmi"/g' src/cmi/monitoring/data_manager.py
```

**Step 3: Replace in settings.py**

```bash
sed -i 's/\.claude-monitor/.cmi/g' src/cmi/core/settings.py
sed -i "s/'claude-monitor'/'cmi'/g" src/cmi/core/settings.py
sed -i 's/"claude-monitor"/"cmi"/g' src/cmi/core/settings.py
```

**Step 4: Verify all replaced**

```bash
grep -r "\.claude-monitor" src/cmi/
grep -r "claude-monitor" src/cmi/
# Expected: (no output)
```

---

## Task 5: Update pyproject.toml Entry Points

**Files:**
- Modify: `pyproject.toml`

**Step 1: Read relevant section**

```bash
grep -A 10 "\[project.scripts\]" pyproject.toml
```

**Step 2: Update entry points**

Replace the `[project.scripts]` section. The old section has:
```toml
[project.scripts]
claude-monitor = "claude_monitor.cli:main"
cmonitor = "claude_monitor.cli:main"
ccmonitor = "claude_monitor.cli:main"
ccm = "claude_monitor.cli:main"
```

Change to:
```toml
[project.scripts]
cmi = "cmi.cli:main"
```

**Step 3: Update package metadata**

```bash
# Update package name
sed -i 's/name = "claude-monitor"/name = "cmi"/' pyproject.toml

# Verify
grep 'name = "cmi"' pyproject.toml
```

**Step 4: Update project description/keywords if they mention old name**

```bash
grep -n "Claude Monitor" pyproject.toml
# Update as needed - if using "Claude Monitor Indicators" now, update refs
```

---

## Task 6: Update Test Configuration Fixtures

**Files:**
- Modify: `src/tests/conftest.py`

**Step 1: Find config directory mocks**

```bash
grep -n "claude-monitor" src/tests/conftest.py
```

**Step 2: Update fixture to use new path**

Replace any references to `~/.claude-monitor` with `~/.cmi` in the conftest.py fixtures.

```bash
sed -i 's/\.claude-monitor/.cmi/g' src/tests/conftest.py
```

**Step 3: Verify**

```bash
grep "\.cmi" src/tests/conftest.py
# Expected: shows updated paths
```

---

## Task 7: Run Full Test Suite

**Files:**
- Test: All tests in `src/tests/`

**Step 1: Run pytest**

```bash
cd src && python -m pytest -v
```

**Expected output:**
- All tests pass
- No import errors
- No module not found errors for `cmi`

**Step 2: Run with coverage**

```bash
cd src && python -m pytest --cov=cmi --cov-report=term-missing
```

**Expected output:**
- Coverage >= 70%
- All tests pass

**Step 3: If tests fail, debug**

```bash
# Check for any remaining old references
grep -r "claude_monitor" src/
grep -r "claude-monitor" src/cmi/
# Fix any found issues and re-run
```

---

## Task 8: Update README.md

**Files:**
- Modify: `README.md`

**Step 1: Find all references**

```bash
grep -n "claude-monitor\|claude_monitor\|Claude Monitor" README.md | head -20
```

**Step 2: Update main references**

- Installation: `pip install cmi` (was `pip install claude-monitor`)
- Command: `cmi` (was `claude-monitor`, `cmonitor`, etc.)
- Config: `~/.cmi/` (was `~/.claude-monitor/`)
- Package imports: `from cmi import` (was `from claude_monitor import`)

Use sed for bulk replacements:
```bash
sed -i 's/claude-monitor/cmi/g' README.md
sed -i 's/\.claude-monitor/.cmi/g' README.md
sed -i 's/from claude_monitor/from cmi/g' README.md
```

**Step 3: Verify**

```bash
grep "claude-monitor\|claude_monitor" README.md
# Expected: (no output)
grep "cmi" README.md | head -10
# Expected: shows updated references
```

---

## Task 9: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

**Step 1: Find references in CLAUDE.md**

```bash
grep -n "claude-monitor\|claude_monitor\|Claude Monitor\|claude_monitor" CLAUDE.md
```

**Step 2: Update main sections**

- Entry points: `cmi` (was `claude-monitor`, `cmonitor`, `ccmonitor`, `ccm`)
- Config: `~/.cmi/` (was `~/.claude-monitor/`)
- Entry Points section
- Quick start commands
- Examples

Use sed:
```bash
sed -i 's/claude-monitor/cmi/g' CLAUDE.md
sed -i 's/\.claude-monitor/.cmi/g' CLAUDE.md
sed -i 's/from claude_monitor/from cmi/g' CLAUDE.md
sed -i 's/import claude_monitor/import cmi/g' CLAUDE.md
```

**Step 3: Verify**

```bash
grep "claude-monitor\|claude_monitor" CLAUDE.md
# Expected: (no output)
```

---

## Task 10: Update DEVELOPMENT.md (if exists)

**Files:**
- Modify: `DEVELOPMENT.md` (if it exists)

**Step 1: Check if file exists**

```bash
ls -la DEVELOPMENT.md
```

**Step 2: If exists, update references**

```bash
sed -i 's/claude-monitor/cmi/g' DEVELOPMENT.md
sed -i 's/\.claude-monitor/.cmi/g' DEVELOPMENT.md
sed -i 's/from claude_monitor/from cmi/g' DEVELOPMENT.md
```

**Step 3: Verify**

```bash
grep "claude-monitor\|claude_monitor" DEVELOPMENT.md
# Expected: (no output)
```

---

## Task 11: Update CONTRIBUTING.md (if exists)

**Files:**
- Modify: `CONTRIBUTING.md` (if it exists)

**Step 1: Check if file exists**

```bash
ls -la CONTRIBUTING.md
```

**Step 2: If exists, update**

```bash
sed -i 's/claude-monitor/cmi/g' CONTRIBUTING.md
sed -i 's/\.claude-monitor/.cmi/g' CONTRIBUTING.md
sed -i 's/from claude_monitor/from cmi/g' CONTRIBUTING.md
```

---

## Task 12: Update GitHub Workflows

**Files:**
- Modify: `.github/workflows/*.yml` (all workflow files)

**Step 1: Find workflow files**

```bash
ls -la .github/workflows/
```

**Step 2: Update references in workflows**

```bash
find .github/workflows -name "*.yml" -type f -exec sed -i 's/claude-monitor/cmi/g' {} +
find .github/workflows -name "*.yml" -type f -exec sed -i 's/claude_monitor/cmi/g' {} +
find .github/workflows -name "*.yml" -type f -exec sed -i 's/\.claude-monitor/.cmi/g' {} +
```

**Step 3: Check specific files for package references**

```bash
grep -l "claude_monitor\|claude-monitor" .github/workflows/*.yml
# If any found, manually review and update
```

---

## Task 13: Run Code Quality Checks

**Files:**
- Test: All Python files

**Step 1: Run linting**

```bash
uv run ruff check src/
# Expected: PASS (no errors)
```

**Step 2: Run formatting check**

```bash
uv run ruff format --check src/
# If fails, run: uv run ruff format src/
```

**Step 3: Run type checking**

```bash
uv run mypy src/
# Expected: no type errors
```

**Step 4: Run pre-commit hooks**

```bash
uv run pre-commit run --all-files
# Expected: PASS on all hooks
```

---

## Task 14: Verify CLI Works

**Files:**
- Test: CLI entry point

**Step 1: Test help command**

```bash
uv run cmi --help
# Expected: Usage info displays, no import errors
```

**Step 2: Test with a plan**

```bash
uv run cmi --plan pro
# Expected: Works without errors (assuming Claude Code is running)
```

**Step 3: Test alternate plan**

```bash
uv run cmi --plan custom
# Expected: Works without errors
```

---

## Task 15: Remove Old Package Directory

**Files:**
- Delete: `src/claude_monitor/`

**Step 1: Verify new package works**

```bash
python -c "from cmi import __version__; print(__version__)"
# Expected: prints version number
```

**Step 2: Remove old directory**

```bash
rm -rf src/claude_monitor
```

**Step 3: Verify it's gone**

```bash
ls -la src/claude_monitor
# Expected: "No such file or directory"
```

---

## Task 16: Final Verification and Commit

**Files:**
- Verify: All modified files

**Step 1: Review git status**

```bash
git status
```

**Expected:** Shows all modified files and new `src/cmi/` directory, `src/claude_monitor/` deleted

**Step 2: Run full test suite one more time**

```bash
uv run pytest --cov=cmi
# Expected: All tests pass, coverage >= 70%
```

**Step 3: Check for any remaining old references**

```bash
grep -r "claude_monitor" src/ docs/ .github/
grep -r "\.claude-monitor" src/ docs/ .github/
# Expected: (no output except in design doc)
```

**Step 4: Create single atomic commit**

```bash
git add -A
git commit -m "refactor: rebrand claude_monitor to cmi (Claude Monitor Indicators)

- Rename src/claude_monitor/ → src/cmi/
- Update all imports from claude_monitor to cmi
- Update CLI entry point: cmi (single command, no aliases)
- Update config directory: ~/.cmi/ (was ~/.claude-monitor/)
- Update PyPI package name: cmi
- Update all documentation and references
- Update GitHub workflows
- Verified: all tests pass, coverage 70%+, linting/type checking pass"
```

**Step 5: Verify commit**

```bash
git log -1 --stat
# Expected: Shows all files changed in one commit
```

---

## Summary

All changes executed in sequence to:
1. Create new `cmi` package structure
2. Update all imports and references
3. Update configuration and entry points
4. Run comprehensive testing and verification
5. Update all documentation
6. Clean up old structure
7. Create single atomic commit

**Success criteria:**
- ✅ All tests pass (70%+ coverage)
- ✅ All linting/type checking passes
- ✅ CLI command `cmi` works
- ✅ No references to old names remain (except docs/plans design.md)
- ✅ Single atomic commit with all changes
