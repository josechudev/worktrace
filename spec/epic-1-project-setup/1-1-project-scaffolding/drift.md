# Drift Report: Story 1-1 Project Scaffolding

**Date:** 2026-06-22
**Drift gap:** 17% (1 partial AC, 1 rogue artifact)
**Reviewed by:** spec-red-team

---

## AC Status

| AC Item | Status |
|---|---|
| `pyproject.toml` with all prod deps + `[project.scripts]` | IMPLEMENTED |
| `uv sync` exits 0, all deps resolvable | IMPLEMENTED (uv.lock present) |
| Dev deps available via `uv run pytest` and `uv run ruff` | IMPLEMENTED |
| All stub files exist (12 files + 2 directories) | PARTIAL — `tests/unit/collectors/` missing; `main.py` at root is rogue artifact |
| `.github/workflows/ci.yml` with correct 4-step sequence | IMPLEMENTED |
| `uv run ruff check .` exits 0 on stubs | IMPLEMENTED |

---

## Blocking Issues

### B1 — `cli.py` stub inconsistency under ruff F401

`plan.md` Step 3 puts real imports (`import typer; app = typer.Typer()`) in `cli.py` while all other stubs are `pass`. `agent-prompt.md` says "stub files contain no logic" — direct contradiction.

Under ruff with `F401` enabled, unused imports in `cli.py` will fail lint. Under `E402`, import order rules may also trigger. Future ruff major versions could make this worse silently.

**Fix:** Make `cli.py` a pure `pass` stub matching all other stubs, OR add `# noqa: F401` and explicitly document `cli.py` as the one exception in `plan.md` and `agent-prompt.md`.

---

### B2 — CI Python version mismatch

`architecture.md` matrix comment specifies Python 3.11 + 3.12. `pyproject.toml` has `requires-python = ">=3.13"`. A CI run on Python 3.12 against this project fails with a resolver error — the gate that story 1-1 is supposed to establish will be broken from day one.

**Fix:** `ci.yml` must target Python 3.13 only. Correct or remove the "3.11 + 3.12" comment in `architecture.md`.

---

## Warnings

### W1 — `uv init` overwrites `main.py`

`uv init` in existing directory creates/overwrites `main.py` with its hello-world scaffold. Plan does not address the pre-existing `main.py`. Current repo has `main.py` at root — a loose importable module not required by any AC.

**Fix:** Add explicit task to delete `main.py` after `uv init`, or note it as a known artifact to remove.

---

### W2 — `tests/unit/collectors/__init__.py` missing from plan

Plan's Step 3 file list omits `tests/unit/collectors/__init__.py`. `architecture.md` specifies this sub-directory. Any later story dropping test files under `tests/unit/collectors/` without the `__init__.py` will fail with import errors.

**Fix:** Add `tests/unit/collectors/__init__.py` to the file list in `plan.md` Step 3.

---

### W3 — `publish.yml` omitted with no deferral note

`architecture.md` specifies `.github/workflows/publish.yml` (tag push → build → publish). Plan creates `.github/workflows/` with only `ci.yml` and no note that `publish.yml` is deferred. Agent following this plan will consider CI work complete.

**Fix:** Add comment in `plan.md` noting `publish.yml` is out of scope for story 1-1 and deferred to a later story.

---

## Nits

### N1 — `ruff` missing upper-bound cap

`uv add --dev "ruff>=0.15.16"` has no `<1.0` cap. All other locked deps in `CLAUDE.md` have upper bounds. A future ruff major version could break lint rules silently.

**Fix:** Use `"ruff>=0.15.16,<1.0"` in `plan.md` and `pyproject.toml`.

---

### N2 — `.python-version` gitignore entry undocumented

`.python-version` is in the gitignore template but plan does not document whether `uv init` generates it or whether it should be committed (for reproducibility) or ignored.

**Fix:** Note explicitly in plan whether to commit or ignore `.python-version`.

---

### N3 — Task ordering diverges between `agent-prompt.md` and `plan.md`

`agent-prompt.md` labels `.gitignore` creation as Task 3 and package skeleton as Task 4. `plan.md` reverses this. Harmless but creates sequencing ambiguity for an agent following either file.

**Fix:** Align task numbering between `agent-prompt.md` and `plan.md`.
