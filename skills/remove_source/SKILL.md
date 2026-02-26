---
description: "Use when the user wants to remove a data source, unregister a source, delete a connection, or archive a source. Trigger phrases: 'remove source', 'delete source', 'unregister', 'archive source', 'disconnect'."
tools:
  - dataraum:remove_source
  - dataraum:discover_sources
alwaysApply: false
---

# Remove Source

Archive or delete a registered data source using the DataRaum MCP tool.

## How to Use

```
remove_source(name="sales")
remove_source(name="old_data", purge_results=true)
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | **Required**: name of the source to remove. |
| `purge_results` | boolean | Optional: also delete stored analysis results. Default: false (archive only). |

## Archive vs Purge

- **Default (archive)**: The source is unregistered but analysis history is preserved. The source can be re-added later and previous results remain available for comparison.
- **Purge** (`purge_results=true`): The source and all its analysis results are permanently deleted. Use this when the data is no longer relevant and you want to clean up.

## Response Pattern

**Resuming after skill load:** If this skill loaded mid-conversation, resume immediately — do not wait for the user to re-prompt. Check the conversation history and continue from where things left off.

1. Confirm the source name with the user if ambiguous. If the user says "remove the source" without a name, call `discover_sources()` to list registered sources and ask which one to remove.
2. Ask about purge preference if the user hasn't specified: "Should I preserve the analysis history (default) or permanently delete it too?"
3. Call `remove_source(name="...", purge_results=...)`.
4. Confirm: "Removed **name**. Analysis history [preserved / deleted]."
5. If the source set changed and the user has remaining sources, suggest: "You may want to re-run `analyze()` to update results without this source."
