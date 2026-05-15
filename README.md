# Revere

[Claude Cowork](https://claude.com/product/cowork) and [Claude Code](https://claude.com/product/claude-code) plugin for [Revere](https://userevere.com) — policy intelligence brought into the assistant surface.

> **Primary surface is Cowork.** Most users of this plugin are policy professionals working in Claude Desktop, not developers in a terminal. Claude Code install is supported but secondary.

## Install in Claude Cowork

1. [Install Claude Desktop](https://claude.com/download)
2. Get access to Claude Cowork
3. Open Cowork's plugin settings and add `useREVERE/revere` as a plugin marketplace
4. Install the `revere` plugin from that marketplace
5. When a skill first needs Revere data, Cowork will prompt you to authorize the Revere connector. Sign in with the account you use at [userevere.com](https://userevere.com).

Run the load check:

```
/revere:hello-revere
```

You should see a confirmation line.

## Install in Claude Code

```
/plugin marketplace add useREVERE/revere
/plugin install revere@revere
```

Restart Claude Code, then:

```
/revere:hello-revere
```

## What's in this release

- Marketplace manifest at `.claude-plugin/marketplace.json`
- One plugin: `revere/`
- One skill: `hello-revere` — prints a confirmation line, no tool calls
- Revere MCP connector declared in `revere/.mcp.json` (`https://userevere.com/mcp`)

The connector is declared but not exercised by `hello-revere`. That's intentional — this release validates packaging and connector declaration; the first MCP-backed skill lands in the next slice.

## What's not in this release (yet)

- No skill that calls the Revere MCP (deferred — next slice)
- No memory or preference learning
- No `/brief` skill content
- No scheduled or interactive agents
- No two-tier identity (shared profile + per-plugin `CLAUDE.md`)

Each item above ships in its own thin slice.

## Modeled on

[`anthropics/claude-for-legal`](https://github.com/anthropics/claude-for-legal) — same marketplace + plugin + MCP pattern, scoped down to a single walking-skeleton plugin.
