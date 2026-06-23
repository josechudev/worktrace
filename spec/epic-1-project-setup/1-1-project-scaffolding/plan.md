# Plan: Story 1.1 — Project Scaffolding

## Context

Repo exists but has no Python package yet — only `README.md` and `spec/`. All work happens in repo root `worktrace`.

---

## Step 1: Init project with uv

```bash
uv init --python 3.13 --name worktrace .
```

Then edit `pyproject.toml` — add production deps and entry point:

```toml
[project]
name = "worktrace"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    "fastmcp>=3.4.2,<4.0",
    "typer[all]>=0.26.7,<1.0",
    "pydantic>=2.13.4,<3.0",
    "sqlmodel>=0.0.38,<1.0",
    "rich>=15.0.0,<16.0",
    "platformdirs>=4.10.0,<5.0",
]

[project.scripts]
worktrace = "worktrace.cli:app"
```

Run `uv sync` — verify exit 0.

---

## Step 2: Add dev dependencies + ruff config

```bash
uv add --dev "pytest>=9.0.3,<10.0" "pytest-mock>=3.14" "ruff>=0.15.16"
```

Add to `pyproject.toml`:

```toml
[tool.ruff]
target-version = "py313"
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I"]
```

Verify: `uv run ruff check .` exits 0.

---

## Step 3: Create package skeleton

Create files with minimal stubs (no logic). Import layering rule (ARCH-07):
- `schema.py` — no worktrace imports
- `db.py` — imports schema only
- `collectors/*` — imports schema + config only
- `workspace.py` — imports db only
- `server.py`, `cli.py` — imports workspace + db + config + collectors

Files to create:

```
worktrace/__init__.py          # __version__ = "0.1.0" only
worktrace/schema.py            # pass
worktrace/db.py                # pass
worktrace/workspace.py         # pass
worktrace/config.py            # pass
worktrace/server.py            # pass
worktrace/cli.py               # import typer; app = typer.Typer()
worktrace/collectors/__init__.py
worktrace/collectors/git.py    # pass
worktrace/collectors/spec.py   # pass
tests/__init__.py
tests/conftest.py              # pass
tests/unit/__init__.py
tests/integration/__init__.py
```

Verify: `uv run ruff check .` exits 0, `uv run pytest` collects 0 items (no error).

---

## Step 4: .gitignore

Create `.gitignore` at repo root:

```gitignore
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/

# uv / venv
.venv/
.python-version

# Ruff
.ruff_cache/

# Pytest
.pytest_cache/

# OS
.DS_Store
```

---

## Step 5: CI workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v4
        with:
          python-version: "3.13"
      - run: uv sync
      - run: uv run ruff check .
      - run: uv run ruff format --check .
      - run: uv run pytest
```

---

## Acceptance Checklist

- [ ] `pyproject.toml` has all prod deps + `[project.scripts]`
- [ ] `uv sync` exits 0
- [ ] `uv run pytest` runs (0 items OK)
- [ ] `uv run ruff check .` exits 0
- [ ] All stub files exist per list above
- [ ] `.gitignore` exists with Python/uv/ruff/pytest entries
- [ ] `.github/workflows/ci.yml` exists with correct steps
