---
name: talk-to-lexi
description: Handle a message addressed to Lexi, the person's Hire Lexi assistant, from inside Claude. Use when the person writes "Hey Lexi ...", "Lexi, can you ...", "ask Lexi ...", "tell Lexi ...", or otherwise speaks to Lexi by name in a chat where the Lexi connector is connected. Routes the ask to the right Lexi tool (share, read, stop sharing, or a request for the Hire Lexi team) and reports back exactly what Lexi did, never more.
---

# Talk to Lexi

Lexi is the person's assistant at Hire Lexi, reached in Claude through the Lexi connector. When the person talks to Lexi by name, they want something done on Lexi's side. You are still Claude: do the ask through Lexi's tools and report what Lexi answered. Do not speak as Lexi or invent what she would say.

## Route the ask

| The person says | Do this |
|---|---|
| "Hey Lexi, here's the new version" / "Lexi, take a look at this" / "share this with Lexi" | Share it, following the share-with-lexi skill (opt-in is satisfied: they asked). |
| "Lexi, put this on our website" / "can you upload this for us?" / "make this a PDF for the client" | Share it with the person's words in `request` (reuse the document's slug, share-with-lexi skill). The request is emailed to the Hire Lexi team; nothing is published from Claude. If `request_delivered` is true, say the team has it and will check with them before anything goes live; if false, say it did not reach the team and to email support@hirelexi.com. |
| "Lexi, what do you have from us?" / "what have I shared?" | `read_from_lexi` with no slug; list titles, dates, who shared, links. |
| "Lexi, pull up the spec book" / "what does the latest version say about X?" | `read_from_lexi` with the slug (list first if you do not know it), then answer from what came back, and say which version and date you read. |
| "Lexi, stop sharing this" / "take it down" | `stop_sharing_with_lexi`, and remove the sync from the artifact (share-with-lexi skill, "Stop sharing"). |
| Anything else for Lexi herself (a question about their leads, their calendar, their business) | This connector does not carry those yet. Say so in one line, and that they can ask Lexi the usual way (her text or WhatsApp line). If it is something to pass on, offer to send it as a `request` with a share. |

## Rules

- Public website publishing is an outward act. Lexi never puts anything on a public website straight from Claude: the request goes to the team, and someone checks with the person first. Never tell the person a page is live, published or sent unless a tool result says exactly that.
- Report what the tool returned: the link, the version, "unchanged" when Lexi already had that version. If a tool refused (wrong client, not signed in), say what it said and what to do.
- If the person is on more than one client, ask which one before acting (share-with-lexi skill, "Which client").
- If the Lexi connector is not connected in this chat, say so and point them to Settings, Connectors, where Lexi is connected with their work email.
