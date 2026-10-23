# Lexi for Claude (Hire Lexi plugin)

One install that lets anyone's Claude share artifacts and documents with Lexi, their Hire Lexi assistant, in plain words: "share this with Lexi", "stop sharing this with Lexi", "Hey Lexi, put this on our website". Generic: nothing here knows any client. Linear: DUKE-268. Plan: `core/memory/2026-10-22-hire-lexi-claude-connector-BUILD-PLAN.md` in the life-OS repo.

Layout copies Canva's plugin (canva-sdks/canva-skills):

- `.mcp.json`: the Lexi connector, `https://brain.hirelexi.com/mcp` (Streamable HTTP; each person signs in with an emailed code; the server is lexi-brain, branch `share-to-lexi`).
- `.claude-plugin/plugin.json`: the manifest, listing the two skills.
- `.claude-plugin/marketplace.json`: makes this repo a one-plugin marketplace, so Claude Code can install it straight from GitHub.
- `skills/share-with-lexi/SKILL.md`: opt-in sharing, the self-syncing snippet for HTML artifacts (it checks Lexi's fingerprint first and sends only on a real change, and strips Claude's viewer frame exactly as the server does; lexi-brain's `npm run test:connector` checks that), stop sharing, the per-Project "everything here is shared" rule.
- `skills/talk-to-lexi/SKILL.md`: "Hey Lexi ..." routed to the right tool; public website publishing always goes through the team first.

## Install

Claude Code (private repo, needs GitHub access):

```
/plugin marketplace add Matt515c/hire-lexi-claude-plugin
/plugin install hire-lexi@hire-lexi
```

claude.ai, Desktop and the phone: until the directory listing, add the connector itself (Settings, Connectors, Add custom connector, URL `https://brain.hirelexi.com/mcp`), or open the prefilled link:
`https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=Lexi&connectorUrl=https%3A%2F%2Fbrain.hirelexi.com%2Fmcp`

Help page and privacy policy: hirelexi.com/claude and hirelexi.com/claude/privacy (hirelexi-site, branch `share-to-lexi-pages`).
