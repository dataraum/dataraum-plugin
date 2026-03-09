# Connectors

## MCP Server

This plugin uses an MCP (Model Context Protocol) server to communicate with the DataRaum engine. No HTTP API or external services are needed — everything runs locally.

## Setup

### 1. Install DataRaum

```bash
pip install dataraum
# or with uv
uv add dataraum
```

### 2. Configure MCP Server

The `.mcp.json` file in this plugin directory configures the MCP server. Set `DATARAUM_OUTPUT_DIR` to your pipeline output directory.

The MCP server starts automatically when Claude Code or Claude Desktop loads the plugin.

## Available MCP Tools

### Core tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `analyze` | Run analysis pipeline on CSV/Parquet data | `path`, `name?`, `gate_mode?` |
| `get_context` | Full data context: schema, relationships, semantic annotations, quality | — |
| `get_quality` | Unified quality report: entropy, contract evaluation, resolution actions | `contract_name?`, `table_name?`, `priority?`, `include?` |
| `query` | Natural language query against the data | `question`, `contract_name?` |

The `include` parameter on `get_quality` accepts a list of sections: `entropy`, `contract`, `actions`. Defaults to all three.

### Source management tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `discover_sources` | Scan workspace for data files and list registered sources | `path?`, `recursive?` |
| `add_source` | Register a file or database as a data source | `name`, `path?`, `backend?`, `tables?`, `credential_ref?` |

### Contract names

`exploratory_analysis`, `data_science`, `operational_analytics`, `aggregation_safe`, `executive_dashboard`, `regulatory_reporting`

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATARAUM_OUTPUT_DIR` | Path to pipeline output directory | `./pipeline_output` |
| `DATARAUM_{SOURCE}_URL` | Connection URL for a database source (uppercase source name) | — |

### Database Credentials

Database sources registered via `add_source` need connection credentials. Two options:

**Environment variable** (recommended for CI/containers):
```
DATARAUM_PRODUCTION_URL=postgresql://user:pass@host:5432/dbname
```

**Credentials file** (recommended for local development):
```yaml
# ~/.dataraum/credentials.yaml
sources:
  production:
    url: postgresql://user:pass@host:5432/dbname
```

The credential lookup key defaults to the source name but can be overridden with the `credential_ref` parameter on `add_source`.
