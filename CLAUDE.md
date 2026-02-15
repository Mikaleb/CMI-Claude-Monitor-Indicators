# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Claude Code Usage Monitor** is a real-time terminal monitoring tool for tracking Claude AI token usage with advanced analytics, machine learning-based predictions, and a Rich UI. It monitors token consumption, burn rate, cost analysis, and provides intelligent predictions about session limits.

- **Current Version**: 3.1.0
- **Python**: 3.9+
- **Key Tech**: Rich (terminal UI), Pydantic (configuration), NumPy (analytics)
- **Package Name**: `cmi` (on PyPI)
- **Entry Points**: `cmi`

## Quick Start Commands

### Development Setup
```bash
# Clone and navigate
git clone https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor.git
cd Claude-Code-Usage-Monitor

# Install with UV (recommended)
uv sync --extra dev

# Install pre-commit hooks
uv run pre-commit install

# Activate environment (if using venv)
source venv/bin/activate  # Linux/Mac
```

### Common Development Tasks
```bash
# Run tests
uv run pytest
# or
cd src && python -m pytest

# Run tests with coverage report
uv run pytest --cov=cmi --cov-report=html

# Run specific test
uv run pytest src/tests/test_cli_main.py -v

# Lint and format code
uv run ruff check .           # Check linting
uv run ruff format .          # Auto-format code
uv run mypy src/             # Type check

# Pre-commit check (all hooks)
uv run pre-commit run --all-files

# Run the tool locally
uv run cmi --help
python -m cmi --plan pro
```

### Code Quality Standards
- **Line Length**: 88 characters (Black style)
- **Linting**: Ruff with E, W, F, I rules
- **Type Checking**: MyPy with strict mode
- **Coverage Requirement**: 70% minimum for tests
- **Format**: Black + isort for imports
- **Pre-commit**: Ruff (lint + format), trailing-whitespace, end-of-file-fixer, check-yaml, check-toml, mixed-line-ending

## Architecture Overview

### Modular Design (SRP Compliance)
The codebase is organized in seven main layers with clear separation of concerns:

```
src/cmi/
├── cli/                  # Entry point & CLI argument parsing
├── core/                 # Core models, settings, plans, P90 calculations
├── data/                 # Data reading, analysis, aggregation
├── monitoring/           # Orchestrator, session monitor, data manager
├── ui/                   # Display controller, layouts, components, progress bars
├── terminal/             # Theme system, terminal manager
└── utils/                # Utilities (formatting, timezone, time, models)
```

### Data Flow
```
Claude Config Files
  → DataManager (cache + file I/O)
  → SessionMonitor (5-hour window tracking)
  → Analysis Engine (P90 calculator, burn rate)
  → UI Components (progress bars, tables, layouts)
  → Terminal Display (with Rich framework)
```

### Key Components

**MonitoringOrchestrator** (`monitoring/orchestrator.py`): Central control hub
- Manages data manager and session monitor
- Runs background monitoring thread with configurable intervals (1-60 seconds)
- Callback-driven architecture for UI updates
- Thread-safe with proper event synchronization

**SessionMonitor** (`monitoring/session_monitor.py`): Real-time tracking
- Monitors 5-hour rolling session windows
- Tracks active sessions and their token usage
- Detects session boundaries and overlaps

**DataManager** (`monitoring/data_manager.py`): Data access layer
- Manages caching (TTL-based, default 5 seconds)
- Handles file I/O for Claude configuration
- Atomic file operations for safety

**Settings** (`core/settings.py`): Configuration management
- Pydantic-based with CLI parsing
- Automatic parameter persistence (saved to `~/.cmi/last_used.json`)
- Plan validation and token limit calculation

**P90Calculator** (`core/p90_calculator.py`): Machine learning analytics
- 90th percentile analysis for intelligent limit detection
- Historical pattern recognition from last 192 hours (8 days)
- Statistical confidence scoring

**Plans** (`core/plans.py`): Token limit definitions
- Pro: ~19,000 tokens
- Max5: ~88,000 tokens
- Max20: ~220,000 tokens
- Custom: P90-based auto-detection (default)

### UI Architecture
- **DisplayController** (`ui/display_controller.py`): Orchestrates display updates
- **Layouts** (`ui/layouts.py`): Responsive layout management
- **ProgressBars** (`ui/progress_bars.py`): WCAG-compliant visual indicators
- **TableViews** (`ui/table_views.py`): Daily/monthly aggregated statistics
- **Themes** (`terminal/themes.py`): Auto-detecting light/dark/classic themes with scientific color schemes

## Testing

### Test Structure
- **Location**: `src/tests/`
- **Pattern**: `test_*.py` files with pytest fixtures in `conftest.py`
- **Coverage**: 70% minimum enforced (target: 80%+)
- **Test Types**: Unit, integration, benchmark tests marked with pytest markers

### Key Test Files
- `test_cli_main.py`: CLI argument parsing and entry point
- `test_settings.py`: Configuration and parameter persistence
- `test_monitoring_orchestrator.py`: Orchestrator threading and callbacks
- `test_analysis.py`: P90 calculations and burn rate math
- `test_session_analyzer.py`: Session detection and overlap logic
- `test_data_reader.py`: File I/O and cache operations

### Running Tests
```bash
# All tests
uv run pytest

# With coverage
uv run pytest --cov=cmi

# Specific test file
uv run pytest src/tests/test_cli_main.py -v

# With markers
uv run pytest -m "not integration"
```

## Key Patterns & Standards

### Configuration & Settings
- Use Pydantic v2 for all config classes
- CLI args automatically parsed via `BaseSettings.model_validate()`
- Always provide defaults for optional parameters
- Validate input in field validators

### Error Handling
- Import and use `report_error()` from `cmi.error_handling`
- Catch exceptions at component boundaries, not internally
- Log with contextual information before reporting
- Optional Sentry integration for production monitoring

### Threading & Async
- Use `threading.Thread` with daemon=True for background tasks
- Always provide `stop()` method to cleanly shutdown threads
- Use `threading.Event()` for synchronization
- Use callbacks for inter-component communication

### Type Annotations
- All function signatures must have type hints
- Use `Optional[T]` not `T | None` (Python 3.9 compatibility)
- Document complex types with comments
- Export types from `__init__.py` for public APIs

### Data Persistence
- Use `LastUsedParams` class for configuration saving
- Always use atomic file operations (write to temp, then rename)
- Handle missing/corrupted files gracefully with defaults
- Config stored in `~/.cmi/last_used.json`

## Important Files & Patterns

### Entry Points
- **`src/cmi/__main__.py`**: Module execution entry point
- **`src/cmi/cli/main.py`**: CLI bootstrap and orchestrator setup
- **`src/cmi/cli/bootstrap.py`**: Argument parsing and initial setup

### Dependency Locations
- **`pyproject.toml`**: All dependencies, build config, tool configuration
- **Version**: `src/cmi/_version.py` (auto-updated by CI)
- **Tests**: Require `pytest>=8.0.0`, `pytest-cov>=6.0.0`, and related plugins

### CI/CD
- **`.github/workflows/test.yml`**: Matrix testing on Python 3.9-3.13
- **`.github/workflows/lint.yml`**: Ruff and pre-commit checks
- **`.github/workflows/release.yml`**: PyPI publishing with trusted OIDC
- **`.github/workflows/version-bump.yml`**: Automated version management
- **`.pre-commit-config.yaml`**: Local hook configuration

## Development Conventions

### Naming Conventions
- Variables: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Private methods: `_leading_underscore`

### Import Organization
- Standard library imports first
- Third-party imports second (sorted alphabetically)
- Local imports third
- Use absolute imports from `cmi` package

### Documentation
- Use docstrings for all public functions/classes (Google style)
- Include type hints in docstrings for complex parameters
- Add examples in README/DEVELOPMENT.md, not in code comments
- Keep comments minimal - code should be self-documenting

### Commit Messages
- Format: `Add: feature`, `Fix: bug`, `Update: enhancement`, `Refactor: code`
- Keep messages descriptive but concise
- Reference issues with `#123` when applicable
- Never commit generated files or dependencies

## Real-time Features

### Refresh Rates
- Data refresh: 1-60 seconds (configurable, default 10)
- Display refresh: 0.1-20 Hz (default 0.75)
- Cache TTL: 5 seconds default in DataManager

### Session Tracking
- 5-hour rolling windows for Claude Code sessions
- Automatic session overlap detection
- Real-time token consumption tracking within active sessions

### Analytics
- Burn rate calculated from last 1 hour of data
- P90 percentile from last 8 days (192 hours)
- Cost tracking with model-specific pricing
- Cache token support (creation + read tokens)

## Troubleshooting

### Common Issues
- **"No active session found"**: Launch Claude Code and send at least two messages first
- **"Command not found"**: Ensure `uv run` prefix or proper venv activation
- **Tests fail with import errors**: Run from project root, use `cd src && pytest` alternatively

### Debug Mode
```bash
# Enable debug logging
uv run cmi --debug

# Log to file
uv run cmi --log-file ~/.cmi/debug.log --log-level DEBUG
```

## Resources

- **GitHub**: https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor
- **PyPI**: https://pypi.org/project/cmi/
- **Issues**: Use GitHub Issues for bugs and feature requests
- **Documentation**: README.md for user guide, DEVELOPMENT.md for roadmap, CONTRIBUTING.md for contributor guide
