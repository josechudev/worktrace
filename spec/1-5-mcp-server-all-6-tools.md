# Story 1.5: MCP Server with All 6 Tools

## Status: ready-for-dev

## Story

As a coding agent (Claude Code, Cursor, Codex, or any MCP-compatible agent),
I want to connect to `worktrace serve` via stdio and call workspace context tools,
So that I start every session with full cross-repo context — who did what, what's stubbed, what decisions were made — without the developer re-explaining.

## Acceptance Criteria

- [ ] `MCPTestClient(mcp).call_tool("get_workspace_context", {"workspace_id": "auth-overhaul"})` returns dict matching `WorkspaceContext.model_dump()` with all repos and state; response <200ms (FR-MCP-08)
- [ ] `get_workspace_context("nonexistent-ws")` returns exactly: `{"workspace_id": "nonexistent-ws", "status": "empty", "message": "No sessions logged yet. Run worktrace log or call update_repo_state after a session.", "repos": []}` — never `{}`, never exception (FR-MCP-09)
- [ ] `list_workspaces()` returns list of dicts each with `workspace_id`, `ticket_ref` (str or null), `updated_at`
- [ ] `get_repo_state("ws", "api")` returns `RepoState.model_dump()` for `api` including both `agent_summary` and git-derived fields
- [ ] `get_decisions("ws")` returns list of `Decision.model_dump()` dicts
- [ ] `get_stubs("ws")` returns list of `Stub.model_dump()` dicts across all repos
- [ ] `update_repo_state("ws", "api", "implemented JWT login")` returns `{"status": "ok"}`; subsequent `get_workspace_context("ws")` shows `agent_summary: "implemented JWT login"` with `agent_updated_at` set
- [ ] Any tool handler exception returns `{"status": "error", "message": str(e)}` — NEVER propagates (FR-MCP-10)
- [ ] ALL 6 tool handlers return `dict` — NO Pydantic model returned directly (FR-MCP-11)
- [ ] `worktrace serve --dry-run` prints all 6 tool names and signatures; server does NOT start (FR-MCP-12)
- [ ] `uv run pytest tests/integration/test_mcp_tools.py` passes: all 6 tools happy path + empty workspace onboarding + error→error dict + update→query round-trip

## Tasks/Subtasks

- [ ] Task 1: FastMCP app setup in `server.py`
  - [ ] Create FastMCP app instance
  - [ ] All handlers as `sync def` (ARCH-01 — FastMCP v3 supports sync as first-class; no async/thread executors)
  - [ ] Import only from `worktrace.workspace`, `worktrace.db`, `worktrace.config`, `worktrace.schema` (ARCH-07)

- [ ] Task 2: Implement `get_workspace_context` tool
  - [ ] Query workspace + all repos + repo_states + decisions + stubs + latest agent overrides
  - [ ] Assemble `WorkspaceContext` → return `.model_dump()`
  - [ ] Unknown workspace_id or empty → return onboarding dict (exact message string from FR-MCP-09)
  - [ ] Wrap in try/except → `{"status": "error", "message": str(e)}` on failure

- [ ] Task 3: Implement `get_repo_state` tool
  - [ ] Query single repo's latest state + latest agent override
  - [ ] Return `RepoState.model_dump()`
  - [ ] try/except wrapper

- [ ] Task 4: Implement `get_decisions` tool
  - [ ] Query all decisions for workspace_id
  - [ ] Return `[d.model_dump() for d in decisions]`
  - [ ] try/except wrapper

- [ ] Task 5: Implement `get_stubs` tool
  - [ ] Query stubs across all repo_states for workspace_id
  - [ ] Return flat list of `Stub.model_dump()` dicts
  - [ ] try/except wrapper

- [ ] Task 6: Implement `list_workspaces` tool
  - [ ] Query all workspaces
  - [ ] Return list of dicts with workspace_id, ticket_ref, updated_at
  - [ ] try/except wrapper

- [ ] Task 7: Implement `update_repo_state` tool
  - [ ] Signature: `update_repo_state(workspace_id: str, repo_name: str, summary: str) -> dict`
  - [ ] Append to `agent_state_overrides` table via `db.append_agent_override()`
  - [ ] SQLite `busy_timeout=30s` handles concurrent `worktrace log` writes (FR-MCP-07)
  - [ ] Return `{"status": "ok"}`
  - [ ] try/except wrapper

- [ ] Task 8: `--dry-run` support in serve command
  - [ ] Print all 6 tool names + parameter schemas to stdout
  - [ ] Exit without starting server (FR-MCP-12)

- [ ] Task 9: Integration tests
  - [ ] `tests/integration/test_mcp_tools.py` using `MCPTestClient` from fastmcp
  - [ ] All 6 tools happy path with seeded in-memory DB
  - [ ] Empty workspace onboarding response exact match
  - [ ] Error injection → verify `{"status": "error", ...}` returned not raised
  - [ ] `update_repo_state` then `get_workspace_context` round-trip

## Dev Notes

**ARCH-01:** ALL tool handlers MUST be `sync def`. FastMCP v3 handles sync as first-class. Do not use `async def`, `asyncio.run`, or `ThreadPoolExecutor` in any handler.

**FR-MCP-09 exact response (do not paraphrase):**
```python
{
    "workspace_id": workspace_id,
    "status": "empty",
    "message": "No sessions logged yet. Run worktrace log or call update_repo_state after a session.",
    "repos": []
}
```

**FR-MCP-10/11:** Every handler must:
1. Be wrapped in `try/except Exception as e: return {"status": "error", "message": str(e)}`
2. Never return a Pydantic model — always `.model_dump()` or a literal dict

**FR-MCP-08:** Response time <200ms for typical workloads (≤10 repos, ≤1000 commits). Use indexed queries; avoid N+1 patterns.

**MCPTestClient import:** `from fastmcp.testing import MCPTestClient` (fastmcp ≥ 3.4.2)

**Session management (ARCH-03):** Open session in each handler; pass to db.py helpers. Do not let db.py open sessions.

## Dev Agent Record

### Implementation Plan

### Debug Log

### Completion Notes

## File List

## Change Log
