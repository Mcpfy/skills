# Safety review

Agents that use WebMCP act **as the signed-in user**. Treat every registered
tool as an authenticated capability of that person.

## Checks

1. **Least privilege.** Register only what the current user may do. Remove
   tools on sign-out.
2. **Server-side authorization.** Tools call your existing endpoints; those
   must enforce permissions and CSRF protection themselves. WebMCP does not
   replace either.
3. **Truthful annotations.** `readOnlyHint`, `untrustedContentHint` and
   `consequentialHint` must match reality.
4. **Clean metadata.** Names, titles, descriptions and parameter text are read
   by a model. Keep them factual: no instructions, no hidden directives, no
   secrets.
5. **Careful output.** Bound the size of results. If they include text written
   by users or third parties, set `untrustedContentHint: true` and avoid echoing
   raw HTML.
6. **Own validation.** Check inputs in code and reject bad ones with a clear
   message.
7. **Irreversible actions need a person.** Payments, deletions, outgoing
   messages and publishing should require an explicit parameter, an on-page
   confirmation, or (for forms) no auto-submit.
8. **Data minimisation.** Do not return secrets or bulk personal data.

## Threats to keep in mind

| Threat | Mitigation |
|---|---|
| Prompt injection through tool output | bound and label untrusted content, do not return markup |
| Tool poisoning through descriptions | you author them; keep them plain |
| Confused deputy via the user's session | authorize on the server for every mutating call |
| Cross-origin exposure | keep tools same-origin unless deliberately shared with `exposedTo` and `allow="tools"` |

## Bridge specifics

The bridge exposes whatever the MCP server lists. Review that list as if it
were a public API, and confirm the server's CORS and auth settings before
release.
