# Figma MCP — Known Failure Signatures

## Connection-layer

| Signature | Cause | Fix |
|---|---|---|
| `{"jsonrpc":"2.0","error":{"code":-32001,"message":"Invalid sessionId"}}` | Stale endpoint config | Remove/re-add MCP server config, use current `/mcp` endpoint not deprecated `/sse` |
| Tool list only shows ~4-10 tools, `use_figma` missing | Account-side feature-flag bug | Reconnect/re-auth; server-side issue if persistent |
| "This figma file could not be accessed" | Auth account not on same team/plan as file owner | Run `whoami`, confirm access — don't retry, escalate |
| 404 on `localhost:3845/assets/...` | Local asset server not serving | Flag it, don't fabricate a placeholder |
| Tool call hangs indefinitely | Large selection, or annotations present | Try smaller selection; annotations are the likely cause |

## Silent quality failures (no error thrown — these are the dangerous ones)

| Signature | Cause | What it means |
|---|---|---|
| `get_code`/`get_design_context` "succeeds" but content is generic/plausible rather than matching the frame | No Code Connect on this account tier — tool silently falls back to screenshot internally | Don't trust its output at all; lean entirely on `get_metadata`/`get_variable_defs` |
| `get_code` fails specifically on selections with designer annotations | Reproducible bug | Re-select without annotations |
| `get_variable_defs` returns empty/sparse on a frame that obviously uses the design system | Selection scoped too narrow, or variables not shared to your access level | Try the parent frame; don't assume no tokens exist |
| A value "looks about right" but isn't an exact match to any variable | Rounding to nearest familiar value instead of the real one | Never treat proximity as a match — flag it as unresolved |

## Rate limits

- Free/Starter or View/Collab seats: ~6 tool calls/month via desktop MCP.
- Dev/Full paid seats: per-minute limits matching Figma Tier 1 REST API. Batch calls per frame.

## Debugging order

1. `whoami` first.
2. Smaller selection to isolate size/complexity issues.
3. Restart Figma desktop + MCP client.
4. Only then treat it as a genuine data-quality issue.
