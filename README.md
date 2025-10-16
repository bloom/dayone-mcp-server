# Day One MCP Server

A Model Context Protocol (MCP) server that provides AI assistants with programmatic access to Day One journals through stdio communication.

## What is this?

The Day One MCP Server is provided as part of the Day One Mac application and implements the [Model Context Protocol](https://modelcontextprotocol.io/) to expose Day One journal operations as tools that AI assistants can invoke.

**This repository provides:**
- 📦 Distribution of the `.mcpb` bundle for easy installation in MCP clients
- 📖 Documentation for setup and usage

**Capabilities:**
- 📝 Programmatic journal entry creation and updates
- 🔍 Full-text search across journal entries
- 🏷️ Tag and metadata management
- 📅 Date-based entry retrieval and filtering

## Technical Overview

The MCP server runs as a local stdio-based process that communicates via JSON-RPC 2.0:

- **Transport**: stdio (standard input/output)
- **Data Access**: Direct read/write to Day One's Core Data store
- **Security**: Journal-level access control with opt-in configuration

> **Privacy Note:** The MCP server runs entirely on your local machine. However, MCP clients will send retrieved journal content to their respective LLM providers for processing.

## Requirements

- **macOS only** - The Day One MCP server is currently available only for Day One for Mac
- **Day One Mac app** - Available on the [Mac App Store](https://apps.apple.com/us/app/day-one/id1055511498?mt=12)
- **Day One CLI** - Required for MCP server functionality

## Installation

### Step 1: Install Day One for Mac

Download and install Day One from the Mac App Store:

**[Day One on Mac App Store](https://apps.apple.com/us/app/day-one/id1055511498?mt=12)**

### Step 2: Install Day One CLI

The MCP server requires the Day One CLI. Install it by running:

```bash
sudo bash /Applications/Day\ One.app/Contents/Resources/install_cli.sh
```

This installs the `dayone` command to `/usr/local/bin/dayone`.

Verify installation:
```bash
/usr/local/bin/dayone --version
```

### Step 3: Configure MCP Client

The Day One MCP server is invoked via the `dayone mcp` command, which starts a stdio-based MCP server process.

#### For Claude Desktop

Download and install the pre-built `.mcpb` bundle:

1. Download the latest `.mcpb` file from [Releases](https://github.com/automattic/dayone-mcp-server/releases)
2. Open Claude Desktop → Settings → Extensions
3. Drag and drop the `.mcpb` file to install

#### For Claude Code CLI

```bash
claude mcp add --scope user --transport stdio dayone-cli /usr/local/bin/dayone mcp
```

#### For Cursor

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "dayone-cli": {
      "command": "/usr/local/bin/dayone",
      "args": ["mcp"],
      "transport": "stdio"
    }
  }
}
```

#### Manual Configuration for Other MCP Clients

Most MCP clients accept stdio server configurations. Configure with:

- **command**: `/usr/local/bin/dayone`
- **args**: `["mcp"]`
- **transport**: `stdio`

The server communicates via JSON-RPC 2.0 over stdin/stdout. Refer to your MCP client's documentation for configuration specifics.

### Step 4: Configure Journal Access

For security and privacy, you must explicitly grant MCP access to journals:

1. **Open Day One** on your Mac
2. **Go to Preferences → Labs**
3. **Enable** "Mac CLI MCP Server"
4. **Click** "MCP Access Control"
5. **Toggle on** the journals you want to make accessible

> **Important:** Only enabled journals will be visible to AI assistants. You have complete control over access.

## MCP Tools

The server exposes the following MCP tools via the `tools/list` and `tools/call` methods:

### `list_journals`

Returns all journals with MCP access enabled.

**Example:**
```
"Show me all my journals"
```

---

### `create_entry`

Creates a new journal entry with markdown content and optional metadata.

**Parameters:**
- `text` (required) - Entry content in markdown format
- `journal_id` or `journal_name` (optional) - Target journal
- `date` (optional) - ISO8601 timestamp (e.g., `2025-08-20T15:30:00Z`)
- `tags` (optional) - Comma-separated tag list
- `attachments` (optional) - Comma-separated file paths
- `starred`, `all_day` (optional) - Boolean flags

**Example:**
```
"Create a journal entry about today's meeting with tags 'work' and 'meeting'"
```

---

### `get_entries`

Retrieves entries via full-text search, date filters, or journal constraints.

**Parameters:**
- `query` (optional) - Full-text search query (triggers search index)
- `journal_ids` or `journal_names` (optional) - Filter by journals
- `start_date`, `end_date` (optional) - Date range in YYYY-MM-DD format
- `on_this_day` (optional) - MM-DD format for anniversary queries
- `limit` (optional) - Max results (default: 10, max: 50)
- `offset` (optional) - Pagination offset (ignored for search queries)

**Examples:**
```
"Find entries about vacation from last summer"
"What did I write on this day in previous years?"
"Show recent entries from my Travel Journal"
```

---

### `update_entry`

Updates an existing entry's content or metadata.

**Parameters:**
- `entry_id` (required) - Entry UUID
- `journal_id` (optional) - Journal sync ID for disambiguation
- `text` (optional) - New markdown content
- `tags` (optional) - Comma-separated tags (replaces existing)
- `attachments` (optional) - File paths to add
- `starred`, `all_day` (optional) - Boolean flags

**Example:**
```
"Add the tag 'important' to my last entry"
```

## Security Model

### Password Lock Protection
The MCP server refuses to start when Day One's password lock is enabled, returning an error on stderr. This prevents unauthorized access when the Mac is unlocked but Day One requires authentication.

### Journal-Level Access Control
Access is controlled at the journal level via Day One's preferences. The server only exposes journals with the `isMcpAccessAllowed` flag set to `true`. All operations validate journal access before executing.

### Local-Only Execution
The MCP server process runs locally with no network I/O. All data access is performed directly against Day One's Core Data SQLite store. However, note that MCP clients will typically send retrieved content to remote LLM APIs.


## Support

For issues or questions:
- 📖 [Day One Help Center](https://dayoneapp.com/guides/)
- 🐛 [Report an Issue](https://github.com/automattic/dayone-mcp-server/issues)

---

© 2025 Automattic, Inc. Day One is a registered trademark of Automattic, Inc.
