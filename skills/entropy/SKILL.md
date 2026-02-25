---
description: "Use when the user asks about data entropy, uncertainty, reliability, or wants to understand how trustworthy their data is across different dimensions. Trigger phrases: 'entropy', 'uncertainty', 'how reliable', 'data quality dimensions', 'structural entropy', 'semantic entropy', 'value entropy'."
tools:
  - dataraum:get_entropy
alwaysApply: false
---

# Entropy Analysis

Get entropy analysis showing data uncertainty by dimension using the DataRaum MCP tool.

## How to Use

Call the `get_entropy` MCP tool:

```
get_entropy()
get_entropy(table_name="orders")
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `table_name` | string | Optional: filter to a specific table |

## What You Get

Entropy measures data uncertainty across four dimensions:

### Dimensions

| Dimension | What It Measures |
|-----------|-----------------|
| **Structural** | Schema ambiguity: missing types, nullable columns, inconsistent formats |
| **Semantic** | Meaning ambiguity: unclear column roles, missing business context |
| **Value** | Data quality: nulls, outliers, distribution anomalies |
| **Computational** | Derivation uncertainty: calculation confidence, aggregation safety |

### Scores

- **0.0**: Perfect certainty (no ambiguity)
- **0.0-0.3**: Low entropy (ready for analysis)
- **0.3-0.6**: Medium entropy (investigate before using)
- **0.6-1.0**: High entropy (needs remediation)

### Composite Score

Each column gets a weighted composite score across all dimensions. Higher scores mean more uncertainty and less reliable analysis results.

## Response Pattern

**Resuming after skill load:** If this skill loaded mid-conversation, resume immediately — do not wait for the user to re-prompt. Check the conversation history and continue from where things left off.

1. **Check for existing data first**: Call `get_context` to see if data has already been analyzed.
   - If `get_context` returns schema/table information → data exists, skip to step 2
   - If `get_context` returns an error or "no sources found" → call `analyze` first with the user's data path, then continue
   - **Do not call `analyze` if data already exists in the database**
2. Call the `get_entropy` tool (optionally filtered to a table)
3. Render the result as an **inline HTML artifact directly in your response** — do NOT write an HTML file to disk. Use the template below.

### Score Thresholds

| Range | Status | Color |
|-------|--------|-------|
| 0.0–0.3 | LOW — ready | green |
| 0.3–0.6 | MEDIUM — investigate | yellow |
| 0.6–1.0 | HIGH — needs remediation | red |

### HTML Output Template

Generate a `<!DOCTYPE html>` artifact. Use only inline CSS — no external dependencies.

**Style block to include in `<head>`:**

```html
<style>
  body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; padding: 20px; color: #1f2937; max-width: 960px; }
  h1 { font-size: 1.25rem; margin: 0 0 4px; }
  h2 { font-size: 1rem; margin: 16px 0 8px; }
  .card { border-radius: 8px; padding: 16px; margin-bottom: 12px; }
  .status-low    { background: #dcfce7; border-left: 4px solid #16a34a; }
  .status-medium { background: #fefce8; border-left: 4px solid #ca8a04; }
  .status-high   { background: #fee2e2; border-left: 4px solid #dc2626; }
  .header { background: #f9fafb; border: 1px solid #e5e7eb; }
  .offender { background: #fff; border: 1px solid #e5e7eb; }
  table { border-collapse: collapse; width: 100%; margin-top: 8px; font-size: 0.875rem; }
  th, td { padding: 8px 10px; text-align: left; border-bottom: 1px solid #e5e7eb; }
  th { background: #f9fafb; font-weight: 600; }
  tr:last-child td { border-bottom: none; }
  .cell-low    { color: #16a34a; font-weight: 500; }
  .cell-medium { color: #b45309; font-weight: 500; }
  .cell-high   { color: #dc2626; font-weight: 500; }
  .bar-wrap { display: flex; align-items: center; gap: 8px; }
  .bar-bg { background: #e5e7eb; border-radius: 4px; height: 8px; width: 80px; flex-shrink: 0; }
  .bar-fill { height: 8px; border-radius: 4px; }
  .bar-green  { background: #16a34a; }
  .bar-yellow { background: #ca8a04; }
  .bar-red    { background: #dc2626; }
  .pill { padding: 2px 8px; border-radius: 12px; font-size: 0.75rem; font-weight: 600; display: inline-block; }
  .pill-low    { background: #dcfce7; color: #16a34a; }
  .pill-medium { background: #fef9c3; color: #b45309; }
  .pill-high   { background: #fee2e2; color: #dc2626; }
  code { background: #f3f4f6; padding: 1px 5px; border-radius: 4px; font-size: 0.85em; }
  .muted { color: #6b7280; font-size: 0.85rem; margin-top: 12px; }
  .legend { display: flex; gap: 12px; font-size: 0.8rem; margin-top: 6px; }
</style>
```

**Structure:**

1. **Header card** (`.card.header`): Title "Entropy Report", optional "Table: name" if filtered. Overall status as a pill (`.pill-low/medium/high`) with composite score. One-line legend: `✅ LOW < 0.3  ⚠️ MEDIUM 0.3–0.6  ❌ HIGH > 0.6`.

2. **Dimension scores table** — one row per column (cap at top 10 by composite score descending; add a note if truncated):
   - Columns: Column | Structural | Semantic | Value | Computational | Composite
   - Each of the 4 dimension cells: colored value using `.cell-low/medium/high` based on score range
   - Composite cell: a CSS progress bar (`.bar-bg` + `.bar-fill` sized as `score * 100%`) plus the numeric score; color the bar green/yellow/red by range
   - Sort rows by composite score descending (worst at top)

   **Example row:**
   ```html
   <tr>
     <td><code>created_at</code></td>
     <td class="cell-high">0.71</td>
     <td class="cell-medium">0.55</td>
     <td class="cell-low">0.20</td>
     <td class="cell-medium">0.50</td>
     <td>
       <div class="bar-wrap">
         <div class="bar-bg"><div class="bar-fill bar-red" style="width:65%"></div></div>
         <span class="cell-high">0.65</span>
       </div>
     </td>
   </tr>
   ```

3. **Top Offenders section** — only for columns with composite ≥ 0.3. One `.card.offender` per column:
   - Heading: `<code>column_name</code>` + status pill
   - One sentence naming the primary driver dimension (highest individual score) and its value
   - One sentence connecting to a practical implication (e.g., "aggregation risk", "join failures may be silent", "temporal analysis blocked")
   - If all columns are below 0.3, replace with a single ✅ line: "All columns have low entropy — data is ready for analysis."

4. **Footer** (`.muted`): "Run `get_actions` to get prioritized steps for improving high-entropy columns."

   If a `table_name` filter was applied, note it in the header: "Entropy Report — Table: orders"
