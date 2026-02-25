---
description: "Use when the user asks about data contracts, compliance, readiness for specific use cases, or whether data meets quality thresholds. Trigger phrases: 'contract', 'compliance', 'data ready for', 'aggregation safe', 'executive dashboard', 'can I use this data for', 'quality thresholds'."
tools:
  - dataraum:evaluate_contract
alwaysApply: false
---

# Contract Evaluation

Evaluate data quality against a named contract using the DataRaum MCP tool.

## How to Use

Call the `evaluate_contract` MCP tool:

```
evaluate_contract(contract_name="aggregation_safe")
evaluate_contract(contract_name="executive_dashboard")
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `contract_name` | string | **Required**: contract to evaluate against |

### Available Contracts

| Contract | Purpose |
|----------|---------|
| `aggregation_safe` | Safe for SUM, AVG, COUNT — no silent data loss |
| `executive_dashboard` | Ready for executive-level reporting |
| `ml_training` | Suitable for machine learning model training |
| `regulatory_reporting` | Meets regulatory compliance requirements |

## What You Get

- **Compliance Status**: PASS or FAIL
- **Overall Score**: 0.0 (worst) to 1.0 (perfect)
- **Dimension Scores**: Per-dimension breakdown against thresholds
- **Violations**: Specific rules that failed with affected columns
- **Threshold Requirements**: What each dimension needs to pass

## Response Pattern

**Resuming after skill load:** If this skill loaded mid-conversation, resume immediately — do not wait for the user to re-prompt. Check the conversation history and continue from where things left off.

**Validation context:** If this evaluation is being run after a `document_` quick-wins session (the user just worked through documentation actions), frame the result as a before/after comparison. Check the conversation history for any prior contract score. If found, show the delta explicitly in your response: "Before documentation: [score] → After: [new score]."

1. **Check for existing data first**: Call `get_context` to see if data has already been analyzed.
   - If `get_context` returns schema/table information → data exists, skip to step 3
   - If `get_context` returns an error or "no sources found" → call `analyze` first with the user's data path, then continue
   - **Do not call `analyze` if data already exists in the database**
2. If unsure which contract to use, recommend `aggregation_safe` as a good default
3. Call the `evaluate_contract` tool with the appropriate contract name
4. Render the result as an **inline HTML artifact directly in your response** — do NOT write an HTML file to disk. Use the template below.

### HTML Output Template

Generate a `<!DOCTYPE html>` artifact. Use only inline CSS — no external dependencies.

**Style block to include in `<head>`:**

```html
<style>
  body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; padding: 20px; color: #1f2937; max-width: 900px; }
  h1 { font-size: 1.5rem; margin: 0 0 6px; }
  h2 { font-size: 1rem; margin: 16px 0 8px; }
  .card { border-radius: 8px; padding: 16px; margin-bottom: 12px; }
  .verdict-pass { background: #dcfce7; border-left: 6px solid #16a34a; }
  .verdict-fail { background: #fee2e2; border-left: 6px solid #dc2626; }
  .section { background: #fff; border: 1px solid #e5e7eb; }
  .next-steps { background: #eff6ff; border-left: 4px solid #3b82f6; }
  .bar-wrap { display: flex; align-items: center; gap: 10px; }
  .bar-bg { background: #e5e7eb; border-radius: 4px; height: 10px; width: 100px; flex-shrink: 0; }
  .bar-fill { height: 10px; border-radius: 4px; }
  .bar-green  { background: #16a34a; }
  .bar-yellow { background: #ca8a04; }
  .bar-red    { background: #dc2626; }
  .badge { display: inline-block; padding: 3px 10px; border-radius: 12px; font-size: 0.8rem; font-weight: 700; }
  .badge-pass { background: #dcfce7; color: #16a34a; }
  .badge-fail { background: #fee2e2; color: #dc2626; }
  .badge-warn { background: #fef9c3; color: #b45309; }
  table { border-collapse: collapse; width: 100%; font-size: 0.875rem; margin-top: 8px; }
  th, td { padding: 8px 10px; text-align: left; border-bottom: 1px solid #e5e7eb; }
  th { background: #f9fafb; font-weight: 600; }
  tr:last-child td { border-bottom: none; }
  tr.row-fail { background: #fff1f2; }
  tr.row-pass { background: #f0fdf4; }
  code { background: #f3f4f6; padding: 1px 5px; border-radius: 4px; font-size: 0.85em; }
  .muted { color: #6b7280; font-size: 0.85rem; }
  .score-large { font-size: 1.1rem; font-weight: 700; }
</style>
```

**Structure:**

1. **Verdict card** (`.card.verdict-pass` or `.card.verdict-fail`):
   - `<h1>` with ✅ PASS or ❌ FAIL + contract name (e.g., `✅ PASS — aggregation_safe`)
   - Overall score as a CSS progress bar (bar width = `score * 100%`, color green if PASS / red if FAIL) with the numeric score beside it
   - Muted sub-line: "Score: X.XX / 1.00"

   **Example (fail):**
   ```html
   <div class="card verdict-fail">
     <h1>❌ FAIL — <code>aggregation_safe</code></h1>
     <div class="bar-wrap" style="margin-top:10px">
       <div class="bar-bg"><div class="bar-fill bar-red" style="width:43%"></div></div>
       <span class="score-large">0.43</span>
     </div>
     <p class="muted">Overall score: 0.43 / 1.00</p>
   </div>
   ```

2. **Dimension scorecard table** (`.card.section`):
   - Heading: "Dimension Scores vs. Thresholds"
   - Columns: Dimension | Score | Required | Progress | Status
   - Sort rows: failing dimensions first (worst score-gap first), then passing
   - Apply `class="row-fail"` or `class="row-pass"` to each `<tr>`
   - Progress column: CSS bar scaled to 0.0–1.0 (not to threshold), colored green/red/yellow by whether it meets threshold
   - Status column: `<span class="badge badge-pass">✅ PASS</span>` or `<span class="badge badge-fail">❌ FAIL</span>`

   **Example rows:**
   ```html
   <tr class="row-fail">
     <td>Semantic</td>
     <td>0.75</td>
     <td>≥ 0.80</td>
     <td><div class="bar-bg"><div class="bar-fill bar-red" style="width:75%"></div></div></td>
     <td><span class="badge badge-fail">❌ FAIL</span></td>
   </tr>
   <tr class="row-pass">
     <td>Structural</td>
     <td>0.88</td>
     <td>≥ 0.80</td>
     <td><div class="bar-bg"><div class="bar-fill bar-green" style="width:88%"></div></div></td>
     <td><span class="badge badge-pass">✅ PASS</span></td>
   </tr>
   ```

3. **Violations section** (`.card.section`) — only if there are violations:
   - Heading: "Violations"
   - Sub-heading "🔴 Critical" for violations that caused a dimension to fail its threshold. Table columns: Rule | Affected Columns | Detail
   - Sub-heading "🟡 Warnings" for violations where the dimension still passed. Same table columns. Omit this sub-section if there are no warnings.
   - If there are no violations at all (pure PASS with no warnings), replace entire section with: `<p>✅ No violations found.</p>`

4. **Next Steps card** (`.card.next-steps`):
   - If PASS: "Data meets `contract_name` requirements. Consider evaluating `executive_dashboard` for higher-stakes reporting."
   - If FAIL: "Run `get_actions(priority=\"high\")` to get targeted fixes. Focus on **[dimension]** first — largest gap from threshold ([score] vs required [threshold])."
