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

This takes **3–7 minutes** depending on file size and number of columns. Tell the user this upfront before starting: "This will take 3–7 minutes — I'll check in every 2 minutes and show you which phase we're on."

The pipeline runs in the background and always returns immediately.

- **With task support**: progress updates are delivered automatically via the tasks API
- **Without task support**: call `get_context` periodically (~2 min intervals) to check progress

### Phase Progress Translation

When polling with `get_context`, translate the current phase into plain language for the user. Always show: "Phase X of ~18: [plain language]. Still going — next check in ~2 minutes."

| Phase name | Plain language |
|-----------|----------------|
| `import` | Loading your data |
| `typing` | Detecting column types |
| `statistics` | Profiling value distributions |
| `correlations` | Checking for correlations |
| `relationships` | Finding relationships between tables |
| `semantic_analysis` | Understanding business meaning of columns *(AI step — takes a moment)* |
| `temporal` | Detecting date/time patterns |
| `quality_rules` | Generating quality rules *(AI step)* |
| `entropy_detection` | Measuring data uncertainty |
| `entropy_interpretation` | Writing quality summaries *(AI step)* |
| `context_assembly` | Assembling final results |

Phases 6, 8, and 10 are AI steps and may each take 1–2 minutes on their own — mention this when the user reaches them.

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

### Resuming After Skill Load

If this skill was just loaded in the middle of an ongoing workflow (e.g. the user had already said "analyze my data" or was partway through a pipeline run), **resume immediately** — do not wait for the user to repeat themselves. Check the conversation history to understand where the workflow was and continue from that point.

1. **Check for existing data first**: Call `get_context` first.
   - If `get_context` returns schema/table information → list the tables and row counts found, then ask: "Data already analyzed — would you like to re-analyze or work with the existing results?"
   - If `get_context` returns an error or "no sources found" → proceed to step 2
   - **Corrupt state recovery**: If `get_context` succeeded but a later tool call (`get_actions`, `get_entropy`, etc.) fails with "pipeline output not found" or "no data in database" — tell the user: "The analysis results exist in memory but the pipeline output files are missing — this can happen if the output directory was cleared. I'll re-run the analysis to regenerate them." Then call `analyze` with the source path from the `output_directory` field in the last `get_context` response and continue from step 5.

2. **Confirm the data path** with the user if it is not already clear from context (e.g. they said "analyze my data" without specifying a path).

3. **Translate path**: Convert the user's VM path to the corresponding host path (see steps above).

4. **Handle uploads**: If the file is in `/uploads/`, copy it to the selected folder first.

5. **Call analyze**: Use the translated host path.

6. **Monitor progress**: Call `get_context` periodically (~2 min intervals) to check progress.

7. **Report results**: When complete, note tables found, phases completed, and any phase failures or warnings.

8. **Automatically show the actions report**: Immediately call `get_actions()` and render the inline HTML actions report (follow the same HTML template defined in the actions skill — do not write a file to disk). Do not wait for the user to ask for it.

9. **Close with**: "Analysis complete. Here are the data quality actions I found. Would you like to work through the quick wins together? I'll walk you through each one and ask for your input."
