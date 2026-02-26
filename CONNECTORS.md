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

| Tool | Description | Required Parameters |
|------|-------------|---------------------|
| `analyze` | Run analysis pipeline on CSV/Parquet data | `path` (optional), `name` (optional) |
| `get_context` | Full data context: schema, relationships, semantic annotations, quality | None |
| `get_entropy` | Entropy analysis: data uncertainty by dimension | `table_name` (optional) |
| `evaluate_contract` | Evaluate data quality against a contract | `contract_name` |
| `query` | Natural language query against the data | `question`, `contract_name` (optional) |
| `get_actions` | Prioritized resolution actions for data quality | `priority` (optional), `table_name` (optional) |
| `discover_sources` | Scan workspace for data files and list registered sources | `path` (optional), `recursive` (optional) |
| `add_source` | Register a file or database as a data source | `name`, `path` or `backend`, `tables` (optional), `credential_ref` (optional) |
| `remove_source` | Archive or delete a registered data source | `name`, `purge_results` (optional) |

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
