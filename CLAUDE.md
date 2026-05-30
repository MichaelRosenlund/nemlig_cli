# CLAUDE.md

AI assistant guidance for this repository. See README.md for project overview and workflow documentation.

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
(otherwise use `uv run python nemlig_cli.py`). Pass `-u`/`-p` or set
`NEMLIG_USER`/`NEMLIG_PASS` (or `~/.config/nemlig/login.json`).

Every command accepts a global `--json` flag (works before or after the
subcommand) to emit machine-readable JSON to stdout instead of colored text;
on failure it prints `{"error": ..., "command": ...}` and exits non-zero.

## MCP Usage

Chrome DevTools MCP is configured via `.mcp.json`. Use for API discovery and debugging.

**Critical**: MCP calls return large payloads (>25KB). Always run MCP interactions from a sub-agent to avoid context bloat.

**Privacy**: Never record actual personal information (real names, addresses, phone numbers, order IDs). Replace with realistic placeholder values when documenting APIs (e.g., "Anders And", "Vesterbrogade 42", "+4512345678").

Pattern:
1. Sub-agent navigates, records network traffic, performs action
2. Sub-agent returns summary (endpoint, headers, body format)
3. Main context updates documentation or implements code

## Diagrams

Diagrams are stored as `.drawio.svg` files (SVG with embedded draw.io source). Keep them updated when architecture changes.

**To edit**: Open `.drawio.svg` directly in draw.io - the source is embedded.

**To create/update**:
```bash
# Create/edit in draw.io, save as .drawio file, then export:
drawio -x -f svg --embed-diagram -o diagram.drawio.svg diagram.drawio
rm diagram.drawio  # Keep only the .svg
```

Current diagrams:
- `arch_api.drawio.svg` - API architecture (endpoints, auth flow)
- `mcp-workflow.drawio.svg` - MCP workflow for API discovery

## Project Commands

Custom slash commands for this project. **Run both in sub-agents in parallel before every commit.**

- `/drawio-updater` - Audit and update `.drawio.svg` diagrams
- `/privacy-checker` - Scan files for personal data leaks

## Files

- `nemlig_cli.py` - Single-file Python client
- `nemlig_api.md` - API documentation (source of truth for endpoints)
- `SKILL.md` - Agent usage guide for the CLI (Claude skill)

**Keep `SKILL.md` up to date.** Whenever a command, flag, JSON shape, credential
behavior, or error format changes in `nemlig_cli.py`, update `SKILL.md` in the same
change so agents always have an accurate guide.
