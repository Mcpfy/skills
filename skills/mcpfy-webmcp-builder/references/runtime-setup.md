# Runtime setup

## What exists in the browser

Per the draft spec, `document.modelContext` is available in secure contexts and
offers `registerTool(tool, options)`, `getTools(options)`, and
`executeTool(tool, input, options)`, plus a `toolchange` event.
`navigator.modelContext` is a deprecated alias; earlier drafts also had
`provideContext`, `clearContext` and `unregisterTool`, which new code should not
use.

Support is limited and moving. Before telling the user which browsers work,
check https://github.com/webmachinelearning/webmcp/blob/main/implementation-status.md
and Chrome's current WebMCP page. Expect origin-trial or flag-gated behaviour.

## Guard everything

Never let a missing API break the page:

```js
// Returns the ModelContext object, or undefined when the browser has none.
export const modelContext = () =>
  globalThis.document?.modelContext ?? globalThis.navigator?.modelContext;

export const canRegister = () => typeof modelContext()?.registerTool === "function";
```

When it is unavailable, skip registration quietly (one log line at most). The
normal UI must keep working.

## Polyfills

Only if the user wants tools in browsers without native support.

```bash
npm install @mcp-b/webmcp-polyfill
```

```ts
import { initializeWebMCPPolyfill } from "@mcp-b/webmcp-polyfill";
initializeWebMCPPolyfill();   // once, before any registerTool or bridge call
```

`@mcp-b/global` is a broader package from the same maintainers. Prefer the
smaller one. Pin versions and bundle the polyfill yourself rather than loading
it from a third-party CDN.

Form annotations alone need no JavaScript and no polyfill.

## Frames

Tools are same-origin by default. To use them across origins:

- the embedding page gives the frame permission:
  `<iframe src="https://widgets.example" allow="tools"></iframe>`
- the frame registers with the parent's origin allowed:
  `registerTool(tool, { signal, exposedTo: ["https://app.example"] })`
- a caller can filter with `getTools({ fromOrigins: [...] })`

Only secure origins are valid in those lists. Never expose privileged tools to
an origin you do not control.
