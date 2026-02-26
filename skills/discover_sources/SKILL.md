---
description: "Use when the user wants to find data files in their workspace, scan for available data sources, or see what sources are already registered. Trigger phrases: 'find data files', 'scan for sources', 'what data is available', 'discover sources', 'what sources do I have'."
tools:
  - dataraum:discover_sources
alwaysApply: false
---

# Discover Sources

Scan the workspace for data files and list existing registered sources using the DataRaum MCP tool.

## CRITICAL: Path Translation Required

The DataRaum MCP server runs on the user's **host machine**, not inside the VM where Claude operates. If the user provides a VM path (like `/sessions/xyz/mnt/folder/`), you must translate it to the corresponding host path before passing it to the tool.

To discover the host workspace root, call `get_context` first and extract the path from the `output_directory` field. See the **analyze** skill for full path translation rules.

## How to Use

Call the `discover_sources` MCP tool:

```
discover_sources()
discover_sources(path="/Users/name/folder/")
discover_sources(path="/Users/name/folder/", recursive=false)
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | string | Optional: root directory to scan. Defaults to current working directory. |
| `recursive` | boolean | Optional: scan subdirectories. Default: true. |

### Supported Formats

Scans for: CSV, Parquet, JSON, XLSX files.

## What You Get

- **Found files**: path, format, column names, estimated row count
- **Registered sources**: sources already added via `add_source`, with their status

## Response Pattern

**Resuming after skill load:** If this skill loaded mid-conversation, resume immediately — do not wait for the user to re-prompt. Check the conversation history and continue from where things left off.

1. Call `discover_sources()` (with a path if the user specified one)
2. Summarize found files in a table:

   | File | Format | Columns | Rows (est.) |
   |------|--------|---------|-------------|
   | sales.csv | CSV | 12 | ~50,000 |
   | products.parquet | Parquet | 8 | ~1,200 |

3. Note any files that are **already registered** as sources
4. For unregistered files, suggest: "Want to register any of these? I can add them with `add_source` so they're included in future analysis runs."
5. If the user has no registered sources and files were found, suggest registering them and then running `analyze()`
