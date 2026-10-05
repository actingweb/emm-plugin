# Shared Memories from Connections

Detailed guidance on accessing and working with memories shared by trusted connections.

## Overview

Emm lets you access memories shared by trusted connections. Connections can be:
- **People** (family, colleagues) who have their own Emm AI account
- **AI agents** (e.g., ChatGPT, other Claude instances) connected to the same user account

Trust relationships and sharing preferences are configured via the web dashboard.

## Search Remote Memories

```
memory_search(query="vacation plans", include_remote=true)
```

## Filter by Connection

```
memory_search(query="allergies", include_remote=true, source_connection="Alice")
```

## Filter Remote Types

Search all local memories but restrict remote results to specific categories:

```
memory_search(query="trip", include_remote=true, source_connection="spouse", remote_types="travel,food")
```

## Discover Connections

Use `list_connections()` to see who shares memories with you, what types they share, and whether they also offer remote actions. By default the response only includes connections that authenticated within the last 30 days; pass `include_stale=true` for an audit view that includes dormant pairings. The envelope carries `recency_window_days` (30) and `stale_filtered_count` so you can see whether anyone was hidden.

## Best Practices

- `memory_search` performs no consent check when `include_remote=true` — the
  tool description says so, but asking is still on you. Ask once per
  conversation, the first time you would set `include_remote=true`, phrased
  as a yes/no ("Want me to check what people you've shared with know about
  this?"). Remember the answer for the rest of the session; re-ask next
  session. Skip the ask if the user named a connection in their prompt —
  that's implicit consent.
- Attribute shared memories to their source naturally (e.g., "Alice mentioned she prefers...")
- Remote memories are read-only — you cannot modify another person's memories
- A connection may share memories, offer remote actions, or both
