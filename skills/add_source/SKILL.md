---
description: "Use when the user wants to register a data source, connect a database, add a CSV or Parquet file to analysis, or set up a data connection. Trigger phrases: 'add source', 'register data', 'connect database', 'add my CSV', 'register source', 'connect to postgres', 'add parquet'."
tools:
  - dataraum:add_source
  - dataraum:discover_sources
alwaysApply: false
---

# Add Source

Register a new data source (file or database) using the DataRaum MCP tool.

## Two Modes

### File Mode

Register a CSV, Parquet, JSON, or XLSX file (or directory):

```
add_source(name="sales", path="/Users/name/folder/sales.csv")
add_source(name="warehouse", path="/Users/name/folder/data/")
```

### Database Mode

Register a database connection:

```
add_source(name="production", backend="postgres")
add_source(name="analytics", backend="postgres", tables=["orders", "customers"])
add_source(name="local_db", backend="sqlite", credential_ref="my_sqlite")
```

## CRITICAL: Path Translation for File Sources

The DataRaum MCP server runs on the user's **host machine**, not inside the VM. For file sources, you must translate VM paths to host paths before passing them to the tool. See the **analyze** skill for full path translation rules.

Database sources do not require path translation.

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | **Required**: Source name. Must match `^[a-z][a-z0-9_]{1,48}$` (lowercase, starts with letter, 2-49 chars). |
| `path` | string | File mode: host path to file or directory. Mutually exclusive with `backend`. |
| `backend` | string | Database mode: `postgres`, `mysql`, or `sqlite`. Mutually exclusive with `path`. |
| `tables` | array | Optional: table name filter for database sources. |
| `credential_ref` | string | Optional: credential lookup key. Defaults to source name. |

You must provide **either** `path` or `backend`, not both.

### Name Rules

- Lowercase letters, digits, and underscores only
- Must start with a letter
- 2-49 characters long
- Examples: `sales`, `q4_orders`, `production_db`

## Credential Handling

Database sources need connection credentials. The response tells you the status:

### `status: "validated"`

Connection works. The source is ready — suggest running `analyze()`.

### `status: "needs_credentials"`

The response includes `credential_instructions` explaining how to provide credentials. Present these instructions to the user:

**Option 1: Environment variable**
```
DATARAUM_{NAME}_URL=postgresql://user:pass@host:5432/dbname
```
Where `{NAME}` is the source name in UPPERCASE (e.g., source `production` → `DATARAUM_PRODUCTION_URL`).

**Option 2: Credentials file**
```yaml
# ~/.dataraum/credentials.yaml
sources:
  production:
    url: postgresql://user:pass@host:5432/dbname
```

**NEVER auto-create credential files or set environment variables containing secrets.** Always show the instructions and let the user set up credentials themselves.

After the user configures credentials, they can re-run `add_source` with the same name to re-validate the connection.

## Response Pattern

**Resuming after skill load:** If this skill loaded mid-conversation, resume immediately — do not wait for the user to re-prompt. Check the conversation history and continue from where things left off.

### File Source Flow

1. Translate the user's path to the host path
2. Call `add_source(name="...", path="...")`
3. Confirm registration: "Registered **name** from `path`."
4. If more sources to add, continue; otherwise suggest `analyze()`

### Database Source Flow

1. Call `add_source(name="...", backend="...")`
2. Check the response status:
   - **validated** → "Connected to **name**. Ready to analyze."
   - **needs_credentials** → show the credential setup instructions from the response
3. Once connected, suggest `analyze()`

### After All Sources Registered

When the user has finished adding sources, suggest:

"All sources registered. Ready to run the analysis pipeline? I'll call `analyze()` to process everything — this takes 3-7 minutes."
