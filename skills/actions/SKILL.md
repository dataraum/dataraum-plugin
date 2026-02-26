---
description: "Use when the user asks about data quality issues, how to fix data problems, improvement recommendations, or resolution actions. Trigger phrases: 'fix data quality', 'improve data', 'resolution actions', 'what's wrong with the data', 'data issues', 'how do I fix', 'what should I fix first'."
tools:
  - dataraum:get_actions
alwaysApply: false
---

# Resolution Actions

Get prioritized actions to improve data quality using the DataRaum MCP tool.

## How to Use

Call the `get_actions` MCP tool:

```
get_actions()
get_actions(priority="high")
get_actions(table_name="orders")
get_actions(priority="high", table_name="orders")
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `priority` | string | Optional: filter by "high", "medium", or "low" |
| `table_name` | string | Optional: filter to actions affecting a specific table |

## What You Get

Resolution actions are concrete steps to fix data issues:

- **Priority**: high, medium, or low based on impact
- **Effort**: low, medium, or high (time/complexity to implement)
- **Affected Columns**: Which data is impacted
- **Expected Impact**: What will improve after fixing
- **Parameters**: Actionable details (thresholds, strategies, targets)

## Action Types

| Prefix | Meaning |
|--------|---------|
| `document_` | Add documentation or metadata |
| `investigate_` | Requires human review |
| `transform_` | Data transformation needed |
| `create_` | Create new artifact |

## Priority Levels

| Level | When to Address |
|-------|-----------------|
| **HIGH** | Fix immediately - blocking reliable analysis |
| **MEDIUM** | Address soon - affects confidence in results |
| **LOW** | Nice to have - minor improvements |

## Quick Wins

Actions marked as **high priority + low effort** are quick wins. Start with these for immediate impact with minimal work.

## Resuming After Skill Load

If this skill was just loaded in the middle of an ongoing workflow (e.g. the user had already asked about data quality or was partway through a quick-wins walkthrough), **resume immediately** — do not wait for the user to repeat themselves. Check the conversation history to understand where the workflow was and continue from that point:

- If the user already completed some `document_` actions → continue from the next unresolved action
- If the user asked for actions but none were shown yet → call `get_actions` and render the report
- If the actions report was already shown → ask if they want to work through the quick wins

## Response Pattern

1. **Check for existing data first**: Call `get_context` to see if data has already been analyzed.
   - If `get_context` returns schema/table information → data exists, skip to step 2
   - If `get_context` returns an error or "no sources found" → call `analyze` first with the user's data path, then continue
   - **Do not call `analyze` if data already exists in the database**
2. Call the `get_actions` tool (optionally filtered by priority or table)
3. Render the result as an **inline HTML artifact directly in your response** — do NOT write an HTML file to disk. Use the template below.

### HTML Output Template

Generate a `<!DOCTYPE html>` artifact. Use only inline CSS — no external dependencies.

**Style block to include in `<head>`:**

```html
<style>
  body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; padding: 20px; color: #1f2937; max-width: 900px; }
  h1 { font-size: 1.25rem; margin: 0 0 4px; }
  h2 { font-size: 1rem; margin: 16px 0 8px; }
  .card { border-radius: 8px; padding: 16px; margin-bottom: 12px; }
  .header { background: #f9fafb; border: 1px solid #e5e7eb; }
  .quick-win { background: #fefce8; border-left: 4px solid #ca8a04; }
  .high { background: #fff1f2; border-left: 4px solid #e11d48; }
  .medium { background: #fffbeb; border-left: 4px solid #d97706; }
  .low { background: #eff6ff; border-left: 4px solid #3b82f6; }
  .summary { display: flex; gap: 12px; flex-wrap: wrap; margin-top: 8px; }
  .pill { padding: 3px 10px; border-radius: 12px; font-size: 0.8rem; font-weight: 600; }
  .pill-red    { background: #fee2e2; color: #dc2626; }
  .pill-yellow { background: #fef9c3; color: #b45309; }
  .pill-blue   { background: #dbeafe; color: #1d4ed8; }
  .pill-green  { background: #dcfce7; color: #16a34a; }
  .pill-gray   { background: #f3f4f6; color: #6b7280; }
  table { border-collapse: collapse; width: 100%; margin-top: 8px; font-size: 0.875rem; }
  th, td { padding: 8px 10px; text-align: left; border-bottom: 1px solid #e5e7eb; vertical-align: top; }
  th { background: #f9fafb; font-weight: 600; }
  tr:last-child td { border-bottom: none; }
  code { background: #f3f4f6; padding: 1px 5px; border-radius: 4px; font-size: 0.85em; }
  .muted { color: #6b7280; font-size: 0.8rem; margin-top: 12px; }
</style>
```

**Structure:**

1. **Header card** (`.card.header`): Title "Resolution Actions", total count, and three pills showing `N 🔴 HIGH`, `M 🟡 MEDIUM`, `L 🔵 LOW`.

2. **Quick Wins section** (`.card.quick-win`) — only if any HIGH priority + LOW effort actions exist:
   - Section heading "⚡ Quick Wins — High Priority · Low Effort"
   - Table with columns: Action | Effort | Column | What to do | Expected Impact

3. **Priority group sections** — one `.card` per group that has actions, in order HIGH → MEDIUM → LOW. Skip empty groups entirely.
   - Heading: "🔴 HIGH Priority" / "🟡 MEDIUM Priority" / "🔵 LOW Priority"
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

## Handling `document_` Quick Wins Interactively

When the user agrees to work through quick wins, guide them through each `document_` action one at a time (highest priority + lowest effort first):

**For each `document_` action:**

1. Present the column and the issue in plain language:
   > **`column_name`** — needs a [unit / null meaning / business rule] declaration.

2. **Always present choices as a numbered list** — never ask open-ended questions. Generate 3–4 options based on what you know about the column (name, data, entropy findings, source system if detected), plus an "Other" option. Format like this:

   > **`kontobewegung_steuersatz`** — What unit should this tax rate column declare?
   >
   > **A)** % (percentage, e.g. 19.0 = 19%)
   > **B)** Decimal fraction (e.g. 0.19 = 19%)
   > **C)** Basis points (e.g. 1900 = 19%)
   > **D)** Other — I'll type it

   Rules for generating options:
   - `document_unit`: list the 2–3 most likely units based on column name and context, plus "Other — I'll type it" as the last option. Put your best guess first.
   - `document_null_semantics`: list the 2–3 most likely meanings (e.g. "Not applicable", "Not yet captured", "Zero / no value"), plus "Other — I'll describe it"
   - `document_business_rule`: list the suggested rule from `parameters` as option A, a simplified variant as B, then "Other — I'll describe it"

3. Wait for the user to reply with a letter or type their own value.

4. Acknowledge and move on:
   > ✅ Noted: `column_name` = [confirmed value]. Moving to next…

5. After all `document_` quick wins: summarize what was captured in a short list, then automatically transition to contract validation — do not wait for the user to ask.

   Say: "You've just answered [N] questions about your data. Let's see if that moved the needle — I'll run the contract evaluation now to show you your updated readiness score."

   Then immediately call `evaluate_contract(contract_name="aggregation_safe")` and render the inline HTML contract report.

   Frame it as a before/after: if a prior contract score appeared earlier in the conversation, say "Before: ❌ FAIL (0.12) → Let's see where we are now." If no prior score is known, say "Here's your current readiness score after the documentation work."

   After showing the contract result, follow up based on the outcome:
   - **PASS**: "✅ Your data is now aggregation-safe. Ready to run some queries? I can hand you validated SQL your team can reuse."
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
## 📋 Open Items

1. **`column_name`** — [what needs investigating, from parameters field] — *open*
2. **`other_column`** — [what the user needs to decide] — *open*
```

Then ask: "Want to note who should look into any of these, or shall I remind you next time we look at this data?"

As items get resolved during conversation, update their status inline:
`— *resolved: [what was decided]*`
