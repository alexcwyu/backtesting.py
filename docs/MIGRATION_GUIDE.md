# Migration Guide: backtesting.py

**Branch**: `feature/ayu_develop`
**Standard**: See `docs/PYTHON_MODERN_STANDARD.md` in the trading workspace root.

## Overview

This project has been modernized on the `feature/ayu_develop` branch to use the 2026 Python tooling stack. When syncing from upstream (default branch), the following changes must be re-applied if upstream overwrites them.

## What Changed

### 1. Build System (pyproject.toml)
- **Build backend**: `hatchling` (was: `hatchling`)
- **PEP 621 metadata**: All project metadata in `[project]` table
- **Dependencies**: Managed by `uv`, lockfile in `uv.lock`

### 2. Removed Legacy Files
The following files were removed (upstream may re-add them on sync):
- None (no legacy files existed at time of migration)

If these reappear after a sync, delete them again. All configuration is in `pyproject.toml`.

### 3. Source Layout
- **Layout**: `src/` layout
- **Package moved**: `backtesting/` -> `src/backtesting/`
- **Import unchanged**: `import backtesting` still works

If upstream adds files to the old location, move them to `src/backtesting/`.

### 4. Tooling Configuration (in pyproject.toml)

#### Ruff (linting + formatting)
```toml
[tool.ruff]
line-length = 100
exclude = ['.git', '.eggs', '__pycache__', 'doc/examples']

[tool.ruff.lint]
ignore = ['UP006', 'UP007', 'UP009', 'N802', 'N806', 'C901', 'B008', 'B011', 'RUF002']
select = ['E', 'W', 'F', 'I', 'UP', 'B', 'SIM', 'C4', 'RUF', 'PERF', 'TC', 'PTH', 'N', 'C', 'T', 'YTT']

[tool.ruff.pep8-naming]
ignore-names = ['l', 'h']
```

Changes from initial migration:
- Moved `select`/`ignore` from `[tool.ruff]` to `[tool.ruff.lint]` (correct ruff v0.14+ structure)
- Added rules: `SIM`, `C4`, `PERF`, `TC`, `PTH` per PYTHON_MODERN_STANDARD

#### Pyright (type checking)
```toml
[tool.pyright]
pythonVersion = "3.13"
typeCheckingMode = "basic"
```

#### Pytest
```toml
[tool.pytest.ini_options]
minversion = "9.0"
addopts = ["-ra", "-q", "--strict-markers", "--import-mode=importlib"]
testpaths = ["tests"]
pythonpath = ["src"]
xfail_strict = true
filterwarnings = ["error"]
```

Changes from initial migration:
- Added `--import-mode=importlib` to addopts
- Added `xfail_strict = true` and `filterwarnings = ["error"]`

### 6. File Reorganization
- `MIGRATION_GUIDE.md` moved from repo root to `docs/MIGRATION_GUIDE.md`

### 5. Python Version
- `.python-version` set to `3.13`
- `requires-python = ">=3.13"` in pyproject.toml

## After Upstream Sync Checklist

When merging upstream changes into `feature/ayu_develop`:

1. **Delete re-added legacy files**: `setup.py`, `setup.cfg`, `requirements.txt`, `MANIFEST.in`, `poetry.lock`
2. **Check pyproject.toml**: Upstream may modify `[project]` metadata (version bumps, new deps). Merge those changes but keep `[build-system]`, `[tool.ruff]`, `[tool.pyright]`, `[tool.pytest]` sections intact.
3. **Check source layout**: If upstream adds new modules to the old path, move them to `src/backtesting/`.
4. **Re-lock**: Run `uv lock` to update `uv.lock` with any new/changed dependencies.
5. **Verify**: Run `uv sync && uv run python -c "import backtesting" && uv run pytest` (if tests exist).

## Quick Commands

```bash
uv sync                                    # Install all deps
uv run python -c "import backtesting"      # Verify import
uv run pytest                              # Run tests
uv run ruff check .                        # Lint
uv run ruff format .                       # Format
uv lock                                    # Re-generate lockfile
```
