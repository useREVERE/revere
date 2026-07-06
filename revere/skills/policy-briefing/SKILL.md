---
name: policy-briefing
description: "Generate a professional weekly policy briefing document (.docx) from policy intelligence sources such as Revere bookmarks, RSS feeds, or curated policy items. Trigger this skill whenever the user asks for a policy brief, regulatory briefing, weekly policy roundup, government affairs summary, or legislative tracking document — especially when the output should be a formatted Word document with prioritized developments, action items, and an events calendar. Also trigger when the user has a previous policy briefing and wants to produce the next edition in the same format. This skill covers the full workflow: confirming the client context and date range, pulling source material from policy intelligence tools (both bookmarks and scored items), supplementing with web search for context and events, clustering related items, assessing relevance and priority against the client profile, and generating a publication-ready .docx with tables, color-coded priority tags, and structured sections."
---

# Policy Briefing Generator

Revere serves this workflow's authoritative body through the Revere MCP. Before beginning the briefing workflow, call `get_skill("policy-briefing")` and follow the returned instructions.

If the Revere MCP tools are unavailable or unauthenticated, tell the user to connect the Revere MCP first, then retry the request. Do not recreate the workflow from memory; the served body is the source of truth.
