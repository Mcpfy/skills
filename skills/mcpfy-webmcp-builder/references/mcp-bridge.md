# Bridging an existing MCP server

If the product already runs a remote MCP server, reuse it. This includes any
server built with `mcpfy-sdk` that is reachable over Streamable HTTP or SSE.
The bridge connects from the browser, lists the server's tools, registers each
through `document.modelContext`, and forwards agent calls back to the server as
`tools/call`.

## Package

`webmcp-proxy` on npm does exactly this (verify the current version and README
before wiring):

```bash
npm install webmcp-proxy
```

```ts
import { createWebMcpProxy } from "webmcp-proxy";

const proxy = await createWebMcpProxy({
  url: "https://mcp.example.com/mcp",
  // headers: { Authorization: `Bearer ${token}` },  // only when the session cookie is not enough
});

// proxy.tools lists what was registered
// await proxy.disconnect() removes them
```

React and Vue wrappers exist as `webmcp-proxy/react` and `webmcp-proxy/vue`
(`<WebMCPProxy url="..." />`). If `document.modelContext` is missing, the proxy
logs a warning and does nothing, so the site still works.

Because it registers tools with `registerTool`, it coexists with hand-written
tools as long as names do not collide.

## Conversation to have before wiring

1. **Same OAuth client?** If the website and the MCP server share one, the
   user's existing session can authorize tool calls with no second login. If
   not, do not invent a hidden second login flow; ask the user how agents
   should authenticate.
2. **Where is the token?** httpOnly cookie, memory, or local storage decides
   whether you pass `headers` or rely on credentialed requests.
3. **CORS.** The MCP endpoint must allow the site's origin. The proxy runs in
   the browser, so a blocked preflight means zero tools.
4. **Which tools?** Everything the server offers will be listed. If some are
   admin-only or unsuitable for a browsing agent, restrict them on the server
   side before the bridge goes live.
5. **No visible feedback.** Calls do not fill fields or navigate. If the user
   wants on-screen behaviour for one flow, add a custom tool for it later and
   make sure the bridge does not also expose the same action.

## Never do this

- Paste long-lived secrets into front-end code.
- Copy each server tool into page code by hand.
- Bridge a server that is not meant to receive browser traffic.

## When you cannot use the package

The behaviour is small enough to reproduce: an MCP client connects, calls
`tools/list`, and for each tool calls `registerTool` with the same name,
description and schema and an `execute` that issues `tools/call` and returns
the result. Prefer the package unless there is a hard reason not to.

Then verify with [testing.md](testing.md). Expect results but no DOM changes.
