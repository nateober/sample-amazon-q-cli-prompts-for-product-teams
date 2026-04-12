# Persona-Aware Dashboards, Content Link Map, and Source Citations Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix three recurring prototype defects: single dashboard ignoring multiple personas, dead in-content links between screens, and unsourced market research data.

**Architecture:** Three independent fixes applied to steering files across Claude Code (`prompts/`) and Kiro (`.kiro/steering/`). Section 1 adds persona-dashboard decision logic. Section 2 adds a Content Link Map contract for subagent prompts (mirrors the Screen Manifest pattern). Section 3 adds source citation requirements to Market Research.

**Tech Stack:** Markdown steering files, JSON hooks configuration. No runtime code.

**Spec:** `docs/superpowers/specs/2026-04-11-persona-dashboards-content-links-sources-design.md`

---

## File Map

| File | Action | Responsibility |
|------|--------|---------------|
| `prompts/PRD Creation Guide.md` | Modify | Add persona dashboard widgets to persona definition |
| `prompts/Prototype Creation Guide.md` | Modify | New Step 2.7 (persona analysis), Step 4.6 (Content Link Map), Step 4.5.3 item #7, Step 9.5 check #8 |
| `prompts/Orchestrator.md` | Modify | Phase A step 4.5 (persona analysis), Content Link Map step + subagent template block, Phase C check #6 |
| `prompts/Market Research Agent.md` | Modify | Source citation requirement + HTML output format |
| `prompts/Claude_Code_Workflow.md` | Modify | Content Link Map and source citation in checkpoints |
| `CLAUDE.md` | Modify | Add Content Link Map to prototype structure |
| `.kiro/steering/prototype-guide.md` | Modify | Content Link Map, dead link validation |
| `.kiro/steering/product-workflow.md` | Modify | Source citation in Market Research validation |
| `.kiro/hooks.json` | Modify | Enhance link checker + Market Research Validator |

---

### Task 1: PRD Creation Guide — Add persona dashboard widgets requirement

**Files:**
- Modify: `prompts/PRD Creation Guide.md:106-114`

- [ ] **Step 1: Add dashboard widgets to persona definition**

Find the "For each persona, define:" list in Step 2 (line 106):

```
**For each persona, define:**
- **Name**: Use real names from `user_provided_context.team_members` if available
- **Role/Title**: Professional context
- **Demographics**: Relevant background info
- **Goals**: What they're trying to achieve
- **Pain Points**: Current frustrations (from market research)
- **Day in the Life**: Typical workflow narrative
- **Success Criteria**: How they measure success
- **Quote**: Representative voice of this persona
```

Replace with:

```
**For each persona, define:**
- **Name**: Use real names from `user_provided_context.team_members` if available
- **Role/Title**: Professional context
- **Demographics**: Relevant background info
- **Goals**: What they're trying to achieve
- **Pain Points**: Current frustrations (from market research)
- **Day in the Life**: Typical workflow narrative
- **Success Criteria**: How they measure success
- **Quote**: Representative voice of this persona
- **Primary Dashboard Widgets/KPIs**: What this persona needs to see on their dashboard — list 3-5 key widgets, metrics, or actions (e.g., "Student progress charts," "Upcoming deadlines," "Admin user management table"). This data is used by the Prototype Agent to decide whether personas need separate dashboards.
```

- [ ] **Step 2: Add dashboard widgets to the persona output schema**

Find the persona object in the output schema (around line 49):

```
    "personas": [
      {
        "name": "string",
        "role": "string",
        "primary_need": "string",
        "key_workflow": "string"
      }
    ],
```

Replace with:

```
    "personas": [
      {
        "name": "string",
        "role": "string",
        "primary_need": "string",
        "key_workflow": "string",
        "dashboard_widgets": ["string (e.g., 'Student progress charts', 'Admin user table')"]
      }
    ],
```

- [ ] **Step 3: Commit**

```bash
git add "prompts/PRD Creation Guide.md"
git commit -m "PRD Guide: add persona dashboard widgets requirement for prototype dashboard splitting"
```

---

### Task 2: Prototype Creation Guide — Add Step 2.7 persona-dashboard analysis

**Files:**
- Modify: `prompts/Prototype Creation Guide.md:201` (after Step 2.5, before Step 1.1)

- [ ] **Step 1: Add Step 2.7 after the Design Token Contract section**

Find the end of Step 2.5 (the line that reads "This contract block is included in every subagent prompt alongside the Screen Manifest and Brand Assets blocks." at line 201). After that line, and before "### Step 1.1: Research Customer Brand" (line 203), add:

```

### Step 2.7: Persona-Dashboard Analysis (REQUIRED Before Screen Manifest)

Before creating the screen manifest (Step 4.5), analyze PRD personas to decide whether to create one dashboard or multiple persona-specific dashboards.

**Process:**
1. List each persona's `dashboard_widgets` from the PRD (the widgets, KPIs, and actions they need)
2. Compare across personas — count how many widgets are shared vs. unique
3. **Decision rule:**
   - If personas share **>70%** of dashboard content → one dashboard with role-specific sections (e.g., tabs, collapsible panels)
   - If personas share **<70%** of dashboard content → create separate dashboard screens per persona (e.g., `Screen_Dashboard_Teacher`, `Screen_Dashboard_Admin`)
4. Document the decision and reasoning before proceeding to the screen manifest

**Output:** Either one `Screen_Dashboard` entry or multiple persona-specific entries — these feed directly into the screen manifest (Step 4.5).

**Example analysis:**
```
Personas: Teacher, Admin, Student
Teacher widgets: Student progress, Assignment grades, Upcoming deadlines, Class roster
Admin widgets: System health, User management, License usage, Audit log, Analytics
Student widgets: My grades, Study progress, Upcoming exams, Flashcard decks

Shared across all 3: 0 widgets (0%)
Teacher-Admin overlap: 0 widgets (0%)
Decision: <70% overlap → create 3 separate dashboards
  - Screen_Dashboard_Teacher_[Product]_[Date].html
  - Screen_Dashboard_Admin_[Product]_[Date].html
  - Screen_Dashboard_Student_[Product]_[Date].html
```

This step ensures that each persona gets a dashboard tailored to their workflow rather than a single dashboard that serves no one well.
```

- [ ] **Step 2: Commit**

```bash
git add "prompts/Prototype Creation Guide.md"
git commit -m "Prototype Guide: add Step 2.7 persona-dashboard analysis for multi-persona prototypes"
```

---

### Task 3: Orchestrator.md — Add persona analysis step to Phase A

**Files:**
- Modify: `prompts/Orchestrator.md:275-276` (between step 4 and step 5)

- [ ] **Step 1: Insert step 4.5 for persona analysis**

Find the line for step 5 (Create the screen manifest) at line 276:

```
5. **Create the screen manifest** — a list of EXACT filenames, one per screen:
```

Insert before that line:

```
4.5. **Analyze PRD personas for dashboard splitting** — For each persona in the PRD, list their `dashboard_widgets`. If personas share <70% of dashboard content, plan separate dashboard screens (e.g., `Screen_Dashboard_Teacher`, `Screen_Dashboard_Admin`). If >70% overlap, plan one dashboard with role-specific sections. Document the decision. The resulting screen list feeds into the screen manifest (step 5).
```

- [ ] **Step 2: Commit**

```bash
git add prompts/Orchestrator.md
git commit -m "Orchestrator: add step 4.5 persona-dashboard analysis before screen manifest"
```

---

### Task 4: Prototype Creation Guide — Add Step 4.6 Content Link Map

**Files:**
- Modify: `prompts/Prototype Creation Guide.md:559-563` (between Step 4.5.3 and Step 5)

- [ ] **Step 1: Add Step 4.6 after Step 4.5.3**

Find the `---` separator between Step 4.5 and Step 5 (line 561). Replace it with the new section:

```
---

### Step 4.6: Create Content Link Map (REQUIRED Before Building Screens)

After the screen manifest (Step 4.5), create a **Content Link Map** — a list of expected in-content links between screens. This prevents dead links (`href="#"`) in dashboard cards, action buttons, CTAs, and table row actions.

**How to create:**
1. Read the PRD user flows — every step that crosses a screen boundary becomes an entry
2. For each screen, list what content-area elements should link to which target screens
3. Use EXACT filenames from the screen manifest for all targets

**Content Link Map format:**
```
CONTENT LINK MAP
────────────────
Screen_Dashboard → "View Exam Results" card → Screen_ExamResults_[Product]_[Date].html
Screen_Dashboard → "Start Study Module" button → Screen_StudyModule_[Product]_[Date].html
Screen_Dashboard → "Take Mock Exam" card → Screen_MockExam_[Product]_[Date].html
Screen_ExamResults → "Review Domain" button → Screen_StudyModule_[Product]_[Date].html
Screen_StudyModule → "Take Practice Quiz" CTA → Screen_DomainQuiz_[Product]_[Date].html
Screen_DomainQuiz → "Back to Study Module" → Screen_StudyModule_[Product]_[Date].html
```

Each subagent receives a filtered view showing only their screen's outbound links (see Step 4.5.3 item #7).

**Rules:**
- Every actionable element (card, button, CTA, table row action) that logically leads to another screen MUST have a Content Link Map entry
- Target filenames MUST match the screen manifest exactly
- Do NOT use `href="#"` or `javascript:void(0)` for any element that should navigate to another screen

---

```

- [ ] **Step 2: Add item #7 to Step 4.5.3's "must include" list**

Find the numbered list in Step 4.5.3 (line 547). After item 6 (Design Token Contract), add:

```
7. **The Content Link Map entries for this screen** — the specific in-content links (dashboard cards, action buttons, CTAs) that should navigate to other screens, with exact target filenames. Subagents must wire these into their page content. Do NOT use `href="#"` or `javascript:void(0)` for any element that should navigate.
```

- [ ] **Step 3: Update the explicit instruction quote**

Find the quoted instruction in Step 4.5.3 (line 557):

```
> "Use ONLY filenames from the manifest for all href links. Do NOT rename, abbreviate, or invent alternative filenames. Paste the sidebar nav HTML VERBATIM — only add 'active' to your screen's nav item. Use var(--variable-name) for ALL colors — never hardcode hex values. Use component classes from the Design Token Contract instead of writing custom styles."
```

Replace with:

```
> "Use ONLY filenames from the manifest for all href links. Do NOT rename, abbreviate, or invent alternative filenames. Paste the sidebar nav HTML VERBATIM — only add 'active' to your screen's nav item. Use var(--variable-name) for ALL colors — never hardcode hex values. Use component classes from the Design Token Contract instead of writing custom styles. Wire all Content Link Map entries into your page content — do NOT use href='#' or javascript:void(0) for elements that should navigate."
```

- [ ] **Step 4: Commit**

```bash
git add "prompts/Prototype Creation Guide.md"
git commit -m "Prototype Guide: add Step 4.6 Content Link Map and update Step 4.5.3 with item #7"
```

---

### Task 5: Prototype Creation Guide — Add Step 9.5 check #8 Content Link Audit

**Files:**
- Modify: `prompts/Prototype Creation Guide.md:1109-1111` (after check #7 Visual Consistency)

- [ ] **Step 1: Add check #8 after the Visual Consistency Check**

Find the end of check #7 (the line "**e. Fix violations:**..." ending around line 1109), then the `---` separator. Before the `---`, add:

```

#### 8. Content Link Audit

Verify that in-content links (dashboard cards, action buttons, CTAs, table row actions) connect to the correct screens:

**a. Scan for dead links:**
```bash
grep -rn 'href="#"' documents/Screen_*.html
grep -rn 'javascript:void' documents/Screen_*.html
```
Flag any matches — these are elements that should navigate but don't.

**b. Verify Content Link Map entries:**
For each entry in the Content Link Map, verify the source screen contains an `<a>` or `<button>` element with the correct `href` to the target screen filename.

**c. Flag missing links:**
Any Content Link Map entry with no matching element in the source screen is a missing in-content link. Add it.

**d. Fix dead links:** Replace `href="#"` and `javascript:void(0)` with the correct target filename from the Content Link Map or screen manifest.
```

- [ ] **Step 2: Commit**

```bash
git add "prompts/Prototype Creation Guide.md"
git commit -m "Prototype Guide: add Step 9.5 check #8 Content Link Audit for dead links"
```

---

### Task 6: Orchestrator.md — Add Content Link Map to Phase A, subagent template, and Phase C

**Files:**
- Modify: `prompts/Orchestrator.md` (Phase A, Phase B template, Phase C)

- [ ] **Step 1: Add Content Link Map step to Phase A**

Find step 8 in Phase A (line 311, "Compile the Design Token Contract block"). After it, add:

```
9. **Create the Content Link Map** — derived from PRD user flows. For each screen, list content-area elements (dashboard cards, action buttons, CTAs, table row actions) that should link to other screens. Use EXACT filenames from the manifest:
   ```
   CONTENT LINK MAP
   ────────────────
   Screen_Dashboard → "View Details" card → Screen_[Target]_[Product]_[Date].html
   Screen_Dashboard → "Start Module" button → Screen_[Target]_[Product]_[Date].html
   Screen_[Source] → "[Element]" → Screen_[Target]_[Product]_[Date].html
   ```
```

- [ ] **Step 2: Add Content Link Map to Hard Gate checklist**

Find the Hard Gate checklist (around line 313). Add a new item:

```
- [ ] Content Link Map with in-content links per screen
```

- [ ] **Step 3: Add Content Links block to the subagent prompt template**

In the Phase B subagent prompt example, find the SCREEN REQUIREMENTS section (around line 404):

```
SCREEN REQUIREMENTS
───────────────────
[paste PRD requirements for this specific screen]
```

Insert before SCREEN REQUIREMENTS:

```
CONTENT LINKS FROM YOUR SCREEN (wire these into your page content)
──────────────────────────────────────────────────────────────────
[paste this screen's entries from the Content Link Map, e.g.:]
"View Exam Results" card → href="Screen_ExamResults_SmartSearch_2026-04-05.html"
"Start Study Module" button → href="Screen_StudyModule_SmartSearch_2026-04-05.html"

Every actionable element (card, button, CTA, table row action) that logically leads
to another screen MUST use the href above. Do NOT use href="#" or javascript:void(0)
for any element that should navigate to another screen.

```

- [ ] **Step 4: Add rule to RULES section**

In the RULES section (around line 392), add:

```
- Wire all Content Link Map entries into your page content — do NOT use href="#" or javascript:void(0) for navigation elements
```

- [ ] **Step 5: Add Content Link Audit to Phase C**

Find Phase C item 5 (visual consistency check, line 434). After it, add:

```
6. **Content link audit:** Scan all screen files for `href="#"` and `javascript:void(0)` — flag as dead links. For each Content Link Map entry, verify the source screen contains an element with the correct href to the target. Fix dead links with correct filenames.
```

- [ ] **Step 6: Update Key Points list**

Find the Key Points list (around line 409). Add:

```
- Content Link Map entries tell the subagent which in-content elements (cards, buttons, CTAs) must link to which screens — no dead links
```

- [ ] **Step 7: Update "Everything else stays identical" line**

Find: `Everything else stays identical across all subagent prompts: CSS link, manifest, nav HTML, brand assets, Design Token Contract, rules.`

Replace with: `Everything else stays identical across all subagent prompts: CSS link, manifest, nav HTML, brand assets, Design Token Contract, Content Link Map, rules.`

- [ ] **Step 8: Update "Repeat this template" section**

Find the list starting "Repeat this template for each screen, changing only:" (around line 418). Add a new bullet:

```
- CONTENT LINKS FROM YOUR SCREEN (the relevant entries from the Content Link Map)
```

- [ ] **Step 9: Commit**

```bash
git add prompts/Orchestrator.md
git commit -m "Orchestrator: add Content Link Map to Phase A, subagent template, and Phase C validation"
```

---

### Task 7: Market Research Agent — Add source citation requirement

**Files:**
- Modify: `prompts/Market Research Agent.md`

- [ ] **Step 1: Add source citation requirement**

The file already has a `research_sources` array in the final output schema (around line 291). But there's no requirement to embed sources as clickable links in the HTML output.

Find the "## Final Output Format" section (around line 267). After the closing `}` of the JSON schema and before any trailing content, add:

```

## Source Citations in HTML Output (REQUIRED)

When generating the Market Research HTML file (`MarketResearch_[Product]_[Date].html`), every data claim must include a clickable source citation:

**Citation format:** Superscript numbers linking to a Sources section at the bottom:

```html
<!-- Inline citation -->
The global K-12 edtech market is valued at $18.2B<sup><a href="https://source-url">[1]</a></sup>

<!-- Competitor reference -->
<td>Quizlet</td><td>$7.99/mo<sup><a href="https://quizlet.com/pricing">[3]</a></sup></td>

<!-- Sources section at bottom of document -->
<h2>Sources</h2>
<ol>
  <li><a href="https://source-url" target="_blank">Report Title - Publisher, Date</a></li>
  <li><a href="https://source-url" target="_blank">Article Title - Publication, Date</a></li>
</ol>
```

**What needs citations:**
- Every TAM/SAM/SOM figure — link to the report, article, or database
- Every competitor entry — link to the competitor's website or product page
- Competitor pricing — link to the pricing page
- Customer pain points — link to the forum, review site, or article
- Market trends — link to the analyst report or news article
- Growth rates and statistics — link to the data source

**Requirements:**
- Sources section at bottom of document with numbered references
- All citation links open in new tab (`target="_blank"`)
- Each `research_sources` entry in the JSON output must also appear as an inline citation in the HTML
- No unsourced data claims — if you can't find a source, say "estimated" and explain the basis
```

- [ ] **Step 2: Commit**

```bash
git add "prompts/Market Research Agent.md"
git commit -m "Market Research Agent: add mandatory source citation requirement for HTML output"
```

---

### Task 8: Claude_Code_Workflow.md — Add Content Link Map and source citations to checkpoints

**Files:**
- Modify: `prompts/Claude_Code_Workflow.md`

- [ ] **Step 1: Add Content Link Map to Phase 4 checkpoint**

Find the Phase 4 checkpoint list (around line 236). After the line about Design Token Contract, add:

```
- [ ] Content Link Map created (in-content links between screens)
- [ ] No dead links (`href="#"`, `javascript:void`) in screen content
```

- [ ] **Step 2: Add source citation to Phase 1 checkpoint**

Find the Phase 1 (Market Research) checkpoint list (around line 106). After "No placeholder text" line, add:

```
- [ ] Every data claim has a source citation link
- [ ] Sources section at bottom of document with numbered references
```

- [ ] **Step 3: Commit**

```bash
git add "prompts/Claude_Code_Workflow.md"
git commit -m "Claude_Code_Workflow: add Content Link Map and source citation checkpoints"
```

---

### Task 9: CLAUDE.md — Add Content Link Map mention

**Files:**
- Modify: `CLAUDE.md:60-66`

- [ ] **Step 1: Add Content Link Map to build order**

Find the build order list. Between step 4 (Screen manifest) and step 5 (Screen files), add a note about content links. Change step 4 from:

```
4. Screen manifest + sidebar nav template — exact filenames, verbatim nav HTML
```

To:

```
4. Screen manifest + sidebar nav template + Content Link Map — exact filenames, nav HTML, in-content links between screens
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "CLAUDE.md: add Content Link Map to prototype build order"
```

---

### Task 10: Kiro prototype-guide.md — Add Content Link Map and dead link validation

**Files:**
- Modify: `.kiro/steering/prototype-guide.md`

- [ ] **Step 1: Add Content Link Map to the build order note**

Find the build order list (around line 294). Change step 4 from:

```
4. Screen manifest + sidebar nav template
```

To:

```
4. Screen manifest + sidebar nav template + Content Link Map
```

- [ ] **Step 2: Add Content Link Map to screen builder pass-through**

Find the "Step 3: Pass to every screen builder" line (around line 423). It currently mentions "available CSS class names, and the **Design Token Contract**". Append after the Design Token Contract clause:

Add to the end of that sentence, before the period: `, and the **Content Link Map entries** for that screen (in-content links to other screens — no dead links)`

- [ ] **Step 3: Add dead link check to Post-Build Validation**

Find the Visual Consistency Check (section 5, around line 471). After its last item, add a new section:

```

### 6. Content Link Audit

1. **Grep for dead links:** `grep -rn 'href="#"' Screen_*.html` and `grep -rn 'javascript:void' Screen_*.html` — flag all matches
2. **Verify Content Link Map entries:** For each entry, confirm the source screen has an element with the correct href
3. **Fix:** Replace dead links with correct filenames from the Content Link Map

```

Then renumber the existing sections that follow:
- What was `### 6. Logo & Brand Verification` becomes `### 7. Logo & Brand Verification`
- What was `### 7. Quick Smoke Test` becomes `### 8. Quick Smoke Test`

- [ ] **Step 4: Commit**

```bash
git add .kiro/steering/prototype-guide.md
git commit -m "Kiro prototype-guide: add Content Link Map and dead link validation"
```

---

### Task 11: Kiro product-workflow.md — Add source citation validation

**Files:**
- Modify: `.kiro/steering/product-workflow.md`

- [ ] **Step 1: Add source citation to Market Research validation**

Find the Market Research validation section. Look for the checklist items about Market Research. After the brand assets checks, add:

```
- [ ] Every data claim (TAM/SAM/SOM, competitor pricing, trends) has a source citation link
- [ ] Sources section exists at bottom with numbered references
- [ ] Competitor entries link to their websites
```

- [ ] **Step 2: Commit**

```bash
git add .kiro/steering/product-workflow.md
git commit -m "Kiro product-workflow: add source citation checks to Market Research validation"
```

---

### Task 12: Kiro hooks.json — Enhance link checker and Market Research Validator

**Files:**
- Modify: `.kiro/hooks.json`

- [ ] **Step 1: Enhance the Interactivity & Link Checker hook**

Find the "Interactivity & Link Checker" hook (around line 184). In its prompt string, after the navigation checks, add:

```
\n**Dead Link Detection:**\n6. Grep for href=\"#\" in all Screen_*.html — flag as dead links\n7. Grep for javascript:void in all Screen_*.html — flag as dead links\n8. Every actionable element (card, button, CTA) should link to a real screen, not # or void\n
```

- [ ] **Step 2: Enhance the Market Research Validator hook**

Find the "Market Research Validator" hook (around line 1). In its prompt string, after the existing checklist, add:

```
\n**Source Citations (REQUIRED):**\n- [ ] Sources section exists at bottom of document\n- [ ] TAM/SAM/SOM figures have citation links\n- [ ] Competitor entries link to their websites\n- [ ] No unsourced data claims (figures without citations)\n
```

- [ ] **Step 3: Verify JSON is valid**

```bash
python3 -c "import json; json.load(open('.kiro/hooks.json')); print('Valid JSON')"
```

Expected: `Valid JSON`

- [ ] **Step 4: Commit**

```bash
git add .kiro/hooks.json
git commit -m "Kiro hooks: add dead link detection and market research source citation checks"
```

---

### Task 13: Final verification

- [ ] **Step 1: Grep for "Content Link Map" across all modified files**

```bash
grep -rn "Content Link Map" prompts/Orchestrator.md "prompts/Prototype Creation Guide.md" CLAUDE.md .kiro/steering/prototype-guide.md "prompts/Claude_Code_Workflow.md"
```

Expected: References in all 5 files.

- [ ] **Step 2: Grep for "dashboard_widgets" in PRD guide and handoff schema**

```bash
grep -rn "dashboard_widgets" "prompts/PRD Creation Guide.md" "prompts/Handoff Schema.md"
```

Expected: At least 1 match in PRD Creation Guide.

- [ ] **Step 3: Grep for "Source" or "citation" in Market Research Agent**

```bash
grep -rn "citation\|Sources section" "prompts/Market Research Agent.md"
```

Expected: Multiple matches in the new section.

- [ ] **Step 4: Grep for dead link patterns mentioned in validation**

```bash
grep -rn 'href="#"\|javascript:void' prompts/Orchestrator.md "prompts/Prototype Creation Guide.md" .kiro/steering/prototype-guide.md
```

Expected: Matches in validation sections of all 3 files.

- [ ] **Step 5: Validate hooks.json**

```bash
python3 -c "import json; json.load(open('.kiro/hooks.json')); print('Valid JSON')"
```

Expected: `Valid JSON`

- [ ] **Step 6: Final git log**

```bash
git log --oneline -15
```

Expected: ~12 new commits on the branch.
