---
name: mcpfy-webmcp-builder
description: Make a website callable by in-browser AI agents using WebMCP (the browser API where a page registers tools on document.modelContext). Decide with the user which site actions agents should get, then implement them as annotated HTML forms, hand-written registerTool tools, or a bridge that republishes an existing MCP server (such as one built with mcpfy-sdk) inside the page. Trigger on requests about WebMCP, browser-side agent tools, registerTool, toolname/tooldescription form attributes, webmcp-proxy, or letting browser agents operate a site without scraping its DOM.
---

# WebMCP for websites

WebMCP lets a web page hand a list of callable tools to the AI agent built into
(or driving) the user's browser. The tools are plain JavaScript or annotated
HTML that runs **inside the open tab**, under the signed-in user's session.

It is easy to confuse with neighbouring things, so fix the mental model first:

| | Remote MCP server | MCP App widget | WebMCP |
|---|---|---|---|
| Code runs in | your backend | a sandboxed iframe in the chat client | the website's own page |
| Who calls it | any MCP client | the chat client | the browser's agent |
| Auth | its own OAuth / keys | the host's | the user's existing page session |
| Exists when | always | while embedded | while the tab is open |

If the user actually wants a remote server or a chat widget built with
`mcpfy-sdk`, stop and use the `mcpfy-server-builder` skill instead.

API surface (current draft): `document.modelContext` with `registerTool`,
`getTools`, `executeTool`, and a `toolchange` event. The older
`navigator.modelContext` alias is deprecated. Secure context (HTTPS or
localhost) only. Support is still origin-trial level, so read
[references/runtime-setup.md](references/runtime-setup.md) before promising
anything about which browsers work.

## How to run the job

Work through these stages in order. The one hard rule: **nothing gets
implemented until the user has approved a written proposal** (stage 2). Which
actions an agent may perform on someone's site is a product and security
decision, not an implementation detail.

### Stage 1 — Survey the project

Before asking questions, look:

- Where does the browser code start (HTML shell, `main.tsx`, root layout,
  router)? Which framework? If the repo holds several front ends, ask which.
- Which `<form>` elements exist? Which real user journeys (booking, checkout,
  onboarding, search) span several screens?
- Is there already an MCP server for this product? Check for `mcpfy-sdk`, an
  `/mcp` route, or a deployed server URL.
- How does the site authenticate users (cookie session, bearer token in
  memory)?

### Stage 2 — Propose, then wait

Read [references/choosing-an-approach.md](references/choosing-an-approach.md).
Then write the user a proposal in this shape and stop for a reply:

```
Proposed WebMCP surface for <site>

| # | Agent-facing action | Approach | Inputs | What the user sees / what changes |
|---|---------------------|----------|--------|-----------------------------------|
| 1 | ...                 | Form / Custom / Bridge | ... | ... |

Not included this round: ...
Risks I want you to rule on: ... (anything that spends money, deletes, or sends)
```

Keep the first release small (a handful of actions). Suggested defaults when the
user has no preference:

- an MCP server already exists → **Bridge** first, for breadth
- the site is mostly forms → **Form annotations**
- one journey matters far more than the rest → **Custom tool** for that journey

### Stage 3 — Runtime guard

Every approach except pure form annotations needs `document.modelContext`.
Add a feature check that never throws, and a polyfill only if the user wants
non-native browsers covered. Details and snippets:
[references/runtime-setup.md](references/runtime-setup.md).

### Stage 4 — Build the approved surface

| Approach | Reference |
|---|---|
| Form annotations (attributes on real `<form>`s) | [references/form-annotations.md](references/form-annotations.md) |
| Custom tools (`registerTool`) | [references/custom-tools.md](references/custom-tools.md) |
| Framework wiring (vanilla, React, Next, Vue, Angular) | [references/framework-recipes.md](references/framework-recipes.md) |
| Bridge from an existing MCP server | [references/mcp-bridge.md](references/mcp-bridge.md) |

Rules that hold for every approach:

1. One job, one tool. Never publish the same capability through two approaches.
2. Reuse the app's existing logic (API clients, stores, router). Do not script
   fake clicks.
3. Leave the human UI working on browsers that lack WebMCP.
4. Tool metadata (name, description, parameter text) is written for agents and
   must contain no instructions, secrets, or persuasion.

### Stage 5 — Prove it works

Follow [references/testing.md](references/testing.md): list the tools in a real
browser, call each one the way an agent would, and confirm the visible or
stored outcome matches the approved proposal. Fix and repeat. A build that
merely compiles is not finished.

### Stage 6 — Safety pass

Follow [references/safety.md](references/safety.md). At minimum: annotations
are truthful, irreversible actions need confirmation, the server re-checks
authorization, and nothing untrusted flows back to the agent unlabeled.

## Mistakes to avoid

- Coding before the proposal is approved.
- Building a server transport when the user asked for page tools.
- Re-implementing an existing MCP server's tools by hand in page code.
- Publishing near-duplicate tools (`find_x`, `search_x`, `query_x`).
- Relying on `navigator.modelContext` alone, or on removed calls from earlier
  drafts (`provideContext`, `clearContext`, `unregisterTool`). Use the
  `AbortSignal` passed to `registerTool` to remove a tool.
- Returning DOM nodes, functions, or circular objects from `execute`.
- Trusting `inputSchema` as validation. Agents can send anything.
- Declaring the work done without exercising the tools in a browser.
