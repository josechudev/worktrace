# Story 1.1: Project Scaffolding and Package Structure

## Status: ready-for-dev

## Story

As a developer,
I want a properly scaffolded WorkTrace Python package with locked dependencies and CI,
So that every subsequent implementation story has a consistent, reproducible foundation to build on.

## Acceptance Criteria

- [ ] `pyproject.toml` exists with `[project.scripts] worktrace = "worktrace.cli:app"` and all production deps pinned: `fastmcp>=3.4.2,<4.0`, `typer[all]>=0.26.7,<1.0`, `pydantic>=2.13.4,<3.0`, `sqlmodel>=0.0.38,<1.0`, `rich>=15.0.0,<16.0`, `platformdirs>=4.10.0,<5.0`
- [ ] `uv sync` completes with exit code 0 and all production deps resolvable without conflicts
- [ ] Dev deps added: `pytest>=9.0.3,<10.0`, `pytest-mock>=3.14`, `ruff>=0.15.16` — available via `uv run pytest` and `uv run ruff`
- [ ] Package skeleton with all stub files: `worktrace/__init__.py` (version string only), `worktrace/schema.py`, `worktrace/db.py`, `worktrace/workspace.py`, `worktrace/config.py`, `worktrace/server.py`, `worktrace/cli.py`, `worktrace/collectors/__init__.py`, `worktrace/collectors/git.py`, `worktrace/collectors/spec.py`, `tests/conftest.py`, `tests/unit/`, `tests/integration/`
- [ ] `.github/workflows/ci.yml` created; workflow triggers on push/PR with steps: `uv sync` → `ruff check .` → `ruff format --check .` → `pytest`
- [ ] `uv run ruff check .` exits 0 on scaffolded stubs

## Tasks/Subtasks

- [ ] Task 1: Initialize project with uv
  - [ ] Run `uv init worktrace --python 3.13` (or scaffold manually if already in repo)
  - [ ] Add all production deps to `pyproject.toml` with correct version pins
  - [ ] Add `[project.scripts]` entry pointing to `worktrace.cli:app`
  - [ ] Run `uv sync` and verify exit 0

- [ ] Task 2: Add dev dependencies
  - [ ] Add pytest, pytest-mock, ruff as dev deps
  - [ ] Add ruff config section in `pyproject.toml`
  - [ ] Verify `uv run ruff check .` exits 0

- [ ] Task 3: Create package skeleton
  - [ ] Create `worktrace/` package with all stub files (no logic, just `pass` or bare imports)
  - [ ] Create `worktrace/collectors/` subpackage with stubs
  - [ ] Create `tests/conftest.py`, `tests/unit/` and `tests/integration/` directories with `__init__.py`

- [ ] Task 4: CI workflow
  - [ ] Create `.github/workflows/ci.yml` with uv sync → ruff check → ruff format check → pytest steps

## Dev Notes

**Architecture constraints (ARCH-07):** Import layering must be established from day one:
`schema.py` ← no worktrace imports; `db.py` ← schema only; `collectors/*` ← schema + config only; `workspace.py` ← db only; `server.py` + `cli.py` ← workspace + db + config + collectors

**Version pins are locked** — do not deviate from ranges in pyproject.toml. These are tested together.

**All scripts run via `uv run`**, not system Python. Never use bare `python` or `pip`.

**ARCH-09:** Project initialization (`uv init worktrace --python 3.13` + all deps) is the first implementation story.

## Dev Agent Record

### Implementation Plan

### Debug Log

### Completion Notes

## File List

## Change Log
