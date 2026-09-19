# Testing in a real browser

WebMCP tools are an API for agents, so test them the way an agent would call
them.

## With Chrome DevTools MCP

Chrome DevTools MCP has two WebMCP tools, `list_webmcp_tools` and
`execute_webmcp_tool`. They are **off by default**: start the server with the
flag `--categoryExperimentalWebmcp` (also written `--categoryExperimentalWebmcp=true`).
Example registration in Claude Code:

```bash
claude mcp add chrome-devtools -- npx chrome-devtools-mcp@latest --categoryExperimentalWebmcp
```

You also need a Chrome build where WebMCP is enabled (Canary or Dev is
usually easiest; check current flags).

Procedure:

1. Open the page under test.
2. `list_webmcp_tools` (needs the `pageId`). Confirm the names, descriptions
   and schemas match the approved proposal, and that nothing extra or stale
   is present.
3. For each tool call `execute_webmcp_tool` with `toolName` and `input`, where
   `input` is a **JSON string**, e.g. `"{\"count\":2}"`.
4. Check the outcome by type:
   - form annotation: fields filled, highlight visible, submit behaves as
     designed
   - custom tool: the agreed screen or state change happened
   - bridge: the right data came back; no page change expected
5. Change route, sign out and back in, and list again. Tools should come and go
   as designed.
6. Fix and re-run until each proposal row passes.

## Without it

Use the console:

```js
const mc = document.modelContext;
const tools = await mc.getTools();
console.table(tools.map(({ name, description }) => ({ name, description })));
await mc.executeTool(tools.find((t) => t.name === "search_shows"), { text: "jazz" });
```

## Failure guide

| Symptom | Likely cause |
|---|---|
| No tools listed | API missing, polyfill not initialized before registration, flag not enabled, tool not mounted on this route |
| DevTools MCP has no WebMCP tools at all | server started without `--categoryExperimentalWebmcp` |
| Tool vanishes after navigation | aborted on unmount and never re-registered |
| Registration error | duplicate name, empty description, or invalid schema |
| Execute fails | input JSON malformed, schema mismatch, thrown error in `execute` |
| Empty or failed result | return value not serializable, unhandled rejection |
| Bridge shows no tools | CORS blocked, wrong MCP URL, auth rejected |

## Finish line

- [ ] pages still work in a browser with no WebMCP
- [ ] every proposal row was executed successfully by an agent-style call
- [ ] every write tool was run once and its real side effect confirmed
- [ ] aborting or leaving a page removes its tools from the list
- [ ] error messages tell the agent how to correct the call
