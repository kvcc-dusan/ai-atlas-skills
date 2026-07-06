# Figma MCP — Known Failure Signatures

v2 — merged with Figma's own published Q&A troubleshooting docs
(developers.figma.com/docs/figma-mcp-server/) as of mid-2026. Original signatures
preserved; additions marked `[new]`.

## Connection-layer

| Signature | Cause | Fix |
|---|---|---|
| `{"jsonrpc":"2.0","error":{"code":-32001,"message":"Invalid sessionId"}}` | Stale endpoint config | Remove/re-add MCP server config, use current `/mcp` endpoint not deprecated `/sse` |
| Tool list only shows ~4-10 tools, `use_figma` missing | Account-side feature-flag bug | Reconnect/re-auth; server-side issue if persistent |
| "This figma file could not be accessed" | Auth account not on same team/plan as file owner | Run `whoami`, confirm access — don't retry, escalate |
| 404 on `localhost:3845/assets/...` | Local asset server not serving | Flag it, don't fabricate a placeholder |
| Tool call hangs indefinitely | Large selection, or annotations present | Try smaller selection; annotations are the likely cause |
| **[new]** 500 error on any tool call | Transient server-side issue, or a malformed/oversized request | Retry once; if it repeats, reduce selection size before assuming it's a data problem |
| **[new]** Tools aren't loading / "connection lost" | Client-side MCP registration issue, not a Figma-side outage | Restart the MCP client, re-check the server URL, confirm the client version supports the tool set you expect |
| **[new]** Selection-based prompt returns nothing on remote server | Remote server has no concept of "current selection" — only local desktop server does | Always pass an explicit node-id URL when using the remote server |

## Silent quality failures (no error thrown — these are the dangerous ones)

| Signature | Cause | What it means |
|---|---|---|
| `get_design_context` "succeeds" but content is generic/plausible rather than matching the frame | No Code Connect on this account tier — tool silently falls back to screenshot internally | Don't trust its output at all; lean entirely on `get_metadata`/`get_variable_defs` |
| `get_design_context` fails specifically on selections with designer annotations | Reproducible bug | Re-select without annotations |
| `get_variable_defs` returns empty/sparse on a frame that obviously uses the design system | Selection scoped too narrow, or variables not shared to your access level | Try the parent frame; don't assume no tokens exist |
| A value "looks about right" but isn't an exact match to any variable | Rounding to nearest familiar value instead of the real one | Never treat proximity as a match — flag it as unresolved |
| **[new]** You asked for variables/tokens, agent returned code instead | Tool auto-routing picked the wrong tool for a vague prompt — this is a documented, acknowledged Figma behavior, not a one-off bug | Be explicit: "get the variable names and values for this selection," don't rely on the model inferring which tool to call |
| **[new]** `get_code_connect_map` returns empty on a component you know has a mapping | `clientFrameworks`/`clientLanguages` params scoped to the wrong framework label (e.g. only `React` requested when the mapping is under `SwiftUI`) | Confirm which Code Connect label(s) are set up before assuming "no mapping exists" |

## Rate limits

- Free/Starter or View/Collab seats: ~6 tool calls/month via desktop MCP.
- Dev/Full paid seats: per-minute limits matching Figma Tier 1 REST API. Batch calls per frame.
- **[new]** `generate_diagram` and `generate_figma_design` are currently exempt from
  standard rate limits during their beta period — don't assume the same budget applies
  to every tool uniformly.

## Debugging order

1. `whoami` first (remote server only — confirms auth/plan/seat).
2. Confirm remote vs. desktop server, and that selection-based prompting is only being
   relied on with the desktop server.
3. Smaller selection to isolate size/complexity issues — this fixes more "stuck or
   slow" reports than anything else.
4. Restart Figma desktop + MCP client.
5. Only then treat it as a genuine data-quality issue.

## Sources

- Figma MCP Server Q&A: developers.figma.com/docs/figma-mcp-server/ (tools-not-loading,
  stuck-or-slow, variables-vs-code, getting-500-error, mcp-clients-issues,
  images-stopped-loading, rate-limits-access)
- Figma MCP Server — Avoid selecting large, heavy frames
- Original signature table from this file's v1 (internal testing, INOVA IT)
