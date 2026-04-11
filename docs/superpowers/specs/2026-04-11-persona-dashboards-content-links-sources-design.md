# Persona-Aware Dashboards, Content Link Map, and Source Citations

**Date:** 2026-04-11
**Status:** Approved
**Trigger:** Post-build review of multiple prototypes — three recurring defects

## Problems

Three defects observed across prototype builds:

1. **Single dashboard for multiple personas** — The prototype creates one Dashboard screen regardless of how many personas the PRD defines. When personas have divergent needs (Teacher vs Admin vs Student), a single dashboard either serves none of them well or becomes an incoherent mashup of unrelated widgets.

2. **Content-area links don't connect to related screens** — The sidebar nav contract ensures navigation menus work, but in-content links (dashboard cards with "View Details," action buttons in tables, contextual CTAs like "Start Study Module") use `href="#"` or `javascript:void(0)` instead of linking to the correct screen. Subagents don't know what content-area links should point where.

3. **Market research lacks source citations** — The Market Research HTML references data (TAM/SAM/SOM figures, competitor pricing, pain points) but doesn't link back to the sources. Reviewers can't verify claims.

## Solution

### Section 1: Persona-Aware Dashboard Splitting

Add a decision step before the screen manifest where the main agent analyzes PRD personas and decides whether to create one dashboard or multiple persona-specific dashboards.

**New Step 2.7 in Prototype Creation Guide (between Step 2.5 Token Contract and Step 3 Map User Flows):**

Before creating the screen manifest, analyze PRD personas:
1. List each persona's dashboard needs (widgets, KPIs, primary actions)
2. Score overlap: if personas share >70% of dashboard content, one dashboard with role-specific sections is sufficient. If <70% overlap, create separate dashboard screens per persona.
3. Output: either one `Screen_Dashboard` entry or multiple persona-specific entries (`Screen_Dashboard_Teacher`, `Screen_Dashboard_Admin`, etc.) — these feed into the screen manifest.

This is a decision step, not a hard rule. The agent evaluates and decides, but must document the reasoning.

**Orchestrator Phase A — new step 4.5:**

Between "Extract Design Token Contract" (step 4) and "Create screen manifest" (step 5):
- Analyze PRD personas. For each persona, list their primary dashboard actions/widgets.
- If persona needs diverge significantly (shared dashboard content <70%), plan separate dashboard screens.
- Document the decision and reasoning.
- The resulting screen list feeds into the screen manifest (step 5).

**PRD Creation Guide — persona dashboard requirements:**

Add to each persona section a required field: "Primary dashboard widgets/KPIs" — a list of what this persona needs to see on their dashboard. This gives the prototype builder the data to make the split decision.

### Section 2: Content Link Map

Mirror the Screen Manifest pattern for in-content links. Before dispatching subagents, the main agent creates a Content Link Map derived from PRD user flows. Each subagent receives their screen's outbound content links.

**Content Link Map (created in Phase A, after screen manifest):**

```
CONTENT LINK MAP (paste into each subagent prompt)
──────────────────────────────────────────────────
Screen_Dashboard → "View Exam Results" card → Screen_ExamResults
Screen_Dashboard → "Start Study Module" button → Screen_StudyModule
Screen_Dashboard → "Take Mock Exam" card → Screen_MockExam
Screen_ExamResults → "Review Domain" button → Screen_StudyModule
Screen_StudyModule → "Take Practice Quiz" CTA → Screen_DomainQuiz
Screen_DomainQuiz → "Back to Study Module" → Screen_StudyModule
```

Derived from PRD user flows: every directional step in a user flow that crosses screen boundaries becomes an entry.

**Subagent prompt addition — content links for each screen:**

Each subagent receives a filtered view showing only their screen's outbound links:

```
CONTENT LINKS FROM YOUR SCREEN (wire these into your page content)
──────────────────────────────────────────────────────────────────
"View Exam Results" card → href="Screen_ExamResults_CloudReady_2026-04-05.html"
"Start Study Module" button → href="Screen_StudyModule_CloudReady_2026-04-05.html"
"Take Mock Exam" card → href="Screen_MockExam_CloudReady_2026-04-05.html"

Every actionable element (card, button, CTA, table row action) that logically leads
to another screen MUST use the href above. Do NOT use href="#" or javascript:void(0)
for any element that should navigate to another screen.
```

**Prototype Creation Guide — Step 4.6: Create Content Link Map:**

After the screen manifest (Step 4.5) and before building screens (Step 5):
1. Read PRD user flows
2. Map each flow step that crosses a screen boundary to a source screen → element description → target screen entry
3. Produce the Content Link Map
4. Add to Step 4.5.3's "must include" list as item #7

**Post-build validation — Step 9.5 check #8: Content Link Audit:**

```
8. Content Link Audit
   a. Grep all Screen_*.html for href="#" and href="javascript:void(0)" — flag as dead links
   b. For each entry in the Content Link Map, verify the source screen contains
      an <a> or <button> with the correct href to the target screen
   c. Flag any Content Link Map entry with no matching element in the source screen
   d. Fix: replace dead links with correct hrefs from the Content Link Map
```

**Orchestrator Phase C — add check #6:**

Scan all screen files for `href="#"` and `href="javascript:void(0)"`. Verify Content Link Map entries have matching elements. Fix dead links.

**Kiro hooks.json — enhance Interactivity & Link Checker:**

Add dead link detection: `href="#"`, `javascript:void(0)`, `javascript:void`. Flag any in-content element that should navigate but doesn't.

### Section 3: Market Research Source Citations

Require every data point in Market Research HTML to include a clickable source link.

**Market Research Agent.md — source citation requirement:**

Every data claim must include an inline citation linking to its source:
- TAM/SAM/SOM figures: link to the report, article, or database
- Competitor info: link to the competitor's website, pricing page, or product page
- Customer pain points: link to the forum, review site, or article
- Market trends: link to the analyst report or news article

Format: superscript citation numbers linking to a Sources section at the bottom:

```html
The global K-12 edtech market is valued at $18.2B<sup><a href="https://source-url">[1]</a></sup>
...
<h2>Sources</h2>
<ol>
  <li><a href="https://source-url" target="_blank">Report Title - Publisher, Date</a></li>
</ol>
```

Requirements:
- Sources section at bottom of document with numbered references
- Every TAM/SAM/SOM figure has a citation
- Every competitor entry links to the competitor's website
- Citation links open in new tab (`target="_blank"`)

**Kiro hooks.json — enhance Market Research Validator:**

Add:
- Sources section exists with numbered references
- Each TAM/SAM/SOM figure has a citation link
- Each competitor entry links to their website
- No unsourced data claims

**Kiro product-workflow.md — Market Research validation:**

Add source citation checks to the existing Market Research validation checklist.

## File Change Map

| File | Sec 1 (Dashboards) | Sec 2 (Content Links) | Sec 3 (Sources) |
|------|--------------------|-----------------------|-----------------|
| `prompts/Orchestrator.md` | Step 4.5 persona analysis | Content Link Map in Phase A + subagent template, Phase C check #6 | -- |
| `prompts/Prototype Creation Guide.md` | Step 2.7 persona-dashboard analysis | Step 4.6 Content Link Map, Step 4.5.3 item #7, Step 9.5 check #8 | -- |
| `prompts/PRD Creation Guide.md` | Persona dashboard widgets requirement | -- | -- |
| `prompts/Market Research Agent.md` | -- | -- | Source citation requirement + output template |
| `prompts/Claude_Code_Workflow.md` | -- | Content Link Map in checkpoint | Source citation in checkpoint |
| `CLAUDE.md` | -- | Brief mention | -- |
| `.kiro/steering/prototype-guide.md` | -- | Content Link Map, dead link validation | -- |
| `.kiro/steering/product-workflow.md` | -- | -- | Source citation validation |
| `.kiro/hooks.json` | -- | Enhance link checker | Enhance Market Research Validator |

## Design Decisions

**Why a decision step for dashboards, not a hard rule?** Some products genuinely need one unified dashboard (e.g., a tool with one primary user type and a secondary admin). Others need persona-specific views. The 70% overlap heuristic guides the decision but the agent must evaluate context.

**Why a Content Link Map, not just "link everything"?** Without an explicit map, subagents don't know which elements should link where. "Link everything that should link" is too vague — the map makes it concrete and verifiable, just like the Screen Manifest makes filenames concrete.

**Why superscript citations, not inline URLs?** Inline URLs clutter the document and break reading flow. Superscript numbers with a collected Sources section is standard academic/business practice and keeps the HTML clean.
