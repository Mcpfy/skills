# Framework recipes

The API is a browser API, so registration must happen in client-side code that
runs after the page exists, and must be torn down when the feature goes away.
The pattern is always: create an `AbortController`, register with its signal,
abort on teardown.

## Plain JavaScript / any SPA

Keep each feature's tools in one module that receives its dependencies and
returns a cleanup function:

```js
export function mountShowTools({ bookingService, router }) {
  const mc = document.modelContext ?? navigator.modelContext;
  if (!mc?.registerTool) return () => {};

  const lifetime = new AbortController();

  mc.registerTool(
    {
      name: "search_shows",
      description: "Find upcoming shows by title, genre or city.",
      inputSchema: {
        type: "object",
        properties: { text: { type: "string", description: "Title, genre or city" } },
        required: ["text"],
      },
      annotations: { readOnlyHint: true },
      execute: ({ text }, { signal }) => bookingService.search(text, { signal }).then(summarize),
    },
    { signal: lifetime.signal },
  ).catch((err) => console.warn("WebMCP registration failed:", err));

  return () => lifetime.abort();
}
```

Call the returned cleanup on route exit, sign-out, or app teardown.

## React

Two options.

**Direct effect** (no extra dependency):

```tsx
"use client";
import { useEffect } from "react";

export function ShowTools({ search }: { search: (q: string, s: AbortSignal) => Promise<string> }) {
  useEffect(() => {
    const mc = document.modelContext;
    if (!mc?.registerTool) return;
    const lifetime = new AbortController();
    mc.registerTool(
      {
        name: "search_shows",
        description: "Find upcoming shows by title, genre or city.",
        inputSchema: {
          type: "object",
          properties: { text: { type: "string", description: "Title, genre or city" } },
          required: ["text"],
        },
        annotations: { readOnlyHint: true },
        execute: ({ text }, { signal }) => search(text, signal),
      },
      { signal: lifetime.signal },
    ).catch(console.warn);
    return () => lifetime.abort();
  }, [search]);
  return null;
}
```

**`usewebmcp` hook.** A small package (`npm install usewebmcp`) whose
`useWebMCP({ name, description, inputSchema, execute, enabled })` registers on
mount and unregisters on unmount; pass `enabled` for conditional tools. It
expects `document.modelContext` to exist already (native or polyfill). Its
`execute` takes only the input, so use the direct effect if you need the
cancellation signal.

Either way: register only from client components, and mount the component
where the tool is valid.

## Next.js (App Router)

- WebMCP is a document API. Never call `registerTool` from Server Components
  or Route Handlers.
- Put polyfill initialization and any always-on tools in one small client
  component rendered from the root layout.
- Route-specific tools go in client components inside that route's tree so
  they unmount on navigation.

## Vue

```vue
<script setup>
import { onMounted, onBeforeUnmount } from "vue";
let lifetime;
onMounted(() => {
  const mc = document.modelContext;
  if (!mc?.registerTool) return;
  lifetime = new AbortController();
  mc.registerTool({ /* descriptor */ }, { signal: lifetime.signal }).catch(console.warn);
});
onBeforeUnmount(() => lifetime?.abort());
</script>
```

## Angular

Chrome documents experimental Angular integration. Check its current docs
first; otherwise register from a service and abort in `ngOnDestroy` or when the
injector is destroyed.

## Existing forms and existing MCP servers

Do not rewrite forms as JS tools by default; annotate them
([form-annotations.md](form-annotations.md)). Do not hand-wrap an existing
MCP server's tools; bridge it ([mcp-bridge.md](mcp-bridge.md)).
