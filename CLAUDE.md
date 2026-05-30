# CLAUDE.md

Guidance for AI assistants working in this repository. See README.md for the
user-facing overview.

## Project

A minimal, single-file CLI for the **nemlig.com grocery API**, focused on
programmatic/agent use. This is a slimmed fork of
[eisbaw/nemlig_cli](https://github.com/eisbaw/nemlig_cli): the AI meal-planner,
Google Sheets import, camera scanner, local grocery list, fridge inventory and
interactive REPL have been removed. Scope is the five nemlig.com API commands —
`search`, `details`, `basket`, `add`, `history` — each with a global `--json` flag.

## Conventions (must follow)

- **Single file.** All logic lives in `nemlig_cli.py`. Do not split it into a package.
- **No new dependencies.** Runtime deps are `requests` + `argcomplete` only (stdlib
  otherwise). Do not reintroduce anthropic / google / opencv / pyzbar / Pillow / etc.
- **`--json` first.** Every command that produces output routes its result through the
  `emit()` / `emit_error()` helpers; progress notes go through `progress()` (stderr,
  silenced in `--json`). In `--json` mode stdout must be clean JSON — no ANSI codes, no
  status text. Don't scatter `if JSON_OUTPUT` checks; use the helpers.
- **Default output unchanged.** Without `--json`, output stays human-formatted/colored.
- **Credentials** resolve in order: `-u`/`-p` flags → `NEMLIG_USER`/`NEMLIG_PASS` env
  vars → `~/.config/nemlig/login.json`.
- **No real personal data** anywhere (code, docs, commits). Use placeholders like
  "Anders And", "Vesterbrogade 42", order id `12345678`.
- **Keep `SKILL.md` up to date.** Whenever a command, flag, JSON shape, credential
  behavior, or error format changes, update `SKILL.md` in the same change so agents
  always have an accurate guide.

## Quick Commands

```bash
nemlig search "cocio"        # Search products
nemlig --json search "cocio" # Search products, JSON output
nemlig details 701025        # Product details
nemlig basket                # View basket
nemlig add 701025 -q 2       # Add product
nemlig history               # Order history
```

Install with `uv tool install .` to get the `nemlig` executable on `PATH`
(otherwise use `uv run python nemlig_cli.py`). The global `--json` flag works before
or after the subcommand; on failure it prints `{"error": ..., "command": ...}` and
exits non-zero.

## Files

- `nemlig_cli.py` — the CLI (all logic)
- `nemlig_api.md` — nemlig.com API reference (source of truth for endpoints)
- `SKILL.md` — agent usage guide (Claude skill); mirrored at `~/.claude/skills/nemlig-cli/`
- `pyproject.toml` — packaging; `uv tool install` exposes the `nemlig` entry point

## Extending the API client

The nemlig.com API is undocumented; endpoints were reverse-engineered by driving a real
browser. Chrome DevTools MCP is configured in `.mcp.json` (via
`chrome-devtools-mcp-wrapper.sh`) for that work — MCP responses are large (>25KB), so run
them from a sub-agent and have it return only a summary (endpoint, headers, body format).
`arch_api.drawio.svg` documents the auth/search flow. The `/drawio-updater` and
`/privacy-checker` slash commands (in `.claude/commands/`) audit the diagrams and scan
for personal-data leaks before committing.
