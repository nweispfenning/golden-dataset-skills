# Incident-analysis agent: planned tools (design doc, 2026-09)

The agent doesn't exist yet. These are the tools in its design. Every tool is read-only, and the agent has no tool that changes anything.

| Tool | Returns |
|---|---|
| `list_deploys(service, start, end)` | deploys to the service in the window: time, version, deploying team, change notes |
| `list_flag_changes(service, start, end)` | feature-flag changes for the service in the window: time, flag, old and new value, team |
| `query_metric(service, metric, start, end)` | the metric's values in the window, and when it first left its normal range |
| `search_runbooks(query)` | matching runbook sections |
| `get_owner(service)` | the owning team and its on-call, from the service registry |
