# Story 1.4: Configuration and Workspace Resolution

## Status: ready-for-dev

## Story

As a developer,
I want WorkTrace to find my active workspace automatically from my current directory and load config cleanly,
So that I never have to specify `--workspace` on routine commands.

## Acceptance Criteria

- [ ] `settings.db_path` resolves via `platformdirs.user_data_dir("worktrace") / "worktrace.db"`; NO `os.environ` access exists outside `config.py` (FR-CLI-19)
- [ ] `worktrace.toml` with `[log] default_since_days = 14` overrides hardcoded default of 30
- [ ] Unknown key in `worktrace.toml` (e.g. `[future_feature] x = 1`) logs warning but does NOT error (ARCH-05 forward compat)
- [ ] Invalid TOML syntax raises loud error pointing to file path — NOT silent fallback (FM-11)
- [ ] `resolve_workspace(cwd, session)` on repo registered to exactly one workspace returns that workspace_id (via git toplevel → Path.resolve() → repos table lookup on both columns)
- [ ] `resolve_workspace` on repo registered in 2+ workspaces (no override) raises `ValueError`: `"Repo registered in multiple workspaces: [...]. Use --workspace to specify."`
- [ ] `resolve_workspace` on dir not matching any repo raises `ValueError` directing user to `--workspace`
- [ ] `resolve_workspace(cwd, session, workspace_id="auth-overhaul")` returns `"auth-overhaul"` without CWD git lookup
- [ ] `uv run pytest tests/unit/test_config.py tests/unit/test_workspace.py` passes all 4 resolve scenarios + toml scenarios

## Tasks/Subtasks

- [ ] Task 1: Implement `config.py`
  - [ ] Define `Settings` dataclass or Pydantic model with: `db_path`, `log.default_since_days` (default 30)
  - [ ] Load from `worktrace.toml` if present (search CWD upward or fixed location)
  - [ ] toml parse error → loud `ValueError`/`SystemExit` with file path in message (FM-11)
  - [ ] Unknown toml keys → `warnings.warn(...)` only, no error (ARCH-05)
  - [ ] Module-level `settings` singleton (no `os.environ` in callers — FR-CLI-19)

- [ ] Task 2: Implement `workspace.py:resolve_workspace()`
  - [ ] Signature: `resolve_workspace(cwd: Path, session: Session, workspace_id: str | None = None) -> str`
  - [ ] If `workspace_id` provided: return it directly (no git lookup)
  - [ ] Otherwise: `git rev-parse --show-toplevel` from `cwd` → `Path.resolve()` → query `repos` table WHERE `local_path = val OR local_path_canonical = val`
  - [ ] Exactly one match: return workspace_id
  - [ ] Zero matches: raise `ValueError` with `--workspace` hint
  - [ ] 2+ matches: raise `ValueError` listing workspace names + `--workspace` hint

- [ ] Task 3: Unit tests
  - [ ] `tests/unit/test_config.py`: toml load override, unknown key warning, invalid TOML error, env passthrough via config only
  - [ ] `tests/unit/test_workspace.py`: 4 resolve scenarios (mocker.patch `run_git` to return fake toplevel; use in-memory DB with seeded repos)

## Dev Notes

**ARCH-04:** CWD → workspace resolution uses `git rev-parse --show-toplevel` then `Path.resolve()` for canonical form. Query `repos` table on BOTH `local_path` AND `local_path_canonical` columns. The dual-column check is required by FM-12.

**ARCH-05:** Config resolution order: `worktrace.toml` > env vars (via config.py) > CLI flags > hardcoded defaults. Do NOT let toml parse errors silently fall through — FM-11 is a hard requirement.

**FR-CLI-19:** The strict rule is: NO `os.environ` access anywhere EXCEPT `config.py`. This is verified by grep. If you access env vars in cli.py or server.py, that's a bug.

**Workspace.py import:** Only imports from `db.py` (ARCH-07). Must not import schema, collectors, or server.

**git binary:** `resolve_workspace` uses `run_git` subprocess. The git binary check (FM-03) lives in `worktrace log` and `worktrace serve` startup, not here. `resolve_workspace` can let CalledProcessError propagate — callers handle it.

## Dev Agent Record

### Implementation Plan

### Debug Log

### Completion Notes

## File List

## Change Log
