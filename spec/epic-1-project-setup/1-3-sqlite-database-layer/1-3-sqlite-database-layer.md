# Story 1.3: SQLite Database Layer

## Status: ready-for-dev

## Story

As a developer,
I want a reliable SQLite database layer with proper concurrency handling and schema versioning,
So that the MCP server and CLI can safely share the same database without corruption or surprise schema drift.

## Acceptance Criteria

- [ ] Module-level `engine` created at import time with WAL mode and `connect_args={"timeout": 30, "check_same_thread": False}` (FR-SCH-12)
- [ ] `create_all()` runs `DB_PATH.parent.mkdir(parents=True, exist_ok=True)` BEFORE `SQLModel.metadata.create_all()` — no FileNotFoundError (FM-01)
- [ ] Fresh DB has `schema_version` table with `version=1`; `get_schema_version(session)` returns 1
- [ ] `check_schema_version(session)` raises `RuntimeError` with message `"Schema version N found, expected M. Run: worktrace db migrate"` when stale (FM-02)
- [ ] All 5 tables exist: `workspaces`, `repos` (with `local_path` + `local_path_canonical`), `repo_states`, `decisions`, `agent_state_overrides` (with `session_id` column) (FM-12)
- [ ] Two rows with same `(workspace_id, repo_name)` but different `session_id` both persist in `agent_state_overrides` (append-only); `get_latest_agent_override()` returns row with most recent `updated_at` (FM-13)
- [ ] ALL `db.py` CRUD functions accept `Session` as first parameter; NONE open sessions internally (ARCH-03)
- [ ] `uv run pytest tests/unit/test_db.py` passes: write+read workspace, write+list decisions, write+get_latest agent_override, schema_version pass + fail scenarios

## Tasks/Subtasks

- [ ] Task 1: SQLModel table models in `db.py`
  - [ ] `Workspace` table: id, workspace_id (unique), ticket_ref, created_at, updated_at
  - [ ] `Repo` table: id, workspace_id (FK), repo_name, local_path TEXT, local_path_canonical TEXT, last_run (datetime | None)
  - [ ] `RepoStateRow` table: workspace_id, repo_name, git_updated_at, agent_updated_at, completed_files (JSON), interfaces_defined (JSON), stubs (JSON), agent_summary, spec_snapshot
  - [ ] `DecisionRow` table: id, workspace_id, repo, source, recorded_at, message
  - [ ] `AgentStateOverride` table: id, workspace_id, repo_name, session_id, summary, updated_at (append-only)
  - [ ] `SchemaVersion` table: id, version (int)

- [ ] Task 2: Engine and DB path setup
  - [ ] `DB_PATH` computed via `platformdirs.user_data_dir("worktrace") / "worktrace.db"` (FR-SCH-08)
  - [ ] Module-level `engine` with WAL mode + busy_timeout (FR-SCH-12)
  - [ ] `create_all()` function: mkdir then SQLModel.metadata.create_all + seed schema_version=1

- [ ] Task 3: CRUD helpers (all accept `Session` as first param)
  - [ ] `get_schema_version(session) -> int`
  - [ ] `check_schema_version(session)` — raises RuntimeError if stale
  - [ ] `create_workspace(session, workspace_id, ticket_ref) -> Workspace`
  - [ ] `get_workspace(session, workspace_id) -> Workspace | None`
  - [ ] `list_workspaces(session) -> list[Workspace]`
  - [ ] `upsert_repo(session, workspace_id, repo_name, local_path, local_path_canonical)`
  - [ ] `upsert_repo_state(session, workspace_id, repo_name, **fields)`
  - [ ] `get_repo_state(session, workspace_id, repo_name) -> RepoStateRow | None`
  - [ ] `add_decision(session, workspace_id, repo, source, message, recorded_at)`
  - [ ] `list_decisions(session, workspace_id) -> list[DecisionRow]`
  - [ ] `append_agent_override(session, workspace_id, repo_name, session_id, summary)`
  - [ ] `get_latest_agent_override(session, workspace_id, repo_name) -> AgentStateOverride | None`

- [ ] Task 4: Unit tests with in-memory SQLite fixture
  - [ ] `tests/unit/test_db.py` with `engine = create_engine("sqlite:///:memory:")` fixture
  - [ ] Test schema_version pass and fail (stale) scenarios
  - [ ] Test workspace create + get round-trip
  - [ ] Test decisions write + list
  - [ ] Test agent_override append-only + get_latest returns most recent
  - [ ] Test FM-01: parent dir creation (use tmp_path)

## Dev Notes

**ARCH-02:** Single SQLModel engine instance created once at `db.py` import time — shared by `server.py` and `cli.py`. Do not create engine elsewhere.

**ARCH-03:** Session lifecycle belongs to the CALLER (CLI command or MCP handler). `db.py` helpers accept `Session` as parameter. If you open a session inside `db.py`, that's a bug.

**FM-12:** `repos` table stores BOTH `local_path` (as-given) AND `local_path_canonical` (`Path.resolve()`). Resolution checks both columns. This prevents symlink/relative-path mismatch.

**FM-13:** `agent_state_overrides` is append-only with `session_id` column. `get_latest_agent_override` uses `ORDER BY updated_at DESC LIMIT 1` — never UPDATE the row.

**WAL config pattern:**
```python
engine = create_engine(
    f"sqlite:///{DB_PATH}",
    connect_args={"timeout": 30, "check_same_thread": False},
)
with engine.connect() as conn:
    conn.execute(text("PRAGMA journal_mode=WAL"))
```

## Dev Agent Record

### Implementation Plan

### Debug Log

### Completion Notes

## File List

## Change Log
