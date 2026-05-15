# Revere

Claude Code marketplace for [Revere](https://userevere.com) — policy intelligence brought into the assistant surface.

## Install

In Claude Code:

```
/plugin marketplace add useREVERE/revere
/plugin install revere@revere
```

Then in any session:

```
/revere:hello-revere
```

You should see a one-line confirmation that the plugin loaded. That is the entirety of this release — a walking skeleton that proves the marketplace + plugin packaging works end-to-end. Substantive skills land in follow-up versions.

## What's in this release

- Marketplace manifest at `.claude-plugin/marketplace.json`
- One plugin: `revere/`
- One skill: `hello-revere` — prints a confirmation line

## What's not in this release (yet)

- No MCP server connection
- No memory or preference learning
- No `/brief` skill content
- No scheduled or interactive agents
- No two-tier identity (shared profile + per-plugin `CLAUDE.md`)

These are deliberate omissions. The first release validates packaging mechanics in isolation; coupling that validation to substantive skill content would conflate two unknowns. Each item above ships in its own slice.

## Modeled on

[`anthropics/claude-for-legal`](https://github.com/anthropics/claude-for-legal) — same marketplace pattern, scoped down to a single walking-skeleton plugin.
