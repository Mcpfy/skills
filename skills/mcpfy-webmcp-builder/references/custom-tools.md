# Custom tools with registerTool

Use `registerTool` when the agent should get an action shaped like a user goal,
not a copy of today's screens. Agree the design with the user first (see
[choosing-an-approach.md](choosing-an-approach.md)), then build.

Chrome reference: https://developer.chrome.com/docs/ai/webmcp/imperative-api
Spec: https://webmachinelearning.github.io/webmcp/

## Design worksheet

Fill this in per tool before writing code:

| Question | Example (event-ticket site) |
|---|---|
| Goal, one sentence | Hold tickets for a chosen show |
| Human path today | pick show → pick seats → enter details → pay |
| Inputs the agent provides | show name, date, number of tickets, seat preference as text |
| App logic to reuse | `holdSeats()` in the booking service |
| What the user sees afterwards | lands on the payment step with seats reserved |
| State that changes | a 10-minute seat hold; no charge yet |
| Reversible? | yes, holds expire |

Keep the first release to a few tools. Small helpers belong in form annotations
or an existing MCP server.

## Shape of a tool

```js
const mc = document.modelContext;   // fall back to navigator.modelContext only for old runtimes

if (typeof mc?.registerTool === "function") {
  const lifetime = new AbortController();

  await mc.registerTool(
    {
      name: "hold_tickets",
      title: "Hold tickets",
      description:
        "Reserve seats for a show for ten minutes and open the payment step. Nothing is charged until the person confirms payment.",
      inputSchema: {
        type: "object",
        properties: {
          show: { type: "string", description: "Name of the show" },
          date: { type: "string", description: "Performance date, e.g. 2026-11-03" },
          count: { type: "integer", minimum: 1, maximum: 8, description: "Number of tickets" },
          seating: {
            type: "string",
            enum: ["front", "middle", "back", "accessible"],
            description: "Preferred area of the venue",
          },
        },
        required: ["show", "date", "count"],
      },
      annotations: { readOnlyHint: false, consequentialHint: false },
      async execute(input, { signal }) {
        const hold = await bookingService.holdSeats(input, { signal });
        router.navigate(`/checkout/${hold.id}`);
        return `Holding ${hold.count} seats for ${hold.show} on ${hold.date} until ${hold.expiresAt}.`;
      },
    },
    { signal: lifetime.signal },
  );

  // when the tool stops being valid (route change, sign-out):
  // lifetime.abort();
}
```

`registerTool` returns a promise that rejects when the name is already taken,
the name or description is empty, or the schema is invalid. Handle that instead
of letting boot fail.

## Naming

- Verb + object: `hold_tickets`, `search_shows`.
- Make the name say whether it commits or only starts something.
  `create_event` commits; `start_event_creation` opens a form.
- Allowed characters: letters, digits, `_`, `.`, `-`; 1–128 long.
- Never register several names for one job.

## Descriptions

Write for a model deciding between tools. Say what the tool does, when it
applies, and what changes. Describe abilities in positive terms rather than
listing prohibitions; enforce limits through the schema and through errors.

## Input schema

- Top level is `type: "object"`.
- Give every property a description.
- Prefer closed sets (`enum`) when the options are fixed.
- Ask for natural values (`"Express"`, `"next Friday evening"`) rather than
  internal ids, and translate inside `execute`.
- Mark only truly necessary fields as `required`.
- Validate again in code. The schema is guidance, not enforcement.

## Results and errors

- Return plain, serializable data: a string or a simple object. Never DOM
  nodes, functions, cyclic structures, or `undefined`.
- Keep results short and factual. Agents plan their next step from them.
- Update the visible UI to match what happened; agents watch it.
- On failure, throw an `Error` whose message tells the agent how to fix the
  call ("count must be 1–8"). Reasonable retries should be allowed.

## Annotations

| Field | Set it when |
|---|---|
| `readOnlyHint: true` | the tool changes nothing (search, status) |
| `untrustedContentHint: true` | the result contains text written by users or third parties |
| `consequentialHint: true` | the action is high-stakes and should get a confirmation |

Be accurate. A read-only label on a tool that writes teaches agents to call it
casually.

## Lifecycle

- Register a tool only when it is actually usable. If it depends on a route or
  on being signed in, register on entering that state and abort on leaving.
- After an abort, register again with a fresh `AbortController` if the tool
  becomes valid again.
- Forward the `signal` given to `execute` into `fetch` and any cancellable
  work.
- `document.modelContext.addEventListener("toolchange", ...)` fires when the
  set of available tools changes, which is handy in tests.
