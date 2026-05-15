---
name: hello-revere
description: Confirm the Revere plugin loaded. Prints a one-line greeting and exits. Walking-skeleton skill that validates marketplace + plugin packaging end-to-end. The Revere MCP connector is declared in the plugin but this skill makes no tool calls.
---

Output exactly the following text and then stop. Do not add commentary, do not ask follow-up questions, do not run any tools.

```
Revere plugin loaded — marketplace install works end-to-end.

The Revere connector (https://userevere.com/mcp) is declared in this plugin. You will be prompted to authorize it the first time a skill needs it. This walking-skeleton skill itself makes no MCP calls.

Substantive skills (briefs, regulatory monitoring, tool-calling against your Revere data) land in follow-up versions.
```
