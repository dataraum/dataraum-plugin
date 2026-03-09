---
description: "Use when the user asks about data quality, entropy, uncertainty, reliability, contracts, compliance, readiness for use cases, resolution actions, or how to fix data problems. Trigger phrases: 'entropy', 'uncertainty', 'how reliable', 'data quality', 'contract', 'compliance', 'aggregation safe', 'executive dashboard', 'fix data', 'improve data', 'resolution actions', 'what should I fix', 'data issues'."
tools:
  - dataraum:get_quality
alwaysApply: false
---

# Quality Analysis

Unified data quality analysis — entropy, contract evaluation, and resolution actions — using the DataRaum `get_quality` MCP tool.

## How to Use

```
get_quality()
get_quality(include=["entropy"])
get_quality(include=["contract"], contract_name="aggregation_safe")
get_quality(include=["actions"], priority="high")
get_quality(contract_name="executive_dashboard", table_name="orders")
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `contract_name` | string | Optional: contract to evaluate (see table below) |
| `table_name` | string | Optional: filter to a specific table |
| `priority` | string | Optional: filter actions by "high", "medium", or "low" |
| `include` | list | Optional: sections to include — `["entropy", "contract", "actions"]`. Defaults to all three. |

### Available Contracts

| Contract | Purpose |
|----------|---------|
| `exploratory_analysis` | Safe for ad-hoc exploration and data discovery |
| `data_science` | Suitable for ML model training and feature engineering |
| `operational_analytics` | Ready for operational dashboards and monitoring |
| `aggregation_safe` | Safe for SUM, AVG, COUNT — no silent data loss |
| `executive_dashboard` | Ready for executive-level reporting |
| `regulatory_reporting` | Meets regulatory compliance requirements |

## What You Get

The tool returns a unified report with up to three sections:

### Entropy Section

Measures data uncertainty across four dimensions:

| Dimension | What It Measures |
|-----------|-----------------|
| **Structural** | Schema ambiguity: missing types, nullable columns, inconsistent formats |
| **Semantic** | Meaning ambiguity: unclear column roles, missing business context |
| **Value** | Data quality: nulls, outliers, distribution anomalies |
| **Computational** | Derivation uncertainty: calculation confidence, aggregation safety |

Scores: 0.0 (perfect) → 0.3 (low/ready) → 0.6 (medium/investigate) → 1.0 (high/remediate).

### Contract Section

- **Compliance Status**: PASS or FAIL
- **Overall Score**: 0.0 (worst) to 1.0 (perfect)
- **Dimension Scores**: Per-dimension breakdown against thresholds
- **Violations**: Specific rules that failed with affected columns

### Actions Section

Prioritized steps to fix data issues:

| Prefix | Meaning |
|--------|---------|
| `document_` | Add documentation or metadata |
| `investigate_` | Requires human review |
| `transform_` | Data transformation needed |
| `create_` | Create new artifact |

**Quick wins** = HIGH priority + LOW effort. Start here for immediate impact.

## Choosing Which Sections to Request

Match the user's intent to the right `include` selection:

| User intent | `include` | Extra params |
|-------------|-----------|--------------|
| "How's my data quality?" / general | all (default) | — |
| "Show me the entropy" / uncertainty | `["entropy"]` | `table_name?` |
| "Is my data aggregation safe?" / contract | `["contract"]` | `contract_name` |
| "What should I fix?" / actions | `["actions"]` | `priority?`, `table_name?` |
| "Entropy and what to fix" | `["entropy", "actions"]` | `table_name?` |

When the user asks about a specific contract, always set `contract_name`. If unsure which contract, recommend `aggregation_safe` as a good default.

## Response Pattern

**Resuming after skill load:** If this skill loaded mid-conversation, resume immediately — do not wait for the user to re-prompt. Check the conversation history and continue from where things left off.

1. **Check for existing data first**: Call `get_context` to see if data has already been analyzed.
   - If `get_context` returns schema/table information → data exists, skip to step 2
   - If `get_context` returns an error or "no sources found" → call `analyze` first with the user's data path, then continue
   - **Do not call `analyze` if data already exists in the database**
2. Call `get_quality` with the appropriate parameters based on user intent
3. Render the result as an **inline HTML artifact directly in your response** — do NOT write an HTML file to disk. Use the templates below, selecting the right template(s) based on which sections were included.

**Validation context:** If this evaluation is being run after a `document_` quick-wins session, frame the result as a before/after comparison. Check the conversation history for any prior score. If found, show the delta explicitly: "Before documentation: [score] → After: [new score]."

---

## HTML Output Templates

Generate a `<!DOCTYPE html>` artifact. Use only inline CSS — no external dependencies. When multiple sections are included, combine them in a single HTML document in order: entropy → contract → actions.

### Shared Style Block

Include this in `<head>` for all reports:

```html
<style>
  body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; padding: 20px; color: #1f2937; max-width: 960px; }
  h1 { font-size: 1.25rem; margin: 0 0 4px; }
  h2 { font-size: 1rem; margin: 16px 0 8px; }
  .card { border-radius: 8px; padding: 16px; margin-bottom: 12px; }
  .status-low    { background: #dcfce7; border-left: 4px solid #16a34a; }
  .status-medium { background: #fefce8; border-left: 4px solid #ca8a04; }
  .status-high   { background: #fee2e2; border-left: 4px solid #dc2626; }
  .verdict-pass  { background: #dcfce7; border-left: 6px solid #16a34a; }
  .verdict-fail  { background: #fee2e2; border-left: 6px solid #dc2626; }
  .header { background: #f9fafb; border: 1px solid #e5e7eb; }
  .section { background: #fff; border: 1px solid #e5e7eb; }
  .offender { background: #fff; border: 1px solid #e5e7eb; }
  .next-steps { background: #eff6ff; border-left: 4px solid #3b82f6; }
  .quick-win { background: #fefce8; border-left: 4px solid #ca8a04; }
  .high   { background: #fff1f2; border-left: 4px solid #e11d48; }
  .medium { background: #fffbeb; border-left: 4px solid #d97706; }
  .low    { background: #eff6ff; border-left: 4px solid #3b82f6; }
  .summary { display: flex; gap: 12px; flex-wrap: wrap; margin-top: 8px; }
  table { border-collapse: collapse; width: 100%; margin-top: 8px; font-size: 0.875rem; }
  th, td { padding: 8px 10px; text-align: left; border-bottom: 1px solid #e5e7eb; vertical-align: top; }
  th { background: #f9fafb; font-weight: 600; }
  tr:last-child td { border-bottom: none; }
  tr.row-fail { background: #fff1f2; }
  tr.row-pass { background: #f0fdf4; }
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
  .pill-red    { background: #fee2e2; color: #dc2626; }
  .pill-yellow { background: #fef9c3; color: #b45309; }
  .pill-blue   { background: #dbeafe; color: #1d4ed8; }
  .pill-green  { background: #dcfce7; color: #16a34a; }
  .pill-gray   { background: #f3f4f6; color: #6b7280; }
  .badge { display: inline-block; padding: 3px 10px; border-radius: 12px; font-size: 0.8rem; font-weight: 700; }
  .badge-pass { background: #dcfce7; color: #16a34a; }
  .badge-fail { background: #fee2e2; color: #dc2626; }
  .cell-low    { color: #16a34a; font-weight: 500; }
  .cell-medium { color: #b45309; font-weight: 500; }
  .cell-high   { color: #dc2626; font-weight: 500; }
  .score-large { font-size: 1.1rem; font-weight: 700; }
  code { background: #f3f4f6; padding: 1px 5px; border-radius: 4px; font-size: 0.85em; }
  .muted { color: #6b7280; font-size: 0.85rem; margin-top: 12px; }
  .legend { display: flex; gap: 12px; font-size: 0.8rem; margin-top: 6px; }
</style>
```

---

### Entropy Template

Use when the response includes an entropy section.

1. **Header card** (`.card.header`): Title "Entropy Report", optional "Table: name" if filtered. Overall status as a pill (`.pill-low/medium/high`) with composite score. One-line legend: `LOW < 0.3  MEDIUM 0.3-0.6  HIGH > 0.6`.

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

3. **Top Offenders section** — only for columns with composite >= 0.3. One `.card.offender` per column:
   - Heading: `<code>column_name</code>` + status pill
   - One sentence naming the primary driver dimension (highest individual score) and its value
   - One sentence connecting to a practical implication (e.g., "aggregation risk", "join failures may be silent", "temporal analysis blocked")
   - If all columns are below 0.3, replace with a single line: "All columns have low entropy — data is ready for analysis."

---

### Contract Template

Use when the response includes a contract section.

1. **Verdict card** (`.card.verdict-pass` or `.card.verdict-fail`):
   - `<h1>` with PASS or FAIL + contract name (e.g., `PASS — aggregation_safe`)
   - Overall score as a CSS progress bar (bar width = `score * 100%`, color green if PASS / red if FAIL) with the numeric score beside it
   - Muted sub-line: "Score: X.XX / 1.00"

   **Example (fail):**
   ```html
   <div class="card verdict-fail">
     <h1>FAIL — <code>aggregation_safe</code></h1>
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
   - Progress column: CSS bar scaled to 0.0-1.0 (not to threshold), colored green/red/yellow by whether it meets threshold
   - Status column: `<span class="badge badge-pass">PASS</span>` or `<span class="badge badge-fail">FAIL</span>`

3. **Violations section** (`.card.section`) — only if there are violations:
   - Sub-heading "Critical" for violations that caused a dimension to fail its threshold. Table columns: Rule | Affected Columns | Detail
   - Sub-heading "Warnings" for violations where the dimension still passed. Same table columns. Omit if no warnings.
   - If no violations at all, replace with: `<p>No violations found.</p>`

4. **Next Steps card** (`.card.next-steps`):
   - If PASS: "Data meets `contract_name` requirements. Consider evaluating `executive_dashboard` for higher-stakes reporting."
   - If FAIL: "Call `get_quality(include=[\"actions\"], priority=\"high\")` to get targeted fixes. Focus on **[dimension]** first — largest gap from threshold ([score] vs required [threshold])."

---

### Actions Template

Use when the response includes an actions section.

1. **Header card** (`.card.header`): Title "Resolution Actions", total count, and three pills showing `N HIGH`, `M MEDIUM`, `L LOW`.

2. **Quick Wins section** (`.card.quick-win`) — only if any HIGH priority + LOW effort actions exist:
   - Section heading "Quick Wins — High Priority, Low Effort"
   - Table with columns: Action | Effort | Column | What to do | Expected Impact

3. **Priority group sections** — one `.card` per group that has actions, in order HIGH → MEDIUM → LOW. Skip empty groups entirely.
   - Heading: "HIGH Priority" / "MEDIUM Priority" / "LOW Priority"
   - Table with columns: Action | Effort | Column | What to do | Expected Impact
   - Render `Effort` as a pill: LOW=`.pill-green`, MEDIUM=`.pill-yellow`, HIGH=`.pill-red`
   - Render action name as `<code>`
   - Within each group, sort by effort ascending (LOW first)

**"What to do" column rules:**
- Populate from the action's `parameters` field in the tool response
- Translate any raw machine strings to plain language (e.g. `unit_declaration=high` → "Declare the unit for this column (e.g. %, EUR, kg)")
- If `parameters` contains a specific instruction string, use it verbatim
- Examples by action type:
  - `document_unit` → "Declare the unit for `column_name` (e.g. %, EUR, kg)"
  - `document_null_semantics` → "Declare what null values mean in `column_name`"
  - `document_business_rule` → the rule description from parameters
  - `transform_winsorize` → "Cap extreme values at [threshold] percentile"
  - `investigate_*` → "Human review required: [description from parameters]"

4. **Footer** (`.muted`): "Suggested remediation order: Quick Wins → HIGH + LOW effort → HIGH + MEDIUM effort → MEDIUM priority"

**Example row:**
```html
<tr>
  <td><code>document_unit</code></td>
  <td><span class="pill pill-green">LOW</span></td>
  <td><code>kontobewegung_steuersatz</code></td>
  <td>Declare the unit for this column (e.g. %, EUR, kg)</td>
  <td>Removes aggregation uncertainty for tax rate fields</td>
</tr>
```

---

## Handling `document_` Quick Wins Interactively

When the user agrees to work through quick wins, guide them through each `document_` action one at a time (highest priority + lowest effort first):

**For each `document_` action:**

1. Present the column and the issue in plain language:
   > **`column_name`** — needs a [unit / null meaning / business rule] declaration.

2. **Use the `AskUserQuestion` tool** to present a native interactive multiple-choice prompt — never ask open-ended questions in plain text. Generate 3-4 options based on what you know about the column (name, data, entropy findings, source system if detected). The last option should always allow free text input.

   Call `AskUserQuestion` with:
   - `question`: the specific question for this column (e.g. "What unit does `kontobewegung_steuersatz` represent?")
   - `header`: short label shown as a chip (e.g. "Unit", "Null meaning", "Business rule")
   - `options`: 3-4 choices, each with a `label` and a `description`. Always include a final option with label "Other" and description "I'll type my own value".
   - `multiSelect: false`

   Rules for generating options by action type:
   - `document_unit`: list the 2-3 most likely units with examples (e.g. label "%" description "Percentage — e.g. 19.0 = 19%"). Put your best guess first.
   - `document_null_semantics`: list likely meanings (e.g. "Not applicable", "Not yet captured", "Zero / no value")
   - `document_business_rule`: option A = the rule from `parameters`, option B = a simplified plain-language variant

3. Wait for the user's selection. If they choose "Other", ask them to type their value in a follow-up message.

4. Acknowledge and move on:
   > Noted: `column_name` = [confirmed value]. Moving to next...

5. After all `document_` quick wins: summarize what was captured in a short list, then automatically transition to contract validation — do not wait for the user to ask.

   Say: "You've just answered [N] questions about your data. Let's see if that moved the needle — I'll run the quality evaluation now to show your updated readiness score."

   Then immediately call `get_quality(include=["contract"], contract_name="aggregation_safe")` and render the inline HTML contract report.

   Frame it as a before/after: if a prior contract score appeared earlier in the conversation, say "Before: FAIL (0.12) → Let's see where we are now." If no prior score is known, say "Here's your current readiness score after the documentation work."

   After showing the contract result, follow up based on the outcome:
   - **PASS**: "Your data is now aggregation-safe. Ready to run some queries? I can hand you validated SQL your team can reuse."
   - **FAIL but score improved vs. before**: "You've made progress — score moved from [before] to [after]. The remaining gaps are [failing dimensions]. Want to tackle the next action group, or run a query now with what you have?"
   - **No change / same score**: "The score didn't change yet — documentation annotations may need a pipeline re-run to take effect, or the remaining gaps need transform_ actions. Want to re-run the pipeline to pick up the new annotations?"

**Rules:**
- Handle one action at a time — do not present all at once
- Always propose a value; never just ask "what should this be?"
- Do not auto-apply any changes, write files, or run Python
- `transform_`, `investigate_`, and `create_` actions are out of scope for now — do not silently skip them; add them to the Open Items list below

## Open Items

Maintain a running list of unresolved items throughout the session. Add to it from:
- All `investigate_` actions (require human review — cannot be auto-resolved)
- User decisions deferred with "I'll check later" or similar
- Ambiguous values that couldn't be resolved in the `document_` workflow

After completing the `document_` quick wins, always show the current Open Items list:

```
## Open Items

1. **`column_name`** — [what needs investigating, from parameters field] — *open*
2. **`other_column`** — [what the user needs to decide] — *open*
```

Then ask: "Want to note who should look into any of these, or shall I remind you next time we look at this data?"

As items get resolved during conversation, update their status inline:
`— *resolved: [what was decided]*`
