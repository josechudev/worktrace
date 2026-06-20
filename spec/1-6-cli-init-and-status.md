# Story 1.6: CLI Workspace Init and Status Commands

## Status: ready-for-dev

## Story

As a developer,
I want to initialize a WorkTrace workspace and check its status from the terminal,
So that I can register my repos once and immediately start the MCP server for any agent to connect.

## Acceptance Criteria

- [ ] `worktrace init auth-overhaul --repos api,frontend,shared` creates workspace in SQLite; all 3 repos registered with `local_path` and `local_path_canonical`; Rich output shows `✔ Workspace created: auth-overhaul` and `✔ Registered repos: api, frontend, shared`
- [ ] Init output includes tip: `"Tip: add --install-hooks to auto-run \`worktrace log\` after every commit in registered repos."` (ARCH-08)
- [ ] `worktrace init auth-overhaul --repos api --ticket PROJ-123` stores `ticket_ref="PROJ-123"`; workspace_id unchanged
- [ ] `worktrace init auth-overhaul --repos /nonexistent/path,api` prints Rich error for invalid path; valid repo still registered; exit code 1 (FR-CLI-04)
- [ ] `worktrace init auth-overhaul --repos api` run twice: exit code 0; no data loss; idempotent (FR-CLI-05)
- [ ] `worktrace init auth-overhaul --repos new-repo` on existing workspace: adds new-repo; pre-existing repos and state untouched (FR-CLI-06)
- [ ] `worktrace status` shows Rich table: workspace_id, ticket_ref, repo count, last-updated; one row per workspace (FR-CLI-07)
- [ ] `worktrace serve` starts FastMCP stdio server; process listens on stdin/stdout; stays alive until killed (FR-MCP-01)
- [ ] `worktrace --help` lists init, serve, status with descriptions; exit code 0 (FR-CLI-16)
- [ ] CLI errors use Rich error panel + `raise typer.Exit(code=1)` — no `sys.exit()`, no bare `raise SystemExit()` (FR-CLI-18)
- [ ] All flags use kebab-case; all commands support `--help` (FR-CLI-16)
- [ ] All env var / config access via `config.py` only — no `os.environ` in `cli.py` (FR-CLI-19)
- [ ] `uv run pytest tests/integration/test_cli.py -k "init or status or serve"` passes via `typer.testing.CliRunner`

## Tasks/Subtasks

- [ ] Task 1: Typer app skeleton in `cli.py`
  - [ ] Create `app = typer.Typer()` with name "worktrace"
  - [ ] Import only from `worktrace.workspace`, `worktrace.db`, `worktrace.config` (ARCH-07)
  - [ ] Rich console for all user output; `print()` reserved for data-only commands (FR-CLI-17)
  - [ ] Error pattern: Rich error panel + `raise typer.Exit(code=1)` (FR-CLI-18)
  - [ ] User cancel: `raise typer.Abort()` (FR-CLI-18)

- [ ] Task 2: `worktrace init` command
  - [ ] Signature: `init(workspace_name: str, repos: str = typer.Option(...), ticket: Optional[str] = typer.Option(None))`
  - [ ] Split `repos` on comma; validate each is a git repo (has `.git/`) with absolute path resolution (FR-CLI-04)
  - [ ] For each valid repo: store `local_path` (as-given str) + `local_path_canonical` (Path.resolve()) in DB (FM-12)
  - [ ] Invalid path → Rich error per path; continue processing valid paths; exit code 1 after all processed
  - [ ] Idempotent: if workspace exists, update; if repo already registered, upsert without data loss (FR-CLI-05)
  - [ ] `--install-hooks` flag (stub for Story 2.6; flag exists but prints "hooks install coming in Epic 2")
  - [ ] Print tip message after success (ARCH-08)

- [ ] Task 3: `worktrace status` command
  - [ ] Query `list_workspaces()` from db.py
  - [ ] Rich Table with columns: Workspace, Ticket, Repos, Last Updated
  - [ ] Empty state: print "No workspaces initialized. Run `worktrace init` to get started."

- [ ] Task 4: `worktrace serve` command
  - [ ] Check git binary present (FM-03): `subprocess.run(["git", "--version"])` → clear error if fails
  - [ ] Call `check_schema_version(session)` at startup (FM-02)
  - [ ] Call `mcp.run()` for stdio mode (FastMCP default)
  - [ ] `--dry-run`: delegate to server dry-run path (FR-MCP-12)
  - [ ] `--verbose`: enable debug logging (FR-MCP-13)
  - [ ] `--http --port PORT`: enable HTTP mode (FR-MCP-01)

- [ ] Task 5: Integration tests
  - [ ] `tests/integration/test_cli.py` using `typer.testing.CliRunner`
  - [ ] Test: valid init, ticket ref stored, invalid path error, idempotent re-init, add-repo to existing
  - [ ] Test: status with workspaces, status empty
  - [ ] Test: serve --dry-run doesn't hang (quick exit); verify tool names printed

## Dev Notes

**FR-CLI-04:** Validate each repo path has a `.git/` directory. Use `Path(repo_path).is_dir()` and `(Path(repo_path) / ".git").exists()`. Display error per invalid path and continue.

**ARCH-08:** The tip message must appear exactly as: `"Tip: add --install-hooks to auto-run \`worktrace log\` after every commit in registered repos."`

**Rich usage:** Import `from rich.console import Console; console = Console()`. Use `console.print(...)` for all output. Use `console.print(Panel(..., style="red"))` for errors. The ONLY exception is `worktrace export` (future story) which uses `print()` for pipe-safe JSON.

**typer CLI runner in tests:**
```python
from typer.testing import CliRunner
from worktrace.cli import app

runner = CliRunner()
result = runner.invoke(app, ["init", "my-ws", "--repos", "/path/to/repo"])
assert result.exit_code == 0
```

**`worktrace serve` in tests:** Use `--dry-run` to avoid hanging the test runner.

**FM-03:** `worktrace serve` startup must check git binary. Pattern:
```python
import shutil
if not shutil.which("git"):
    console.print("[red]git not found. WorkTrace requires git to be installed.[/red]")
    raise typer.Exit(code=1)
```

## Dev Agent Record

### Implementation Plan

### Debug Log

### Completion Notes

## File List

## Change Log
