# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GnuCash MCP Server - An MCP (Model Context Protocol) server that provides access to GnuCash financial data. Supports read-only mode (default) and write mode for creating transactions. Uses the official GnuCash Python bindings (system package, not pip-installable).

## Development Setup

```bash
# Create venv with system packages (required for gnucash bindings)
python3 -m venv --system-site-packages .venv
source .venv/bin/activate
pip install mcp

# Verify gnucash bindings are available
python3 -c "import gnucash; print('OK')"
```

## Running the Server

```bash
# GNUCASH_FILE is required - server exits if not set
GNUCASH_FILE=/path/to/file.gnucash python3 server.py

# Write mode (enables add_transaction and commit tools)
GNUCASH_FILE=/path/to/file.gnucash python3 server.py --write
```

## Architecture

Single-file MCP server (`server.py`) using FastMCP framework:

- **Auto-open**: File opens automatically on startup from `GNUCASH_FILE` env var (required)
- **Stale lock handling**: Automatically removes stale `.LCK` files if GnuCash app is not running
- **Auto-save on exit**: In write mode, changes are saved automatically when session ends
- **Read tools**: `list_accounts`, `get_account_balance`, `get_transactions`, `search_accounts`, `get_account_info`
- **Write tools** (require `--write` flag): `add_transaction`, `commit`
- **Account matching**: Supports exact match, suffix match (dot notation), and case-insensitive partial match
- **Balance conversion**: GncNumeric values converted via `num()/denom()` to floats
- **Transaction validation**: GnuCash Python bindings enforce double-entry bookkeeping - transactions must balance

## Key Constraints

- **GNUCASH_FILE required**: Server exits with error if not set
- **Read-only by default**: Opens files with `SessionOpenMode.SESSION_READ_ONLY`
- **Write mode optional**: Use `--write` flag to enable `SESSION_NORMAL_OPEN` and write tools
- **System dependency**: Requires `python3-gnucash` package installed via system package manager
- **No uvx/pipx**: Must use `--system-site-packages` venv to access gnucash bindings
- **Same-currency transactions**: `add_transaction` only supports transactions between accounts with the same currency
