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

## Response Pattern

1. If no data has been analyzed yet, call `analyze` first with the user's data path
2. Call the `get_actions` tool (optionally filtered by priority or table)
3. Output a self-contained HTML artifact using the template below

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
   - Table with columns: Action | Type | Affected Columns | Expected Impact

3. **Priority group sections** — one `.card` per group that has actions, in order HIGH → MEDIUM → LOW. Skip empty groups entirely.
   - Heading: "🔴 HIGH Priority" / "🟡 MEDIUM Priority" / "🔵 LOW Priority"
   - Table with columns: Action | Effort | Affected Columns | Expected Impact
   - Render `Effort` as a pill: LOW=`.pill-green`, MEDIUM=`.pill-yellow`, HIGH=`.pill-red`
   - Render action name as `<code>`
   - Within each group, sort by effort ascending (LOW first)

4. **Footer** (`.muted`): "Suggested remediation order: Quick Wins → HIGH + LOW effort → HIGH + MEDIUM effort → MEDIUM priority"

**Example row:**
```html
<tr>
  <td><code>investigate_nulls_order_id</code></td>
  <td><span class="pill pill-green">LOW</span></td>
  <td><code>order_id</code></td>
  <td>Prevents silent join failures</td>
</tr>
```
