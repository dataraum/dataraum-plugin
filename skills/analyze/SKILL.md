---
description: "Use when the user wants to analyze new data, process a CSV or Parquet file, load a dataset, or run the pipeline on their data. Trigger phrases: 'analyze this data', 'process my CSV', 'load this file', 'analyze the data at', 'run the pipeline', 'import this dataset'."
tools:
  - dataraum:analyze
  - dataraum:get_context
alwaysApply: false
---

# Analyze Data

Run the DataRaum analysis pipeline on a CSV or Parquet file (or directory of files) using the MCP tool.

## CRITICAL: Path Translation Required

The DataRaum MCP server runs on the user's **host machine** (Mac/Windows/Linux), not inside the VM where Claude operates. This means:

- **VM paths** (like `/sessions/xyz/mnt/folder/file.csv`) will NOT work
- **Host paths** (like `/Users/name/folder/file.csv`) are required

### Step 1: Discover the Host Workspace Path

**Before analyzing any file**, call `get_context` first. The response will contain path information:

- If a pipeline exists, the response includes the `output_directory` showing the host path
- If no pipeline exists, the error message contains the configured `DATARAUM_OUTPUT_DIR` path

From this, extract the **host workspace root**. For example:
- If `output_directory` is `/Users/florianphilippi/Downloads/Dataraum-claude/pipeline_output`
- Then the workspace root is `/Users/florianphilippi/Downloads/Dataraum-claude/`

### Step 2: Translate the File Path

The user's selected folder is mounted into the VM. Map the paths:

| VM Path Pattern | Host Path Pattern |
|-----------------|-------------------|
| `/sessions/.../mnt/{folder}/...` | `{workspace_root}/...` |

**Example:**
- User says: "analyze `/sessions/abc/mnt/Dataraum-claude/sales.csv`"
- VM folder name: `Dataraum-claude`
- Host workspace: `/Users/florianphilippi/Downloads/Dataraum-claude/`
- Translated path: `/Users/florianphilippi/Downloads/Dataraum-claude/sales.csv`

### Step 3: Handle Uploaded Files

Files uploaded directly to Claude land in `/sessions/.../mnt/uploads/` which has **no corresponding host path**. For these:

1. Copy the file to the user's selected folder first (using shell commands)
2. Then analyze using the translated host path

```bash
# Example: copy uploaded file to selected folder
cp /sessions/abc/mnt/uploads/data.csv /sessions/abc/mnt/Dataraum-claude/data.csv
```

Then analyze at the host path: `/Users/.../Dataraum-claude/data.csv`

## How to Use

This takes **several minutes** and always returns immediately. The pipeline runs in the background.

- **With task support**: progress updates are delivered automatically via the tasks API
- **Without task support**: call `get_context` periodically (~2 min intervals) to check progress — it reports the current phase while running and returns the full context document when done

Call the `analyze` MCP tool with the **host path**:

```
analyze(path="/Users/name/folder/data.csv")
analyze(path="/Users/name/folder/data.parquet")
analyze(path="/Users/name/folder/data_directory")
analyze(path="/Users/name/folder/data.csv", name="my_dataset")
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | string | **Required**: Host machine path to CSV/Parquet file or directory |
| `name` | string | Optional: name for the data source (defaults to filename) |

### Supported Formats

| Format | Extensions | Notes |
|--------|-----------|-------|
| CSV | `.csv` | Loaded as VARCHAR, types inferred by pipeline |
| Parquet | `.parquet`, `.pq` | Types preserved from file schema |

## What You Get

After analysis completes, you receive:

- **Tables found**: Names and row counts
- **Phases completed**: How many of the 18 analysis phases succeeded
- **Duration**: How long the analysis took
- **Next steps**: Suggested tools to call next

## What Happens During Analysis

The pipeline runs 18 phases:

1. **Import**: Load raw data into the analysis engine
2. **Typing**: Infer column types (or preserve Parquet types)
3. **Statistics**: Profile column distributions
4. **Correlations**: Detect numeric/categorical correlations
5. **Relationships**: Find join candidates and foreign keys
6. **Semantic Analysis**: Identify business meaning of columns (uses LLM)
7. **Temporal Analysis**: Detect time series patterns
8. **Quality Rules**: Generate validation rules (uses LLM)
9. **Entropy Detection**: Measure data uncertainty across dimensions
10. **Entropy Interpretation**: Generate human-readable quality assessments (uses LLM)
11. **Context Assembly**: Build the full data context document

## Complete Response Pattern

1. **Discover host path**: Call `get_context` to get the host workspace path from the response
2. **Translate path**: Convert the user's VM path to the corresponding host path
3. **Handle uploads**: If the file is in `/uploads/`, copy it to the selected folder first
4. **Call analyze**: Use the translated host path
5. **Monitor progress**: Call `get_context` periodically (~2 min intervals) to check progress
6. **Report results**: When complete, note any phase failures or warnings
7. **Suggest next steps**:
   - `get_context` to explore the schema
   - `get_entropy` to check data quality
   - `query` to ask questions about the data
