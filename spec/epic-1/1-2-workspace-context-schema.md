# Story 1.2: Workspace Context Schema

## Status: ready-for-dev

## Story

As a coding agent,
I want well-typed Pydantic v2 models for workspace state,
So that I receive structured, validated data from MCP tools I can rely on without parsing surprises.

## Acceptance Criteria

- [ ] `from worktrace.schema import Stub, Decision, RepoState, WorkspaceContext` imports with no errors
- [ ] `Stub` validates with repo, file_path, symbol, stub_type, context; `.model_dump()` returns all 5 fields; `stub_type` accepts only `"TODO"`, `"FIXME"`, `"pass"`, `"NotImplementedError"`
- [ ] `Decision` source field accepts only `"agent"` or `"git_commit"`; `recorded_at` is datetime
- [ ] `RepoState` has `git_updated_at: datetime | None`, `agent_updated_at: datetime | None`, `agent_summary: str | None`, `spec_snapshot: str | None`
- [ ] `WorkspaceContext.model_dump()` returns dict with: workspace_id, ticket_ref, generated_at, repos, decisions, repo_states, status
- [ ] `WorkspaceContext.model_json_schema()` returns schema suitable as MCP tool documentation
- [ ] `schema.py` has ZERO imports from any other `worktrace.*` module (leaf node — ARCH-07)
- [ ] `uv run pytest tests/unit/test_schema.py` passes: model instantiation, field validation, model_dump output, model_json_schema shape

## Tasks/Subtasks

- [ ] Task 1: Implement `Stub` model
  - [ ] Pydantic v2 `BaseModel` with fields: repo (str), file_path (str), symbol (str), stub_type (Literal["TODO","FIXME","pass","NotImplementedError"]), context (str)
  - [ ] Verify `.model_dump()` round-trips correctly

- [ ] Task 2: Implement `Decision` model
  - [ ] Fields: workspace_id (str), repo (str), source (Literal["agent","git_commit"]), recorded_at (datetime), message (str)

- [ ] Task 3: Implement `RepoState` model
  - [ ] Fields: repo_name (str), git_updated_at (datetime | None), agent_updated_at (datetime | None), completed_files (list[str]), interfaces_defined (list[str]), stubs (list[Stub]), agent_summary (str | None), spec_snapshot (str | None)

- [ ] Task 4: Implement `WorkspaceContext` model
  - [ ] Fields: workspace_id (str), ticket_ref (str | None), generated_at (datetime), repos (list[str]), decisions (list[Decision]), repo_states (dict[str, RepoState]), status (Literal["active","empty"])
  - [ ] Verify `model_json_schema()` is usable as MCP tool docs

- [ ] Task 5: Write unit tests
  - [ ] `tests/unit/test_schema.py`: instantiation for all 4 models, field validation rejections (invalid stub_type, invalid source), model_dump shapes, model_json_schema not empty

## Dev Notes

**Leaf node rule (ARCH-07):** `schema.py` must import ONLY from stdlib and pydantic. No `from worktrace.xxx import ...` anywhere in this file.

**FR-SCH-11:** `WorkspaceContext.model_json_schema()` is the canonical schema for MCP tool documentation — must be human-readable.

**FR-MCP-11:** All MCP tools return `dict` via `.model_dump()` — these models must produce clean dicts. Test `model_dump()` explicitly.

**status field:** Two valid values only — `"active"` (workspace has data) and `"empty"` (no sessions logged yet). The `"empty"` state is returned by `get_workspace_context` on unknown workspace (FR-MCP-09).

## Dev Agent Record

### Implementation Plan

### Debug Log

### Completion Notes

## File List

## Change Log
