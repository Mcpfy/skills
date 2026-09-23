# mcpfy-webmcp-builder

Turns a website into something an in-browser AI agent can operate directly, using [WebMCP](https://webmachinelearning.github.io/webmcp/) — the browser API where a page publishes tools on `document.modelContext` for the agent driving that tab to call.

This is a Claude Code / Claude.ai skill: see [SKILL.md](SKILL.md) for the instructions Claude follows, and the root [README.md](../../README.md) for how to install this skills collection.

## What it does

Given a real front-end codebase, the skill guides Claude through:

1. **Survey** — read the app, find its forms and multi-step user journeys, and check whether an MCP server (e.g. one built with `mcpfy-sdk`) already exists for it.
2. **Propose** — write up a table of candidate agent-facing actions and how each would ship, then stop and wait for approval. Nothing is implemented before this.
3. **Build** — implement each approved action via one of:
   - **Form annotations** — `toolname`/`tooldescription` attributes on existing `<form>`s
   - **Custom tools** — hand-written `registerTool` calls for journeys that need one
   - **Bridge** — republish an existing MCP server's tools inside the page
4. **Test** — list and call the tools in a real browser (via Chrome DevTools MCP) and confirm the outcomes match what was approved.
5. **Review** — a safety pass: truthful tool descriptions, confirmation before irreversible actions, server-side authorization, and no unlabeled untrusted output flowing back to the agent.

## When to use it

Trigger this skill for requests about WebMCP, `registerTool`, `toolname`/`tooldescription` form attributes, `webmcp-proxy`, or letting a browser agent operate a site without scraping its DOM.

**Not for** building remote MCP servers or chat-client widgets — use [`mcpfy-server-builder`](../mcpfy-server-builder) for those instead.

## Contents

| File | Purpose |
|---|---|
| [SKILL.md](SKILL.md) | The skill definition and stage-by-stage instructions Claude follows |
| [references/choosing-an-approach.md](references/choosing-an-approach.md) | How to pick between form annotations, custom tools, and a bridge |
| [references/form-annotations.md](references/form-annotations.md) | Annotating existing `<form>`s as agent tools |
| [references/custom-tools.md](references/custom-tools.md) | Writing hand-rolled tools with `registerTool` |
| [references/framework-recipes.md](references/framework-recipes.md) | Wiring for vanilla JS, React, Next.js, Vue, and Angular |
| [references/mcp-bridge.md](references/mcp-bridge.md) | Republishing an existing MCP server's tools inside the page |
| [references/runtime-setup.md](references/runtime-setup.md) | Feature-detecting `document.modelContext` and polyfill guidance |
| [references/testing.md](references/testing.md) | Verifying published tools in a real browser |
| [references/safety.md](references/safety.md) | The safety pass: honest annotations, confirmation, authorization |

## Good to know

WebMCP support is still experimental (origin trials / flags in Chromium-based browsers). The skill checks current support before making promises and every approach keeps the normal UI working in browsers that lack it.

## License

See [LICENSE.txt](LICENSE.txt).
