# Rebrand Design: claude_monitor → cmi (Claude Monitor Indicators)

**Date:** 2026-02-15
**Status:** Approved
**Approach:** Systematic Search-and-Replace, All-at-Once

## Overview

Complete rebrand of the project from `claude_monitor` to `cmi` (Claude Monitor Indicators), affecting package structure, CLI commands, configuration paths, PyPI package name, and all documentation.

## Requirements

- **Python Package Name:** `cmi`
- **CLI Command:** `cmi` (single entry point, no aliases)
- **Config Directory:** `~/.cmi/` (no migration of old data)
- **PyPI Package:** `cmi`
- **GitHub Repo:** Renamed to reflect new identity
- **Execution:** All-at-once atomic commit (not phased)
- **Backwards Compatibility:** None — complete replacement

## Scope of Changes

### 1. Python Package & Modules
- Rename `src/claude_monitor/` → `src/cmi/`
- Update all internal imports: `from claude_monitor import` → `from cmi import`
- Update all relative imports within the package

### 2. Entry Points
- Update `pyproject.toml` scripts section
- Remove: `claude-monitor`, `cmonitor`, `ccmonitor`, `ccm`
- Add: `cmi` pointing to `cmi.cli:main`

### 3. Configuration Paths
- Update hardcoded paths: `~/.claude-monitor/` → `~/.cmi/`
- Update config file references in code
- Update documentation references

### 4. Package Metadata
- Update `pyproject.toml`:
  - `name = "cmi"`
  - `description` (if references old name)
  - `keywords` (if references old name)
  - `urls` (if GitHub repo renamed)

### 5. GitHub Actions & CI
- Update workflow references to new package structure if applicable
- Update any hardcoded paths in CI configurations

### 6. Documentation
- README.md: Installation, usage examples, all command references
- CLAUDE.md: Entry points, development setup, commands
- DEVELOPMENT.md: Any tool invocations
- CONTRIBUTING.md: Any package references

### 7. Tests
- Update `src/tests/conftest.py`: Config path fixtures
- Update individual test files: assertions, mocks, fixtures
- Update any hardcoded references to `claude_monitor` or `claude-monitor`

### 8. Version Management
- After directory rename, update `src/cmi/_version.py` (path reference only, not content)

## Implementation Strategy

### Sequence
1. Create new package structure (copy `src/claude_monitor/` → `src/cmi/`)
2. Update all Python imports in `src/cmi/` and `src/tests/`
3. Update `pyproject.toml` (name, entry points, metadata)
4. Update configuration paths in source code
5. Update test fixtures and assertions
6. Run full test suite (verify passing)
7. Update all documentation files
8. Update GitHub workflows and CI files
9. Remove old `src/claude_monitor/` directory
10. Final verification pass
11. Single atomic commit

### Verification Points

**During each stage:**
- Use `grep -r "claude_monitor" src/` to find missed references
- Use `grep -r "claude-monitor" .` to find config/doc references
- Use `grep -r "Claude Monitor" .` to find description references

**After all changes:**
1. `uv run pytest --cov=cmi --cov-report=html` (70%+ coverage required)
2. `uv run mypy src/` (no type errors)
3. `uv run ruff check .` (linting)
4. `uv run ruff format .` (formatting)
5. `uv run pre-commit run --all-files` (all hooks)
6. Manual CLI test: `uv run cmi --help`
7. Manual CLI test: `uv run cmi --plan pro`

### File Structure Changes

**Before:**
```
src/claude_monitor/
  ├── __init__.py
  ├── _version.py
  ├── cli/
  ├── core/
  ├── data/
  ├── monitoring/
  ├── ui/
  ├── terminal/
  └── utils/
~/.claude-monitor/
  ├── last_used.json
  └── debug.log
```

**After:**
```
src/cmi/
  ├── __init__.py
  ├── _version.py
  ├── cli/
  ├── core/
  ├── data/
  ├── monitoring/
  ├── ui/
  ├── terminal/
  └── utils/
~/.cmi/
  ├── last_used.json
  └── debug.log
```

## Success Criteria

✅ All imports updated and code runs without import errors
✅ Full test suite passes with 70%+ coverage
✅ All linting and type checking passes
✅ CLI command `cmi` works correctly
✅ Documentation reflects new names throughout
✅ Single atomic commit with all changes
✅ Pre-commit hooks pass
✅ Manual testing confirms functionality unchanged

## Breaking Changes

- Old PyPI package `claude-monitor` no longer used
- Old CLI commands (`claude-monitor`, `cmonitor`, etc.) no longer work
- Old config directory `~/.claude-monitor/` no longer used
- Old imports (`from claude_monitor import`) no longer valid
- This is intentional as a complete rebrand

## Next Steps

1. Execute implementation plan (to be created via writing-plans skill)
2. Run full verification suite
3. Create PR with all changes
4. Merge to main branch
5. Deploy new version to PyPI as `cmi`
