# Revere

[Claude Cowork](https://claude.com/product/cowork) and [Claude Code](https://claude.com/product/claude-code) plugin for [Revere](https://userevere.com) — policy intelligence brought into the assistant surface.

> **Primary surface is Cowork.** Most users of this plugin are policy professionals working in Claude Desktop, not developers in a terminal. Claude Code install is supported but secondary.

## What this plugin does

Generates a publication-ready weekly policy briefing as a `.docx` from your Revere bookmarks (supplemented with web search for legislative status, events, and Federal Register deadlines). The output is a fully formatted document with an executive summary table, prioritized recommended actions, thematic sections with source links, and an upcoming events calendar.

Invoke with:

```
/revere:policy-briefing
```

The skill will ask for the organization name and abbreviation, the coverage week, and the priority lanes before pulling source material. If you have a previous briefing, hand it over and the skill will infer most of that and continue the series.

## Install in Claude Cowork

1. [Install Claude Desktop](https://claude.com/download)
2. Get access to Claude Cowork
3. Open Cowork's plugin settings and add `useREVERE/revere` as a plugin marketplace
4. Install the `revere` plugin from that marketplace
5. When the skill first reaches for Revere data, Cowork will prompt you to authorize the Revere connector. Sign in with the account you use at [userevere.com](https://userevere.com).

## Install in Claude Code

```
/plugin marketplace add useREVERE/revere
/plugin install revere@revere
```

Restart Claude Code, then run `/revere:policy-briefing`.

> The policy-briefing skill relies on the Anthropic docx skill (for Word document generation), which Cowork includes by default. Claude Code users may need to install it separately if they don't have it.

## What's in this release

- Marketplace manifest at `.claude-plugin/marketplace.json`
- One plugin: `revere/`
- One skill: `policy-briefing` — generates a weekly policy briefing `.docx`
- Revere MCP connector declared in `revere/.mcp.json` (`https://userevere.com/mcp`)

## What's not in this release (yet)

- No memory or preference learning (the skill re-asks for organization context each run)
- No automated/scheduled briefings — invocation is on-demand
- No cold-start interview to capture a persistent practice profile
- No two-tier identity (shared profile + per-plugin `CLAUDE.md`)

Each item above ships in its own thin slice.

## Modeled on

[`anthropics/claude-for-legal`](https://github.com/anthropics/claude-for-legal) — same marketplace + plugin + MCP pattern.
