# Remote Expense Tracker MCP Server

An expense tracking Model Context Protocol (MCP) server built with Python, [FastMCP](https://github.com/jlowin/fastmcp), and `aiosqlite`. It enables LLM assistants (such as Claude Desktop or Gemini) to record, query, and summarize personal expenses.

This repository features both a standalone HTTP server and an STDIO proxy configuration that links to a remote hosted server instance.

---

## Features

- **Transaction Management**: Record transactions with dates, amounts, categories, subcategories, and custom notes.
- **Filtering & Listing**: Query expenses by date range.
- **Reporting**: Aggregate and summarize expenses grouped by category over specified date ranges.
- **Dynamic Categories Resource**: Exposes a hierarchical taxonomy of valid categories and subcategories.
- **Cloud Proxy Connection**: Contains a local client proxy script that forwards commands to a hosted Cloud MCP server.

---

## File Structure

- [main.py](file:///c:/Users/JS%20BHATI/Desktop/remote-expense-trac-mcp-server/main.py): Contains the primary server code implementing tools (`add_expense`, `list_expenses`, `summarize`) and the `categories` resource. Uses an asynchronous connection to SQLite.
- [proxy.py](file:///c:/Users/JS%20BHATI/Desktop/remote-expense-trac-mcp-server/proxy.py): Run this script to spin up a local STDIO proxy connected to the hosted remote server (`https://arbitrary-turquoise-gorilla.fastmcp.app/mcp`).
- [categories.json](file:///c:/Users/JS%20BHATI/Desktop/remote-expense-trac-mcp-server/categories.json): Defines the hierarchical categories and subcategories (e.g. `food` containing `groceries`, `dining_out`, etc.).
- [pyproject.toml](file:///c:/Users/JS%20BHATI/Desktop/remote-expense-trac-mcp-server/pyproject.toml): Lists metadata and required dependencies (`fastmcp` and `aiosqlite`).

---

## Database

The database is built on SQLite (`expenses.db`). To avoid read/write conflicts, initialization is done synchronously and CRUD tools run asynchronously using `aiosqlite`. 

The database file is stored in your OS temporary directory (`tempfile.gettempdir()`) to ensure reliable write access:
```sql
CREATE TABLE IF NOT EXISTS expenses(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    date TEXT NOT NULL,
    amount REAL NOT NULL,
    category TEXT NOT NULL,
    subcategory TEXT DEFAULT '',
    note TEXT DEFAULT ''
);
```

---

## Getting Started

### Prerequisites
- Python `>= 3.12`
- Recommended: [uv](https://github.com/astral-sh/uv) package manager

### 1. Install Dependencies
You can install dependencies defined in `pyproject.toml`:
```bash
uv pip install -e .
```
Or via standard pip:
```bash
pip install fastmcp aiosqlite
```

### 2. Running the Server

#### Run Local HTTP Server
Starts the server locally as an HTTP service listening on port `8000`:
```bash
python main.py
```

#### Run STDIO Proxy Client
To connect a local client (like Claude Desktop) to the remote FastMCP hosted cloud instance:
```bash
python proxy.py
```

---

## MCP Server Interface

### Tools
- **`add_expense(date: str, amount: float, category: str, subcategory: str = "", note: str = "")`**: Adds a new expense entry.
- **`list_expenses(start_date: str, end_date: str)`**: Lists expenses within an inclusive date range.
- **`summarize(start_date: str, end_date: str, category: str = None)`**: Calculates totals grouped by category for a date range.

### Resources
- **`expense:///categories`**: Returns JSON formatted list of supported categories and their subcategories.

---

## Configuring Claude Desktop

To use this server with Claude Desktop, add the following configuration to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "expense-tracker-proxy": {
      "command": "python",
      "args": ["c:/Users/JS BHATI/Desktop/remote-expense-trac-mcp-server/proxy.py"]
    }
  }
}
```
*Note: Adjust the file path and python command as necessary for your environment.*
