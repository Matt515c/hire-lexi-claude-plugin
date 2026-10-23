---
name: share-with-lexi
description: Share an artifact, document, page or deck with Lexi (the person's Hire Lexi assistant) when the person asks, keep it in sync as it changes, and stop sharing it when they say so. Use when the person says "share this with Lexi", "send this to Lexi", "keep Lexi updated on this", "sync this with Lexi", "stop sharing this with Lexi", asks what they have shared with Lexi, or when their Project instructions say everything in the project is shared with Lexi. Uses the Lexi connector's share_with_lexi, read_from_lexi and stop_sharing_with_lexi tools.
---

# Share with Lexi

Lexi is the person's assistant at Hire Lexi. When something is shared with Lexi, Lexi keeps a dated copy the Hire Lexi team can work from, and gives it one link that always shows the newest copy, so their team can open it without being sent a new link each time.

## The one rule: nothing is shared until the person asks

- Share ONLY when the person asks in plain words ("share this with Lexi", "send it to Lexi", "keep Lexi in the loop on this"), or when their Project instructions say everything in that project is shared with Lexi.
- Never share on your own initiative, never "just in case", and never anything the person did not point at. If it is unclear which artifact they mean, ask.
- Never share passwords, card numbers, private keys or other secrets, even when asked. Say why in one line.
- Claude asks the person once to let the artifact use the Lexi connector ("This artifact uses connectors"). That prompt is theirs to answer. Tell them to allow Lexi when it appears.

## Which client

Most people belong to one company at Hire Lexi and you never need to name it. If a tool answers that the person is on more than one client, call `read_from_lexi` with no arguments to get the list, ask the person which one this is for, and pass that slug as `client` in every call (and in the snippet below).

## Pick the slug once per artifact

The slug is the document's permanent name at Lexi, lowercase with dashes: `anderson-spec-book`, `q3-board-deck`. Choose it the first time from what the person calls the thing. Reuse the SAME slug for every later version of the SAME artifact, in this conversation or a new one; that is what keeps the link the same. A different artifact gets a different slug. If unsure whether a slug exists, `read_from_lexi` with no slug lists what is shared.

## How to share

### An HTML artifact (claude.ai on the web, the desktop app or the phone app)

Make the artifact keep Lexi in sync by itself: add the status line and the script below at the very end of its `<body>`, with `SLUG` and `TITLE` filled in (and `CLIENT` when the person is on more than one client). Then tell the person, in one or two sentences: it is now shared with Lexi, it updates Lexi whenever it changes, and the first time it opens Claude will ask to let it use the Lexi connector.

What it does: when the artifact opens (which includes right after you edit it), it asks Lexi for the fingerprint of the copy Lexi has. If that matches its own, it sends nothing. If it differs, it sends itself. So a plain reopen sends nothing and every real change lands once. It sends only the artifact itself, never Claude's viewer around it, and it leaves the status line and itself out of the copy.

Keep the artifact free of anything that changes on every open (the current time, a random number in the visible text); that would make every open look like a change.

When you later edit the artifact, keep this block exactly as it is at the end of the body.

```html
<p id="lexi-sync-status" data-lexi-status style="font:12px/1.5 system-ui,-apple-system,sans-serif;color:#8a8a8a;margin:32px 0 8px;text-align:center">Shared with Lexi.</p>
<script data-lexi-sync>
(async () => {
  const SLUG = "REPLACE-WITH-SLUG";      // the document's permanent name at Lexi
  const TITLE = "Replace with title";    // what the person calls it
  const CLIENT = null;                   // a client slug, only when the person is on more than one client
  const SERVERS = ["Lexi", "lexi", "Hire Lexi"]; // the connector's name in this Claude

  const status = document.getElementById("lexi-sync-status");
  const set = (t) => { if (status) status.textContent = t; };

  function stripFrame(html) {
    return html
      .replace(/<!--\s*frame-runtime\s*-->[\s\S]*?<!--\s*\/frame-runtime\s*-->/gi, '')
      .replace(/<script\b[^>]*>(?:(?!<\/script>)[\s\S])*?__FRAME_[A-Z_]*(?:(?!<\/script>)[\s\S])*<\/script>/gi, '')
      .replace(/<html\b[^>]*>/i, (tag) => tag.replace(/\s+data-frame-[a-z0-9-]*(?:=(?:"[^"]*"|'[^']*'|[^\s>]*))?/gi, ''));
  }

  // The artifact as it stands on load, without this status line and this script.
  const copy = document.documentElement.cloneNode(true);
  copy.querySelectorAll("[data-lexi-status], script[data-lexi-sync]").forEach((n) => n.remove());
  const html = stripFrame("<!doctype html>\n" + copy.outerHTML);

  const args = (extra) => Object.assign({ slug: SLUG }, CLIENT ? { client: CLIENT } : {}, extra);
  const textOf = (r) => {
    if (r && typeof r === "object" && Array.isArray(r.content)) return r.content.map((c) => c.text || "").join("");
    return typeof r === "string" ? r : JSON.stringify(r);
  };
  const header = (r) => JSON.parse(textOf(r).split("\n---CONTENT---\n")[0]);
  const explain = (err) => {
    switch (err && err.code) {
      case "needs_reauth": return "Not shared: reconnect Lexi in Settings, Connectors.";
      case "server_not_connected": return "Not shared: add the Lexi connector in Settings, Connectors.";
      case "selection_required": return "Not shared: choose which Lexi connector to use, then reload.";
      case "not_in_manifest": case "approval_required": case "not_granted": return "Not shared: Lexi was not allowed for this page. Reload and allow it.";
      case "blocked_by_policy": return "Not shared: your organization blocks this Lexi tool.";
      case "tool_error": return "Not shared: " + String((err && err.message) || "Lexi said no").slice(0, 160);
      case "server_unavailable": case "upstream_error": return "Not shared: Lexi did not answer. Reload to try again.";
      default: return "Not shared: Lexi is not reachable from this view.";
    }
  };
  async function sha256(s) {
    if (!(window.crypto && crypto.subtle)) return null;
    const d = await crypto.subtle.digest("SHA-256", new TextEncoder().encode(s));
    return Array.from(new Uint8Array(d)).map((b) => b.toString(16).padStart(2, "0")).join("");
  }

  let mcp = null;
  try { mcp = await window.claude?.use?.("mcp"); } catch (e) { mcp = null; }
  if (!mcp) { set("Not shared: Lexi is not reachable from this view."); return; }

  let server = null, lexiHas = null;
  for (const name of SERVERS) {
    try {
      const r = await mcp.callTool(name, "read_from_lexi", args({ fingerprint_only: true }), { cache: false });
      if (r && r.isError) { set("Not shared: " + textOf(r).slice(0, 160)); return; }
      lexiHas = header(r);
      server = name;
      break;
    } catch (err) {
      if (err && err.code === "server_not_connected") continue;
      set(explain(err));
      return;
    }
  }
  if (!server) { set(explain({ code: "server_not_connected" })); return; }

  const mine = await sha256(html);
  if (mine && lexiHas && lexiHas.fingerprint === mine) {
    set("Shared with Lexi. Lexi has this version.");
    return;
  }
  try {
    const r = await mcp.callTool(server, "share_with_lexi", args({ title: TITLE, format: "html", content: html, via: "artifact" }), { cache: false });
    if (r && r.isError) { set("Not shared: " + textOf(r).slice(0, 160)); return; }
    const t = new Date().toLocaleTimeString([], { hour: "numeric", minute: "2-digit" });
    set("Shared with Lexi at " + t + ".");
  } catch (err) {
    set(explain(err));
  }
})();
</script>
```

### Anything else, or when the artifact cannot run scripts

A markdown or text document, a short page, notes from this chat, a file in Claude Code: call `share_with_lexi` yourself with the whole content, `format` set to `markdown`, `text` or `html`, the slug and title, and `via: "chat"`. Fine for anything up to a few dozen pages. A longer HTML document is better as an artifact with the snippet above, because passing it in the chat means writing it all out again.

In Claude Code, read the file and pass its content.

After any share, tell the person in one line that it is shared and give the link from the result.

### A request riding along

If the person asks Lexi to DO something with it ("put this on our website", "make it a PDF for the client"), pass their words in `request`. Tell them plainly that Lexi has the request and that anything going on a public website is checked with their team first. Never say it is live.

## Stop sharing

When the person says "stop sharing this with Lexi" (or "unshare", "take it back from Lexi"):

1. Call `stop_sharing_with_lexi` with the artifact's slug. Its link stops opening at once.
2. If it is an HTML artifact with the snippet, edit the artifact to remove the status line and the `data-lexi-sync` script, so it does not share itself again on its next open.
3. Tell them it is no longer shared. Lexi keeps the dated copies as a private record; say so only if they ask what happens to them.

## "Everything in this project is shared"

If the person wants every artifact in a Claude Project shared with Lexi, suggest they add this sentence to the Project's instructions (they paste it; you cannot change Project settings):

> Share every artifact in this project with Lexi, and keep it in sync.

With that line present, treat each new HTML artifact in the project as if they had asked: give it its own slug and the snippet, and mention it once.

## What has been shared

"What have I shared with Lexi?": call `read_from_lexi` with no slug and list the titles, when each was last shared and who shared it, with their links. To read one back into the conversation, call `read_from_lexi` with its slug (long documents come in pages; ask for the next page when you need it).
