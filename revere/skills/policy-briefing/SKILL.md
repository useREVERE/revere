---
name: policy-briefing
description: "Generate a professional weekly policy briefing document (.docx) from policy intelligence sources such as Revere bookmarks, RSS feeds, or curated policy items. Trigger this skill whenever the user asks for a policy brief, regulatory briefing, weekly policy roundup, government affairs summary, or legislative tracking document — especially when the output should be a formatted Word document with prioritized developments, action items, and an events calendar. Also trigger when the user has a previous policy briefing and wants to produce the next edition in the same format. This skill covers the full workflow: confirming the client context and date range, pulling source material from policy intelligence tools (both bookmarks and scored items), supplementing with web search for context and events, clustering related items, assessing relevance and priority against the client profile, and generating a publication-ready .docx with tables, color-coded priority tags, and structured sections."
---

# Policy Briefing Generator

Produce a structured, publication-ready weekly policy briefing as a `.docx` file. The briefing synthesizes policy intelligence items (regulatory actions, legislation, testimony, enforcement, rulemakings) into a prioritized document with executive summary tables, analysis sections, recommended actions, and an events calendar.

## When to Use

- User asks for a "policy brief," "regulatory briefing," "weekly roundup," or similar
- User has policy intelligence items (from Revere, RSS, manual input) and wants a formatted deliverable
- User provides a previous briefing and says "write the next one" or "update this for this week"
- User wants to turn a set of bookmarks or policy items into an organized, prioritized document

## Workflow Overview

The briefing is built in four phases. Do not skip phases or combine them — each produces inputs the next phase needs.

### Phase 1: Gather Source Material

This phase determines what enters the briefing. It runs entirely against Revere MCP tools. Do not skip the confirmation steps (1 and 2) — pulling against the wrong client context or date range wastes the rest of the workflow. If Revere MCP tools are not available, fall back to whatever policy source the user provides (uploaded files, pasted text, URLs) and skip to Phase 2.

#### Step 1 — Resolve and confirm the client context

Call `list_client_contexts`.

- **If the user has multiple contexts:** ask which one to use before pulling anything. Do not assume a default.
- **If the user has exactly one context:** proceed automatically and mention it in passing as you go (e.g., "Pulling items for AIR…"). No need to stop and ask.

Then call `get_client_profile` for the resolved context. The returned profile markdown is the analytical lens for the whole briefing — it defines the organization's priorities and lanes, which is what Phase 3 uses to assess priority. Hold onto it.

#### Step 2 — Confirm the date range

- **If the user specified a range** in their request, use it.
- **Otherwise**, propose the current calendar week defined as **last Saturday through this Friday**, and ask the user to confirm or override before pulling. (The Saturday start catches developments that landed over the weekend, before the workweek begins — it mirrors how the brief is actually consumed.)

The confirmed range maps to the `since` (Saturday) and `until` (Friday) parameters, in `YYYY-MM-DD` format.

#### Step 3 — Pull source material

Within the confirmed context and date range, make two calls:

1. **Bookmarks** — `get_bookmarks(since, until, client_context_id, include_content: true)`. Do not set a `min_relevance` floor; a bookmark is a deliberate human signal of interest and overrides whatever score the item received. Returns titles, links, the generic summary, source content, and **tailored summaries**.
2. **Scored items** — `get_items(since, until, client_context_id, min_relevance: LOW_RELEVANCE, include_content: true)`. This catches everything the user did not manually triage. `LOW_RELEVANCE` as the floor pulls everything except `NOT_RELEVANT`. Returns titles, links, the generic summary, and source content — but **not** tailored summaries.

The absence of bookmarks is normal and must not cause a hiccup. A user may bookmark on some days and not others, or not at all. Treat bookmarks as a strong inclusion signal when present; never treat their absence as a problem.

#### Step 4 — Item-level deduplication

The same Revere item can appear in both result sets (e.g., a Critical item the user also bookmarked). Merge the two sets by item ID so each unique item is processed once. Tag each item with whether it was bookmarked — this signal carries forward into clustering (Step 6) and prioritization (Phase 3).

#### Step 5 — Ensure a tailored summary for every surviving item

Tailored summaries are the client-specific analytical lens; the generic "what's in this" summary is not a substitute.

- **Bookmarked items** already carry a tailored summary from the `get_bookmarks` call.
- **Non-bookmarked items** do not. Use the generic summary to triage whether the item plausibly belongs in the brief, then call `generate_tailored_summary(item_id, client_context_id)` for those that do. This call returns a cached summary when one exists, so it is cheap and idempotent — only pass `force_regenerate: true` if you have reason to believe the cached version is stale.

#### Step 6 — Selective full hydration

`include_content: true` already returned source content in the list calls, so a separate `get_item` call is usually unnecessary. Reach for `get_item` only when the tailored and generic summaries are not enough to write the entry or to resolve whether two items belong in the same cluster — prioritizing bookmarked, `CRITICAL`, and `HIGHLY_RELEVANT` items, which are likeliest to anchor a brief entry and benefit from full text.

#### Step 7 — Cluster by policy event

Group items that cover the same underlying development into a single briefing entry, viewed through the eyes of a government relations analyst. A single policy event — a bill advancing, a rule dropping, an enforcement action — often generates multiple items: agency press releases, newsletter coverage, individual member statements. These belong together as one entry with multiple source links, not as separate subheadings.

This is looser than strict event-level deduplication and tighter than thematic-section grouping (which happens later, in Phase 4). When an item was bookmarked, that is a strong signal it should anchor its cluster.

#### Step 8 — Identify gaps

Compare the clusters against the organization's priority threads — drawn from the client profile (Step 1) and the previous briefing if the user provided one. A thread the org tracks that saw no development in the window should be flagged for the Immediate Actions section as "continue monitoring," **not** written up as fabricated coverage. The previous brief is a reference for what the org cares about, not a template to fill regardless of whether news occurred.

### Phase 2: Supplement with Web Search

Primary sources rarely cover everything needed for a complete briefing. Search for:

- **Legislative status updates** — bill timelines, markup schedules, vote counts, negotiation status. Policy intelligence tools capture official releases but often miss the political dynamics reported by trade press.
- **Upcoming events** — committee hearing schedules, agency roundtables, comment deadlines. Check the relevant committee websites directly (e.g., `financialservices.house.gov` for HFSC schedules).
- **Proposed rule comment periods** — Federal Register publication dates and deadlines. **Always search for the actual calendar deadline** rather than writing "60 days after Federal Register publication." The team needs a concrete date (e.g., "June 9, 2026"), not a relative reference. Check the Federal Register entry or agency press release for the exact date. Include the concrete deadline in both the body text of the relevant section and the events table.
- **Context from the previous briefing** — if the user provides one, check whether action items from last week have progressed. Drop carried-forward items from the events table once they are no longer actionable (e.g., a comment period that has closed, a hearing that has passed).

**Critical:** Track which content came from primary sources vs. web search. If the user asks about sourcing, you must be able to distinguish the two cleanly. When web-sourced material is substantial (e.g., a full section on legislation status), note the sources so the user can link to them.

### Phase 3: Analyze and Prioritize

For each item, assess:

- **Priority** (HIGH / MODERATE / LOW) — based on alignment with the organization's stated priorities, deadline proximity, and whether a response window is open
- **Recommended action** — specific next steps (e.g., "submit comment letter," "staff outreach to X office," "monitor")
- **WHY THIS MATTERS** — a paragraph connecting the development to the organization's work, written analytically rather than as a summary. This section should explain what's new, what it means for the organization specifically, and what gap or opportunity it creates.

Priority should be assessed from the organization's perspective, not general importance. A major regulatory action that doesn't touch the organization's lanes is LOW; a niche staff letter that opens a direct engagement opportunity is HIGH.

Anchor "the organization's perspective" in the client profile pulled in Phase 1, Step 1 — its stated priorities and lanes are the reference for what counts as on-lane versus peripheral. Do not rely on the Revere relevance score alone; that score reflects analytical interest, while priority here reflects whether the org should act now (see the Priority Assessment note under Pitfalls). The bookmark flag from Phase 1 is a useful secondary signal — an item the user bookmarked is one they already flagged as worth their attention — but it is not a substitute for assessing the engagement window.

### Phase 4: Generate the Document

Read the docx skill (`/mnt/skills/public/docx/SKILL.md`) before writing any code. Use `docx-js` (Node.js `docx` package) to generate the `.docx`. See `references/template.md` in this skill for the full code template.

After generation, always validate:
```bash
python /mnt/skills/public/docx/scripts/office/validate.py output.docx
```

## Document Structure

The briefing follows a fixed structure. Every section is required unless explicitly empty.

### 1. Title Block
- Organization name
- "Federal Policy Briefing" (or equivalent title)
- "Week of [date]" — use the **Monday** of the coverage week, not the first date a bookmarked item was published. The header and title block should match.

### 2. AT A GLANCE Table
Three-column table: DEVELOPMENT | PRIORITY | AIR ACTION (or equivalent org action column). Each row is 1–2 sentences summarizing the development, a color-coded priority tag, and the recommended action. This is the executive summary — a reader who only sees this table should understand the week's key items and what to do about them.

**Row formatting:** Each development cell has a bold-ish title followed by a dash and a brief description. Use a consistent separator pattern: "Title - description sentence." (not a newline, not a colon). Keep descriptions to 1–2 sentences max. Avoid day-of-week references (e.g., "Thursday") in descriptions — they age poorly; use "this week" or omit timing.

Only include HIGH and MODERATE items in this table. LOW-priority items get coverage in the thematic sections but do not belong in the executive summary — the At-a-Glance table should be tight and actionable, not comprehensive.

Items that are primarily "monitor" actions (e.g., forthcoming rulemakings with no current comment period, legislation the org is tracking but not actively engaging on) may be better served as Immediate Actions line items rather than full At-a-Glance rows, even if they are HIGH priority in the thematic sections.

### 3. IMMEDIATE ACTIONS
Arrow-prefixed (→) list of deadline-driven items. Intro line should read: "Deadline-driven items - recommended action in the next two weeks." The word "recommended" is intentional — these are suggestions to the team, not directives. Each item names the specific action, the relevant entity, and the deadline or window. These are drawn from HIGH and MODERATE items but reformulated as concrete to-dos.

**Punctuation:** Use plain dashes ( - ) rather than em dashes (—) for mid-sentence breaks within action items. Keep the tone crisp and scannable — these are to-do items, not prose.

### 4. Thematic Sections
Group items by policy domain rather than chronologically. Typical sections:
- AI & Regulatory Modernization
- Digital Assets & Crypto Regulation
- Fraud & Financial Crime
- Prudential Regulation & Other
- (Adapt to the organization's actual priority areas)

Within each section, order items by priority (HIGH before MODERATE before LOW). For sections with legislative tracking items (e.g., CLARITY Act, market structure bills), lead with the legislation status over individual hearings or statements — the legislative trajectory is what the team needs to see first.

Each item within a section gets:
- **Subheading** with title and priority tag
- **Source link** — a hyperlink to the original document (e.g., the agency press release, testimony, Federal Register notice). Revere items include both a primary source URL (`Link` field) and a Revere app URL (`View in Revere` field). Include the primary source link in the document so readers can go straight to the original. The link can be placed inline in the subheading, as a parenthetical after the source name in the body text, or as a standalone line under the subheading — pick whatever reads cleanly. For items sourced from web search rather than the primary intelligence tool, include the URL(s) of the source(s) you drew from. For items with multiple sources, use a pipe-separated format: "Sources: Source1 | Source2 | Source3".
- **Body paragraphs** — what happened, who said what, what the substance is. Write analytically, not as a press release summary.
- **WHY THIS MATTERS** block — shaded background, explains organizational relevance specifically. For LOW-priority items with no meaningful organizational engagement angle, the WHY THIS MATTERS block can be omitted entirely.

**Linking in docx-js:** Use the `ExternalHyperlink` class from docx-js. Import it alongside the other components. Example:
```javascript
const { ExternalHyperlink } = require("docx");

// Single source
function sourceLink(sourceName, url) {
  return new Paragraph({
    spacing: { before: 20, after: 60 },
    children: [
      new TextRun({ text: "Source: ", font: "Arial", size: 18, color: MED_GRAY }),
      new ExternalHyperlink({
        link: url,
        children: [new TextRun({ text: sourceName, font: "Arial", size: 18, color: BLUE, underline: {} })],
      }),
    ]
  });
}

// Multiple sources (pipe-separated)
new Paragraph({ spacing: { before: 20, after: 60 }, children: [
  new TextRun({ text: "Sources: ", font: "Arial", size: 18, color: MED_GRAY }),
  new ExternalHyperlink({ link: "https://...", children: [new TextRun({ text: "Source1", font: "Arial", size: 18, color: BLUE, underline: {} })] }),
  new TextRun({ text: " | ", font: "Arial", size: 18, color: MED_GRAY }),
  new ExternalHyperlink({ link: "https://...", children: [new TextRun({ text: "Source2", font: "Arial", size: 18, color: BLUE, underline: {} })] }),
] })
```

**Items sourced entirely from web search** (not from the primary intelligence source) that have no current action window may not need their own section. If a forthcoming rulemaking has been announced but the proposed rule hasn't dropped yet, a line item in Immediate Actions ("continue monitoring; prepare comment framework") is often sufficient. Save full sections for items where there is enough substance to analyze.

### 5. UPCOMING EVENTS Table
Two-column table: DATE | EVENT. Mark high-relevance events with ★. Include:
- Confirmed committee hearings with time, location, and relevance note
- Agency roundtables and public meetings
- Comment period deadlines (always with concrete calendar dates)
- Ongoing/rolling input opportunities (e.g., "SEC Crypto Task Force — written input via crypto@sec.gov")

### 6. Header/Footer
- Header: Organization name | document type | date range
- Footer: "Prepared for internal use" or equivalent confidentiality line

## Formatting Specifications

These are the tested values that produce a professional result. Do not deviate without reason.

### Colors
```
NAVY    = "1B3A5C"  (headings, header bar)
BLUE    = "2E75B6"  (section underlines, accent, hyperlinks)
LIGHT_GRAY = "F2F2F2"  (WHY THIS MATTERS shading)
DARK_GRAY  = "333333"  (body text)
MED_GRAY   = "666666"  (header secondary text, source labels)
```

### Priority Tag Colors
```
HIGH     = "C0392B" (red)
MODERATE = "E67E22" (amber)
LOW      = "27AE60" (green)
```

### Typography
- Font: Arial throughout (universally supported)
- Body: 10pt (size: 20 in docx-js half-points)
- Table body: 9pt (size: 18)
- Section headings: 13pt bold (size: 26), with blue bottom border
- Subheadings: 11pt bold (size: 22), with inline priority tag
- Title: 20pt bold (size: 40)

### Page Setup
- US Letter (12240 × 15840 DXA)
- Margins: 0.75 inches all sides (1080 DXA) — tighter than default to fit tables
- Table width: 9360 DXA (fills content area with 0.75" margins)

### Table Rules
- Always use `WidthType.DXA` (never percentage — breaks in Google Docs)
- Set both `columnWidths` on table AND `width` on each cell
- Use `ShadingType.CLEAR` (never SOLID — causes black backgrounds)
- Add cell margins: `{ top: 60, bottom: 60, left: 100, right: 100 }`
- Header rows: NAVY background with white text

## Pitfalls and Hard-Won Lessons

These are specific issues encountered during development. A fresh instance should read these before writing any code.

### Source Attribution
The biggest credibility risk is presenting web-searched context as if it came from the primary intelligence source. When supplementing the Revere pull with web search (e.g., legislative status, political dynamics, event schedules), mentally tag each piece of content with its origin. If the user asks "where did this come from?", you need a clean answer. A good practice: after gathering all material, mentally separate "from Revere" and "from web search" before writing. (The bookmark-vs-scored distinction within the Revere pull matters for prioritization, not for sourcing — both are primary intelligence.)

### Get the Content You Need, But Don't Over-Pull
A bare listing (titles, dates, links) is not enough to write an analytical "WHY THIS MATTERS" section — the full text often contains specific language, named officials, or procedural details that change the analysis. But you rarely need a separate `get_item` call per item to get there. Pass `include_content: true` on the `get_bookmarks` and `get_items` calls and the source content comes back with the listing. Combined with the tailored summary (already present for bookmarks; generated via `generate_tailored_summary` for the rest), that is usually enough to write the entry. Reserve `get_item` for the cases in Phase 1 Step 6 — where the summaries genuinely can't carry the entry or resolve a cluster — rather than calling it reflexively for every item.

### Don't Fabricate Coverage
If a topic the organization tracks had no developments in the date range, do not write a section about it. Mention it in Immediate Actions as "continue monitoring" if appropriate. The previous week's briefing is a reference for what to track, not a template to fill regardless of whether news occurred.

### Events Section Requires Active Research
Policy intelligence tools rarely capture upcoming events systematically. You must actively search for:
- Committee hearing schedules (check the actual committee website, not just news)
- Federal Register comment deadlines
- Agency-announced roundtables or public meetings
- Conferences where key officials are speaking

The events section is where web search adds the most value over the primary source alone.

### docx-js Specific Issues
- `\n` inside TextRun does NOT create line breaks — use separate Paragraph elements
- Never use unicode bullet characters — use `LevelFormat.BULLET` with numbering config
- `PageBreak` must be inside a Paragraph, not standalone
- Validate after generation — silent XML errors can produce files that open but render incorrectly
- Arrow character → works fine in TextRun text (no special handling)

### Priority Assessment
Priority should reflect the organization's specific engagement opportunity, not the general importance of the policy development. A Fed rate decision is important but LOW priority for a SupTech-focused org with no monetary policy lane. A niche CFTC no-action letter that sets precedent for software intermediation is HIGH if it maps to the org's regulatory design work.

**Err toward MODERATE over HIGH.** The Revere intelligence source often rates items as "High relevance" from an analytical interest perspective. The briefing's priority assessment is different — it reflects whether the organization should take action now. An item can be analytically fascinating and highly relevant to the org's mission but still MODERATE if the engagement window is not yet open, the item is in a scoping phase, or the org's role would be to monitor rather than engage. Reserve HIGH for items where there is a concrete action the team should take in the next two weeks with a defined counterparty (e.g., submit a comment letter, do staff outreach to a named office, prepare a deliverable for a specific deadline).

**Political dynamics matter for priority.** Two additional factors should push items toward MODERATE or lower:

- **Minority-party oversight actions** — letters, statements, and press releases from the minority party can generate headlines and force public responses, but they cannot move legislation, compel agency action, or open comment periods. Unless the item produces a concrete regulatory or legislative response (e.g., triggers a hearing, forces an agency to issue guidance, or surfaces a structural issue the org can analyze on neutral ground), default to MODERATE. The underlying analytical question may resurface later through a more actionable vehicle.
- **Items that would put the org in politically charged territory** — even when the substance is analytically interesting, if the surrounding debate is primarily partisan (e.g., allegations about specific political figures' financial interests, disputes over an administration's conduct toward an agency), engagement risks being read through a partisan lens regardless of intent. MODERATE is the right call with a "continue monitoring for a less fraught entry point" framing. The org's nonpartisan credibility is an asset worth protecting; don't spend it on items where the political noise drowns out the analytical signal.

### Editorial Tone
The briefing should read as an internal strategic memo from a colleague, not like an AI system explaining its reasoning. Key calibrations:

- **WHY THIS MATTERS blocks should sound human.** Avoid language that exposes the priority-rating machinery ("so MODERATE is the right call") — the team doesn't need to see how you rated an item; they just need to know what to do about it. Instead, use natural framing: "simply monitoring this for now is the right call" reads like a colleague's judgment. Similarly, avoid over-specifying the response form ("it could merit a focused research note") when a broader framing ("it could merit further action") leaves appropriate room for the team to decide.
- **WHY THIS MATTERS blocks should be analytical, not promotional.** Avoid language that oversells the org's position or implies certainty about impact. "This creates an opportunity" is better than "AIR is uniquely positioned to dominate this space." End on the concrete implication, not on a pitch.
- **Trim assertive claims about influence.** Phrases like "exactly when outside input has the most traction" or "staff outreach can shape the final product" overstate the org's leverage. Softer framing — "a scoping phase allowing opportunity for outside analytical input" — is more credible for an internal audience that knows the limits.
- **Don't attribute web-sourced claims to specific individuals unless you're confident.** If a Senator's statement was reported by trade press but you didn't pull it from an official source, attribute it generally ("Senators have signaled...") rather than naming the individual. If the user wants the attribution, they'll add it.
- **Immediate Actions are recommendations, not directives.** The team receiving this brief has context you don't. Frame actions as suggestions ("consider submitting," "evaluate whether," "outreach to X office to explore") rather than instructions.
- **Avoid specific day-of-week references.** "Crypto industry representatives reviewed the proposal on Thursday" ages poorly by Monday. Use "this week" or "stakeholders have been reviewing" instead. The brief should read cleanly for at least a week after distribution.
- **Keep WHY THIS MATTERS blocks tight.** If the last sentence restates something already implied or tacks on a tangential connection ("connecting to AIR's financial inclusion concerns"), cut it. End on the strongest analytical point, not a trailing afterthought.
- **Hedge on politically sensitive topics.** When recommending engagement with items that carry political risk (e.g., administration policy changes affecting equity-focused lending, DEI-adjacent compliance provisions), acknowledge the sensitivity before recommending action. A phrase like "While being mindful of political risks and concerns" before the recommendation signals that the team should weigh those dynamics, without the brief itself taking a political position.
- **Events table hygiene.** Drop carried-forward items from previous weeks once they are no longer actionable — past hearings, closed comment periods, and resolved monitoring items should not persist week over week.

## Reference Template

The full docx-js code template is in `references/template.md`. Read it before generating the document. It contains tested helper functions for every document element (section headings, body text, WHY THIS MATTERS blocks, priority-tagged subheadings, At-a-Glance table rows, and events table rows).

The template is a starting point — adapt section names, column widths, and content to the specific organization and week. The structural patterns and formatting values are stable and should not change.
