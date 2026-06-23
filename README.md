# WorkTrace

Shared workspace memory server for coding agents working across multiple repos on the same ticket.

## Quick Start

```bash
# Install
uvx worktrace serve

# Or clone + develop
git clone https://github.com/josechudev/worktrace.git
cd worktrace
uv sync
uv run worktrace --help
```

## What It Does

Collects cross-repo work state from git logs and spec files, then exposes it via an MCP server so any agent (Claude Code, Cursor, Copilot) can query what happened in every repo without re-explaining context.

```bash
worktrace init TICKET-123 --repos api,frontend,shared
worktrace log        # Run after any coding session
worktrace serve      # Start MCP server
```

MCP config (add to Claude Code / Cursor):
```json
{
  "worktrace": {
    "command": "uvx worktrace serve"
  }
}
```

Data stored locally at `~/.worktrace/worktrace.db` — no cloud, no vendor account.

## Tech Stack

- **Python:** 3.13+
- **Package manager:** uv
- **CLI:** Typer
- **MCP server:** FastMCP
- **Schema:** Pydantic v2
- **ORM:** SQLModel
- **Database:** SQLite

## MVP Scope

Shipping in v1.0 (E1–E5):
- **E1:** MCP server core (FastMCP, 6 tools, SQLite)
- **E2:** Git collector (commits, stubs, decisions, interfaces)
- **E3:** Spec/docs collector (.spec/ folders, BMad stories, CONTEXT.md)
- **E4:** CLI shell (init, log, serve, status)
- **E5:** Workspace context schema (Pydantic v2 + SQLModel)

Deferred to v2: Jira adapter (E7), standup output (E8).

## Development

### Setup

```bash
git clone https://github.com/josechudev/worktrace.git
cd worktrace
uv sync
```

### Running

```bash
# Dev server with hot reload
uv run worktrace serve --reload

# Run tests
uv run pytest

# Build for distribution
uv build
```

### Project Structure

```
worktrace/
├── src/worktrace/
│   ├── __main__.py           # CLI entry point
│   ├── cli/                  # Typer CLI commands
│   ├── server/               # FastMCP server implementation
│   ├── collectors/           # Git, spec, docs collectors
│   ├── schema/               # Pydantic models
│   └── storage/              # SQLModel + SQLite layer
├── tests/
├── pyproject.toml
└── README.md (this file)
```

## Planning & Documentation

All requirements, architecture, and epic definitions live in the separate [worktrace-docs](https://github.com/josechudev/worktrace-docs) repo:

- **PRD:** `prd-draft.md` — full requirements
- **Epics:** `_bmad-output/planning-artifacts/epics.md` — broken-down work
- **Architecture:** `_bmad-output/planning-artifacts/architecture.md` — tech decisions

When starting work, read `worktrace-docs/README.md` first.

## License

BSL 1.1 → Apache 2.0 after 4 years
