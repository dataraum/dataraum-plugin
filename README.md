# DataRaum Plugin

Data context and quality analysis for knowledge work. Understand your data structure, identify quality issues, and get prioritized actions to fix them.

## How It Works

This plugin provides an MCP server that exposes DataRaum tools directly to Claude. No HTTP API, no external services — everything runs locally against your pipeline output.

## Quick Start

### 1. Set Your Output Directory

Edit `.mcp.json` and set `DATARAUM_OUTPUT_DIR` to your pipeline output path.

### 2. Analyze Your Data

Just ask Claude to analyze your data — no CLI required:

- "Analyze the CSV at /path/to/data.csv"
- "Process my Parquet files in /path/to/data/"
- "Scan for data files" (discovers files, then register and analyze)
- "Analyze my data" (runs on all registered sources)

Or run the pipeline from the command line:

```bash
dataraum run /path/to/your/data --output ./pipeline_output
```

### 3. Ask Questions

- "What tables do I have in my data?"
- "Show me the entropy for the orders table"
- "What data quality issues should I fix?"
- "Is my data ready for aggregation?"
- "How many customers placed orders last month?"

## Skills

### Analyze
**Trigger:** "analyze this data", "process my CSV", "load this file", "run the pipeline"

Runs the full analysis pipeline on CSV or Parquet data. Must be called before other tools if no data has been analyzed yet.

### Context
**Trigger:** "what tables", "show me the schema", "describe the data", "what data is available"

Returns schema, semantic annotations, relationships, and quality indicators.

### Quality
**Trigger:** "entropy", "uncertainty", "contract", "compliance", "aggregation safe", "fix quality", "what should I fix", "improve data"

Unified quality analysis — entropy scores, contract evaluation, and resolution actions in a single report. Use `include` to request specific sections.

### Query
**Trigger:** "how many", "total", "calculate", "analyze", "what is the"

Executes natural language queries against the data with entropy-aware confidence levels.

### Export
**Trigger:** "export", "save to file", "write to CSV", "download as parquet", "materialize"

Export query results or SQL output to CSV, Parquet, or JSON files with metadata sidecars carrying provenance (SQL, entropy, assumptions).

### Source Management

### Discover Sources
**Trigger:** "find data files", "scan for sources", "what data is available", "what sources do I have"

Scans the workspace for data files (CSV, Parquet, JSON, XLSX) and lists existing registered sources.

### Add Source
**Trigger:** "add source", "register data", "connect database", "add my CSV", "connect to postgres"

Registers a new data source — either a file path or a database connection (Postgres, MySQL, SQLite).

## MCP Tools

| Tool | Description |
|------|-------------|
| `analyze` | Run analysis pipeline on CSV/Parquet data |
| `get_context` | Full data context document for AI analysis |
| `get_quality` | Unified quality report: entropy, contracts, actions |
| `query` | Natural language query execution |
| `export` | Export query/SQL results to CSV, Parquet, or JSON with metadata |
| `discover_sources` | Scan workspace for data files and list registered sources |
| `add_source` | Register a file or database as a data source |

## Requirements

- DataRaum installed (`pip install dataraum`)
- Pipeline output directory with analyzed data
- `DATARAUM_OUTPUT_DIR` environment variable set

See [CONNECTORS.md](CONNECTORS.md) for detailed setup.
