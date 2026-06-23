# Tasks: Story 1.1 — Project Scaffolding

## Task 1: Init project with uv
**Status:** pending

Run `uv init --python 3.13 --name worktrace .` then edit `pyproject.toml` with all prod deps and entry point.

**Acceptance criteria:**
- `pyproject.toml` has all 6 prod deps with locked version ranges
- `[project.scripts]` has `worktrace = "worktrace.cli:app"`
- `uv sync` exits 0

---

## Task 2: Add dev dependencies and ruff config
**Status:** pending

Run `uv add --dev "pytest>=9.0.3,<10.0" "pytest-mock>=3.14" "ruff>=0.15.16"` then add ruff config to `pyproject.toml`.

**Acceptance criteria:**
- pytest, pytest-mock, ruff in `[dependency-groups]` dev
- `pyproject.toml` has `[tool.ruff]` with `target-version = "py313"`, `line-length = 88`
- `[tool.ruff.lint]` has `select = ["E", "F", "I"]`
- `uv run ruff check .` exits 0

---

## Task 3: Create package skeleton with stub files
**Status:** pending

Create all stub files respecting ARCH-07 import layering.

Files to create:
```
worktrace/__init__.py              # __version__ = "0.1.0" only
worktrace/schema.py                # pass
worktrace/db.py                    # pass
worktrace/workspace.py             # pass
worktrace/config.py                # pass
worktrace/server.py                # pass
worktrace/cli.py                   # import typer; app = typer.Typer()
worktrace/collectors/__init__.py   # pass
worktrace/collectors/git.py        # pass
worktrace/collectors/spec.py       # pass
tests/__init__.py
tests/conftest.py                  # pass
tests/unit/__init__.py
tests/integration/__init__.py
```

**Acceptance criteria:**
- All 14 files exist
- `uv run ruff check .` exits 0
- `uv run pytest` collects 0 items with no errors

---

## Task 4: Create .gitignore
**Status:** pending

Create `.gitignore` at repo root covering Python, uv/venv, ruff, pytest, OS artifacts.

**Acceptance criteria:**
- `.gitignore` exists with entries: `__pycache__/`, `*.py[cod]`, `*.egg-info/`, `dist/`, `build/`, `.venv/`, `.python-version`, `.ruff_cache/`, `.pytest_cache/`, `.DS_Store`

---

## Task 5: Create GitHub Actions CI workflow
**Status:** pending

Create `.github/workflows/ci.yml`.

**Acceptance criteria:**
- `.github/workflows/ci.yml` exists
- Triggers on `push` and `pull_request`
- Steps: `actions/checkout@v4`, `astral-sh/setup-uv@v4` with `python-version: "3.13"`, `uv sync`, `uv run ruff check .`, `uv run ruff format --check .`, `uv run pytest`
