---
description: "Use when the user wants to export, save, or download query results to a file. Trigger phrases: 'export', 'save to file', 'write to CSV', 'download as', 'export to parquet', 'save results', 'materialize', 'write out'."
tools:
  - dataraum:export
alwaysApply: false
---

# Data Export

Export query results or SQL output to CSV, Parquet, or JSON files with metadata sidecars.

## How to Use

Call the `export` MCP tool:

```
export(question="What is total revenue by region?", output_path="./exports/revenue.csv")
export(sql="SELECT * FROM typed_orders WHERE amount > 1000", output_path="./exports/large_orders.parquet", format="parquet")
export(question="Show monthly sales trends", output_path="./exports/trends.json", format="json")
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `question` | string | Natural language question (runs query agent, then exports results) |
| `sql` | string | Raw SQL to execute and export (alternative to question) |
| `output_path` | string | **Required**: destination file path |
| `format` | string | Export format: `csv` (default), `parquet`, or `json` |

Provide either `question` or `sql`, not both.

## What You Get

- **Data file**: CSV, Parquet, or JSON with query results
- **Metadata sidecar**: `.meta.json` file alongside the data with provenance:
  - Source SQL and execution steps
  - Entropy score and confidence level
  - Assumptions made during query
  - Timestamp and execution ID

## When to Use Each Format

| Format | Best for |
|--------|----------|
| `csv` | Sharing with non-technical users, spreadsheet import |
| `parquet` | Large datasets, downstream analytics pipelines |
| `json` | API consumption, programmatic access |

## Response Pattern

**Resuming after skill load:** If this skill loaded mid-conversation, resume immediately — do not wait for the user to re-prompt. Check the conversation history and continue from where things left off.

1. **Check for existing data first**: Call `get_context` to see if data has been analyzed.
   - If no data exists → call `analyze` first, then continue
2. Call the `export` tool with the user's question or SQL and desired output path
3. Confirm the export: file path, format, row count
4. Mention the metadata sidecar for provenance tracking
5. If the export fails, suggest checking the query or data availability
