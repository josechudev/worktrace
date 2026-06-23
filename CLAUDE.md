# WorkTrace — Claude Instructions

## Project

WorkTrace is a persistent workspace memory server for coding agents. Core product: FastMCP stdio MCP server. All other features (git collector, spec collector, CLI) exist to populate data it serves.

## Docs Location

All planning artifacts, architecture decisions, PRD, and epics live at:

```
../worktrace-docs/
  docs/planning-artifacts/
    architecture.md   ← architecture decisions (all 8 steps, complete)
    epics.md          ← epic/story breakdown with full FR inventory
  prd-draft.md        ← product requirements
  project-brief.md    ← project brief
```

Read these before implementing any story. Architecture constraints are binding.

## Spec Location

Stories live in `spec/` within this repo:

```
spec/
  epic-1-project-setup/
    1-1-project-scaffolding/
    1-2-workspace-context-schema/
    1-3-sqlite-database-layer/
    1-4-config-and-workspace-resolution/
    1-5-mcp-server-all-6-tools/
    1-6-cli-init-and-status/
```

Each story folder contains:
- `<story-id>.md` — acceptance criteria, tasks, dev notes
- `plan.md` — implementation plan (if created)
- `agent-prompt.md` — agent instructions (if created)

## Stack (locked — do not deviate)

| Layer | Technology |
|---|---|
| Language | Python 3.13+ / uv |
| MCP server | FastMCP (sync handlers) |
| CLI | Typer + Rich |
| Schema | Pydantic v2 |
| ORM / Storage | SQLModel + SQLite (WAL, busy_timeout=30s) |
| Storage path | `platformdirs.user_data_dir("worktrace")` |
| Git ops | `subprocess.run(["git", ...])` — no gitpython |
| Terminal output | Rich (not for data/export commands) |

## Key Constraints

**Tooling:**
- All scripts via `uv run`. Never bare `python` or `pip`.
- `uv sync` for dependency resolution.

**Import layering (ARCH-07) — enforced from day one:**
```
schema.py          ← no worktrace imports
db.py              ← schema only
collectors/*       ← schema + config only
workspace.py       ← db only
server.py, cli.py  ← workspace + db + config + collectors
```

**Version pins (locked):**
```
fastmcp>=3.4.2,<4.0
typer[all]>=0.26.7,<1.0
pydantic>=2.13.4,<3.0
sqlmodel>=0.0.38,<1.0
rich>=15.0.0,<16.0
platformdirs>=4.10.0,<5.0
pytest>=9.0.3,<10.0
pytest-mock>=3.14
ruff>=0.15.16
```

**MCP tools:** All 6 return `dict` via `.model_dump()`. Handlers wrapped in `try/except` — exceptions return `{"status": "error", "message": str(e)}`. Never return Pydantic model directly.

**SQLite:** WAL mode + `busy_timeout=30s` on all connections. `schema_version` table from day one.

**Performance targets:** MCP tool response <200ms. `worktrace log` <30s for 1000 commits across 5 repos.

## Story Workflow

For each story:
1. Read story spec (`<story-id>.md`) + `plan.md` if present.
2. Check architecture doc for relevant constraints.
3. Plan before implementing — state files to create/modify and commands to run.
4. Verify acceptance criteria after each task.
5. Update `Dev Agent Record` section in story spec with notes.
