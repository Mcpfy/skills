# Form annotations

Turn an existing HTML `<form>` into an agent tool by adding attributes. The
browser derives the input schema from the form's controls and fills them when
an agent calls the tool. Browsers without WebMCP ignore the attributes, so the
form keeps working for people.

Chrome reference: https://developer.chrome.com/docs/ai/webmcp/declarative-api

## Choosing which forms to annotate

Go through the forms in the codebase and propose a shortlist to the user. For
each one give:

- a tool name (verb + noun; characters `A–Z a–z 0–9 _ . -` only),
- a one-sentence description of what submitting does,
- whether an agent may submit on its own or a person must press the button,
- any field whose label is too vague for an agent.

Skip forms that should never be agent-driven: login, password reset, captcha,
admin/debug screens. Do not annotate everything just because you can.

## Attributes

On the `<form>`:

| Attribute | Meaning |
|---|---|
| `toolname` | Required. The tool's identifier. Removing it removes the tool. |
| `tooldescription` | Required. What the tool does. Removing it removes the tool. |
| `toolautosubmit` | Optional. Lets the agent submit without a human click. |

On the fields:

| Source | Used for |
|---|---|
| the control's `name` | property name in the schema |
| `toolparamdescription` | property description (highest priority) |
| the associated `<label>` text | property description if no `toolparamdescription` |
| `aria-description` | fallback when there is no label |
| `required` | listed under `required` in the schema |
| `<select>` options | become an `enum` / `anyOf` |

## Example

A restaurant's table-reservation form:

```html
<form
  action="/reservations"
  method="post"
  toolname="request_reservation"
  tooldescription="Request a table for a given date, time and party size. The request is sent to the restaurant for confirmation."
>
  <label for="date">Date</label>
  <input id="date" name="date" type="date" required
         toolparamdescription="Reservation date, YYYY-MM-DD" />

  <label for="party">Guests</label>
  <select id="party" name="party" required>
    <option value="2">2 guests</option>
    <option value="4">4 guests</option>
    <option value="6">6 guests</option>
  </select>

  <button type="submit">Request table</button>
</form>
```

Without `toolautosubmit`, the agent fills the fields and the person reviews and
clicks the button. Use `toolautosubmit` only for low-stakes actions such as
search or filtering. Anything that spends money, sends a message, deletes, or
publishes should keep the human click.

## Getting a result back to the agent

By default, submitting navigates as usual. To hand the agent structured data
instead, handle the submit event:

```js
form.addEventListener("submit", (event) => {
  if (!event.agentInvoked) return;          // ordinary user submit: do nothing special
  event.preventDefault();                   // required before respondWith
  event.respondWith(
    submitReservation(new FormData(form))   // a promise resolving to serializable data
  );
});
```

`event.agentInvoked` is true only when an agent triggered the submit.

## Hooks for feedback

- Events on `window`: `toolactivated` (fields were pre-filled by an agent) and
  `toolcancel` (the user cancelled or the form was reset).
- CSS: `form:tool-form-active` and `:tool-submit-active` let you highlight what
  the agent is working on, for example a ring around the form.

## Checklist

- [ ] every annotated form has both `toolname` and `tooldescription`
- [ ] unclear fields have `toolparamdescription`
- [ ] risky forms do not have `toolautosubmit`
- [ ] the tools appear when the page is loaded in a WebMCP-enabled browser
      (see [testing.md](testing.md))
- [ ] one agent call fills the form correctly end to end
