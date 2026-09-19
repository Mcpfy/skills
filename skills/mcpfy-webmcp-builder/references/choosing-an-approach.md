# Choosing an approach

Three ways to put actions in front of a browser agent. They can be combined,
but each user-facing capability should be delivered by exactly one of them.

## At a glance

| | Form annotations | Custom tools | Bridge |
|---|---|---|---|
| You write | HTML attributes | JavaScript per tool | one small setup call |
| Best when | real `<form>`s already exist | one journey deserves a purpose-built call | a remote MCP server already has the right tools |
| Agent sees | fields fill in on screen | whatever your code does to the UI | nothing on screen; just a result |
| Typical effort | minutes per form | hours per tool | minutes, plus CORS/auth checks |
| Main risk | over-exposing forms | tool drifts from real UI logic | auth and CORS mismatch |

## Form annotations

The browser reads a normal `<form>`, builds an input schema from its controls,
and lets the agent fill it. Humans keep the same form, and browsers without
WebMCP ignore the extra attributes. Choose this when the work is "let an agent
use the form that's already there". It does not help when the interaction is built from
clickable components that never sit inside a `<form>`, or when several screens
should collapse into a single call.

Detail: [form-annotations.md](form-annotations.md).

## Custom tools

`registerTool` lets you define an action that matches a user *goal* instead of
a screen. Example for a restaurant site: rather than exposing the date picker,
party-size stepper and contact form separately, offer one `reserve_table` tool
that takes date, time and party size, calls the booking API, and lands the user
on the confirmation view.

To design one, settle three things with the user:

1. the goal in one sentence,
2. the inputs the agent must supply (in natural terms, not internal ids),
3. the observable result: which screen, which stored state, what the user is
   told.

Detail: [custom-tools.md](custom-tools.md).

## Bridge

If the product already ships a remote MCP server (for instance one built with
`mcpfy-sdk`), the design work is done. A bridge connects to it from the page,
lists its tools, and registers each one through WebMCP.

Say plainly to the user:

- It is the quickest route and adds no per-tool code.
- Nothing changes on screen when the agent calls a tool, because the work
  happens on the server.
- It can reuse the user's login only if the site and the MCP server share one
  OAuth client. Otherwise agents need a separate auth decision.
- The MCP endpoint must allow the site's origin via CORS.

Detail: [mcp-bridge.md](mcp-bridge.md).

## Combining

| Combination | Why you would |
|---|---|
| Bridge + Custom | server tools for breadth, one hand-built journey with visible UI |
| Forms + Custom | annotate simple forms, replace one multi-step flow with a single tool |
| Bridge + Forms | server handles product actions, forms handle contact/marketing |

Before shipping any combination, list every tool name in one place and delete
duplicates. A capability exposed twice (say `create_order` from the bridge and
again from a hand-written tool) confuses agents and doubles your test surface.

## Deciding on write-capable actions

For each action the agent could take, note whether it reads, writes, or is
irreversible (pay, delete, send, publish). Put the answer in the proposal.
Irreversible ones need a confirmation step designed in from the start; see
[safety.md](safety.md).
