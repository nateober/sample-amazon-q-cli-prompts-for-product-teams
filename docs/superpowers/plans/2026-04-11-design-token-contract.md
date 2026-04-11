# Design Token Contract & Visual Consistency Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Prevent theme divergence in parallel-built prototype screens by adding a Design Token Contract to subagent prompts, enforcing build order, and adding post-build visual consistency validation.

**Architecture:** Three layers applied to 7 steering files across Claude Code (`prompts/`, `CLAUDE.md`) and Kiro (`.kiro/steering/`, `.kiro/hooks.json`). Layer 1 adds a Design Token Contract template to subagent prompts. Layer 2 enforces Design System creation before screens. Layer 3 adds grep-based visual consistency checks. No CSS is defined in steering files — only contract templates with placeholders filled at build time.

**Tech Stack:** Markdown steering files, JSON hooks configuration. No runtime code.

**Spec:** `docs/superpowers/specs/2026-04-11-design-token-contract-design.md`

---

## File Map

| File | Action | Responsibility |
|------|--------|---------------|
| `prompts/Orchestrator.md` | Modify | Phase A reorder, hard gate, token contract in subagent template, var() in brand assets, Phase C check #5 |
| `prompts/Prototype Creation Guide.md` | Modify | New Steps 2 and 2.5, update Step 3.5.3, new Step 8.5 check #7 |
| `CLAUDE.md` | Modify | Update Prototype Structure section with strict build order |
| `.kiro/steering/prototype-guide.md` | Modify | Reorder output files, add hard gate, add token contract pass-through, add validation check |
| `.kiro/steering/specialist-prototype.md` | Modify | Add CSS Variable Usage mandate section |
| `.kiro/steering/specialist-design-system.md` | Modify | Add "create BEFORE screens" build order note |
| `.kiro/hooks.json` | Modify | Update phase transition prompt, enhance Screen Validator |

---

### Task 1: Orchestrator.md — Reorder Phase A and add hard gate

**Files:**
- Modify: `prompts/Orchestrator.md:254-303` (Phase A section)

- [ ] **Step 1: Rewrite the Phase A section header and reorder steps**

Replace the current Phase A section (lines 254–302) with the new order. The existing steps 1-5 become steps 1, 2, 5, 6, 7. New steps 3, 4, 8 are added.

Find this text at line 254:
```
#### Phase A: Build Shared Resources (main agent, BEFORE any screen delegation)

1. **Create shared CSS file** (`[product-slug].css`) — write to `./documents/`
2. **Resolve brand assets** (if building for a known company):
   - Identify the CUSTOMER company from `customer_company.name` in the session state
   - Follow the Logo Discovery Protocol in `Prototype Creation Guide.md` Step 1.1
   - **You MUST pass the Logo Gate** (all 5 checks) before using any logo:
     1. HTTP 200
     2. File size 2KB–50KB
     3. Downloaded and LOOKED AT the image
     4. Image shows the CUSTOMER's brand (not a partner/sponsor/competitor)
     5. Stated: "This logo belongs to [Customer] because [reason]"
   - **If the gate fails for all candidates → ask the user for a logo URL.** Use a text placeholder until they provide one. Do NOT guess.
   - Extract the customer's brand colors and typography from THEIR website
   - Record the gate-verified logo URL, brand colors, and fonts — these become part of the shared contract
   - **Subagents must NOT search for logos themselves.** The resolved brand assets are final.
3. **Create the screen manifest** — a list of EXACT filenames, one per screen:
   ```
   SCREEN MANIFEST (copy verbatim into every subagent prompt):
   ─────────────────────────────────────────────────────────
   CSS file: [product-slug].css
   
   Screens:
   1. Screen_Dashboard_[Product]_[Date].html       (entry point)
   2. Screen_[Name2]_[Product]_[Date].html
   3. Screen_[Name3]_[Product]_[Date].html
   ...
   ─────────────────────────────────────────────────────────
   ```
4. **Create the sidebar nav HTML** — the exact `<nav>` block every screen must use:
   ```html
   <!-- SIDEBAR NAV — paste verbatim, only change which item gets class="active" -->
   <nav class="sidebar-nav">
     <a class="nav-item" href="Screen_Dashboard_[Product]_[Date].html">Dashboard</a>
     <a class="nav-item" href="Screen_[Name2]_[Product]_[Date].html">[Label2]</a>
     <a class="nav-item" href="Screen_[Name3]_[Product]_[Date].html">[Label3]</a>
   </nav>
   ```
5. **Compile the brand assets block** (if applicable) — this gets pasted into every subagent prompt:
   ```
   BRAND ASSETS (use these exactly — do NOT search for alternatives)
   ──────────────────────────────────────────────────────────────────
   Customer: [Company Name]
   Logo URL: [verified URL that returned HTTP 200]
   Logo placement: <img src="[verified URL]" alt="[Company Name] logo" class="header-logo">
   Brand colors: primary [#hex], secondary [#hex], accent [#hex]
   Fonts: headings [font name], body [font name]
   ──────────────────────────────────────────────────────────────────
   ```
```

Replace with:
```
#### Phase A: Build Shared Resources (main agent, BEFORE any screen delegation)

1. **Create shared CSS file** (`[product-slug].css`) — write to `./documents/`
2. **Resolve brand assets** (if building for a known company):
   - Identify the CUSTOMER company from `customer_company.name` in the session state
   - Follow the Logo Discovery Protocol in `Prototype Creation Guide.md` Step 1.1
   - **You MUST pass the Logo Gate** (all 5 checks) before using any logo:
     1. HTTP 200
     2. File size 2KB–50KB
     3. Downloaded and LOOKED AT the image
     4. Image shows the CUSTOMER's brand (not a partner/sponsor/competitor)
     5. Stated: "This logo belongs to [Customer] because [reason]"
   - **If the gate fails for all candidates → ask the user for a logo URL.** Use a text placeholder until they provide one. Do NOT guess.
   - Extract the customer's brand colors and typography from THEIR website
   - Record the gate-verified logo URL, brand colors, and fonts — these become part of the shared contract
   - **Subagents must NOT search for logos themselves.** The resolved brand assets are final.
3. **Create Design System reference page** (`DesignSystem_[Product]_[Date].html`) — this is the visual documentation of all design tokens and components. It MUST exist before any screens are built. It links to the `.css` file for its own styling.
4. **Extract Design Token Contract** — read back `[product-slug].css` and extract:
   - **Theme mode:** light if `--surface-bg` is a light color (#F4F7FB, #FFFFFF, etc.), dark if dark (#1a1a2e, #0d1117, etc.)
   - **All CSS variable names with values** from the `:root` block (surfaces, text, brand, borders, semantic)
   - **All component class names** defined in the CSS (`.card`, `.stat-card`, `.page-content`, `.btn-primary`, etc.)
   - Format these into the Design Token Contract block (see Phase B subagent template below)
5. **Create the screen manifest** — a list of EXACT filenames, one per screen:
   ```
   SCREEN MANIFEST (copy verbatim into every subagent prompt):
   ─────────────────────────────────────────────────────────
   CSS file: [product-slug].css
   
   Screens:
   1. Screen_Dashboard_[Product]_[Date].html       (entry point)
   2. Screen_[Name2]_[Product]_[Date].html
   3. Screen_[Name3]_[Product]_[Date].html
   ...
   ─────────────────────────────────────────────────────────
   ```
6. **Create the sidebar nav HTML** — the exact `<nav>` block every screen must use:
   ```html
   <!-- SIDEBAR NAV — paste verbatim, only change which item gets class="active" -->
   <nav class="sidebar-nav">
     <a class="nav-item" href="Screen_Dashboard_[Product]_[Date].html">Dashboard</a>
     <a class="nav-item" href="Screen_[Name2]_[Product]_[Date].html">[Label2]</a>
     <a class="nav-item" href="Screen_[Name3]_[Product]_[Date].html">[Label3]</a>
   </nav>
   ```
7. **Compile the brand assets block** (if applicable) — this gets pasted into every subagent prompt. Use `var()` references for colors, with hex noted only as comments:
   ```
   BRAND ASSETS (use these exactly — do NOT search for alternatives)
   ──────────────────────────────────────────────────────────────────
   Customer: [Company Name]
   Logo URL: [verified URL that returned HTTP 200]
   Logo placement: <img src="[verified URL]" alt="[Company Name] logo" class="header-logo">
   Brand colors: var(--brand-primary), var(--brand-secondary), var(--brand-accent)
   Fonts: var(--font-display), var(--font-body)
   
   Note: Raw hex values are defined in the Design Token Contract above.
   Use var() names in your CSS — never hardcode hex values.
   ──────────────────────────────────────────────────────────────────
   ```
8. **Compile the Design Token Contract block** — formatted for subagent prompts (see template in Phase B)

**HARD GATE — Do NOT proceed to Phase B until ALL of these exist:**
- [ ] `[product-slug].css` created in `./documents/`
- [ ] `DesignSystem_[Product]_[Date].html` created in `./documents/`
- [ ] Design Token Contract block extracted from CSS
- [ ] Screen manifest with exact filenames
- [ ] Sidebar nav HTML template
- [ ] Brand assets block (if known company)
```

- [ ] **Step 2: Verify the edit preserved all surrounding content**

Read lines 248–310 of the modified file. Confirm:
- "### Prototype Agent" header and intro paragraph are intact above
- "#### Phase B: Dispatch Screen Subagents" follows immediately after

- [ ] **Step 3: Commit**

```bash
git add prompts/Orchestrator.md
git commit -m "Orchestrator: reorder Phase A, add Design System + token contract steps, add hard gate"
```

---

### Task 2: Orchestrator.md — Add Design Token Contract to subagent prompt template

**Files:**
- Modify: `prompts/Orchestrator.md:308-372` (Phase B subagent template)

- [ ] **Step 1: Add Design Token Contract block to the subagent prompt example**

In the example subagent prompt (the block between the triple-backtick fences starting with "You are building ONE screen"), find this section:

```
BRAND ASSETS (use these exactly — do NOT search for alternatives)
──────────────────────────────────────────────────────────────────
Customer: NewsBank
Logo: <img src="https://verified-url.example.com/newsbank-logo.png" alt="NewsBank logo" class="header-logo">
Brand colors: primary #1B365D, secondary #4A90D9, accent #F5A623
Fonts: headings "Merriweather", body "Source Sans Pro"

Do NOT search for logos yourself. Use the exact URL above.
Do NOT use competitor logos from the market research phase.

RULES
─────
- Use ONLY filenames from the manifest above for ALL href links in your screen
- Do NOT rename, abbreviate, or invent alternative filenames
- Do NOT modify the sidebar nav (no reordering, renaming, adding, or removing items)
- The ONLY change to the nav is which item has "active" — it must be YOUR screen
- Use the logo URL provided above — do NOT search for a different one

SCREEN REQUIREMENTS
───────────────────
[paste PRD requirements for this specific screen]

DESIGN SYSTEM CLASSES AVAILABLE
───────────────────────────────
[paste list of CSS class names from smartsearch.css]
```

Replace with:

```
BRAND ASSETS (use these exactly — do NOT search for alternatives)
──────────────────────────────────────────────────────────────────
Customer: NewsBank
Logo: <img src="https://verified-url.example.com/newsbank-logo.png" alt="NewsBank logo" class="header-logo">
Brand colors: var(--brand-primary), var(--brand-secondary), var(--brand-accent)
Fonts: var(--font-display), var(--font-body)

Do NOT search for logos yourself. Use the exact URL above.
Do NOT use competitor logos from the market research phase.

DESIGN TOKEN CONTRACT (use these — do NOT hardcode colors)
──────────────────────────────────────────────────────────
Theme: [THEME_MODE — e.g., LIGHT or DARK]

CSS Variables (use var() syntax, never raw hex):
  Surfaces:    [e.g., var(--surface-bg): #F4F7FB  |  var(--surface-card): #FFFFFF]
  Text:        [e.g., var(--text-primary): #1B2A4A  |  var(--text-secondary): #64748B]
  Brand:       [e.g., var(--brand-primary): #1B365D  |  var(--brand-accent): #F5A623]
  Borders:     [e.g., var(--border-light): #E2E8F0]
  Semantic:    [e.g., var(--color-success): #10B981  |  var(--color-error): #EF4444]

Component Classes (use these instead of writing custom styles):
  [e.g., .card, .card-title, .card-body, .stat-card, .stat-value, .stat-label,
   .page-content, .page-header, .btn-primary, .btn-secondary, .btn-ghost,
   .data-table, .table-header, .table-row, .sidebar-nav, .nav-item]

Values above are examples. Paste the ACTUAL variables and classes extracted from [product-slug].css.

RULES
─────
- Use ONLY filenames from the manifest above for ALL href links in your screen
- Do NOT rename, abbreviate, or invent alternative filenames
- Do NOT modify the sidebar nav (no reordering, renaming, adding, or removing items)
- The ONLY change to the nav is which item has "active" — it must be YOUR screen
- Use the logo URL provided above — do NOT search for a different one
- Use var(--variable-name) for ALL colors — never hardcode hex values
- Use the component classes from the Design Token Contract — do NOT recreate card/button/table styles in <style>
- Screen-specific <style> overrides must be < 50 lines and must use var() for any colors
- This is a [THEME_MODE] mode app — all surfaces and text must match this theme

SCREEN REQUIREMENTS
───────────────────
[paste PRD requirements for this specific screen]
```

- [ ] **Step 2: Update the "Key points" list below the template**

Find the list starting with "**Key points:**" (around line 374). Add two new bullet points at the end of the list, before the "Repeat this template" paragraph:

```
- The Design Token Contract gives the subagent every CSS variable name, value, and component class — it must use var() references, never hardcoded hex
- The theme mode (LIGHT/DARK) is explicitly stated so the subagent cannot independently choose a conflicting aesthetic
```

- [ ] **Step 3: Update the "Everything else stays identical" paragraph**

Find: `Everything else stays identical across all subagent prompts: CSS link, manifest, nav HTML, brand assets, rules.`

Replace with: `Everything else stays identical across all subagent prompts: CSS link, manifest, nav HTML, brand assets, Design Token Contract, rules.`

- [ ] **Step 4: Commit**

```bash
git add prompts/Orchestrator.md
git commit -m "Orchestrator: add Design Token Contract block and var() rules to subagent template"
```

---

### Task 3: Orchestrator.md — Add visual consistency check to Phase C

**Files:**
- Modify: `prompts/Orchestrator.md:390-396` (Phase C section)

- [ ] **Step 1: Add check #5 to Phase C**

Find the Phase C section:

```
#### Phase C: Post-Build Validation

After all screens are built, BEFORE presenting to user:
1. Verify every manifest filename has a corresponding file in `./documents/`
2. Extract all `href` values from all screen files — every one must match a manifest entry
3. Verify all screens have identical sidebar nav (only `active` class differs)
4. Fix any mismatches before proceeding
```

Replace with:

```
#### Phase C: Post-Build Validation

After all screens are built, BEFORE presenting to user:
1. Verify every manifest filename has a corresponding file in `./documents/`
2. Extract all `href` values from all screen files — every one must match a manifest entry
3. Verify all screens have identical sidebar nav (only `active` class differs)
4. Fix any mismatches before proceeding
5. **Visual consistency check:** Scan `<style>` blocks in all screen files for hardcoded hex color values. For each screen:
   - Count `var(--` references vs hardcoded `#` hex colors in the `<style>` block
   - Hardcoded hex count must be LESS than var() count — flag any screen that fails
   - Check for theme violations: dark colors (#1a1a2e, #0d0d0d, #111) in a light-mode app, or light colors (#fff, #f4f7fb) in a dark-mode app
   - Replace hardcoded values with their `var()` equivalents from `[product-slug].css`
   - If no equivalent variable exists, add it to the shared CSS first
```

- [ ] **Step 2: Commit**

```bash
git add prompts/Orchestrator.md
git commit -m "Orchestrator: add visual consistency check to Phase C post-build validation"
```

---

### Task 4: Prototype Creation Guide.md — Add Steps 2 and 2.5

**Files:**
- Modify: `prompts/Prototype Creation Guide.md:152-153` (between Step 1 and Step 1.1)

- [ ] **Step 1: Elevate Design System creation to Step 2 and add Step 2.5**

The current line 152 reads:
```
**Separately, create `DesignSystem_[Product]_[Date].html`** — this is a visual reference page that documents colors, components, and typography for human review. It links to the `.css` file for its own styling.
```

Replace that single line with a full section:

```
### Step 2: Create Design System Reference Page (REQUIRED — BEFORE Any Screens)

**Create `DesignSystem_[Product]_[Date].html` BEFORE building any screen files.** This is the visual reference page that documents colors, components, and typography for human review. It links to the `.css` file for its own styling.

The Design System is a **governing specification**, not post-hoc documentation. It must exist before screen files so that:
- All screen builders (including parallel subagents) reference the same visual contract
- Theme mode (light or dark) is explicitly decided and documented
- Component classes and CSS variables are defined once and used everywhere

**The Design System page must include:**
- Theme declaration (LIGHT or DARK mode)
- Color palette with all CSS variable names and values
- Typography scale with font pairings
- Component library (buttons, cards, forms, navigation) with class names
- Spacing and layout system
- Animation tokens

### Step 2.5: Extract Design Token Contract (REQUIRED — BEFORE Any Screens)

After creating the shared CSS and Design System page, extract a **Design Token Contract** from `[product-slug].css`. This contract is pasted into every subagent prompt to prevent theme divergence.

**How to extract:**
1. Read `[product-slug].css` and collect all `:root` CSS variable names with their values
2. Collect all class names defined in the CSS (`.card`, `.stat-card`, `.page-content`, `.btn-primary`, etc.)
3. Determine theme mode: light if `--surface-bg` is a light color, dark if dark
4. Format into the Design Token Contract template:

```
DESIGN TOKEN CONTRACT (use these — do NOT hardcode colors)
──────────────────────────────────────────────────────────
Theme: [THEME_MODE]

CSS Variables (use var() syntax, never raw hex):
  Surfaces:    [list var names and values]
  Text:        [list var names and values]
  Brand:       [list var names and values]
  Borders:     [list var names and values]
  Semantic:    [list var names and values]

Component Classes (use these instead of writing custom styles):
  [list all class names from the CSS]

RULES:
- Use var(--variable-name) for ALL colors — never hardcode hex values
- Use the component classes above — do NOT recreate card/button/table styles in <style>
- Screen-specific <style> overrides must be < 50 lines and use var() for colors
- This is a [THEME_MODE] mode app — all surfaces and text must match this theme
```

This contract block is included in every subagent prompt alongside the Screen Manifest and Brand Assets blocks.
```

- [ ] **Step 2: Verify surrounding content**

Read lines 145–165 of the modified file. Confirm:
- Step 1 content above (CSS rules, extension requirements) is intact
- Step 1.1 (Research Customer Brand) follows after the new sections

- [ ] **Step 3: Commit**

```bash
git add "prompts/Prototype Creation Guide.md"
git commit -m "Prototype Guide: add Steps 2 and 2.5 for Design System and token contract extraction"
```

---

### Task 5: Prototype Creation Guide.md — Update Step 3.5.3 and add Step 8.5 check #7

**Files:**
- Modify: `prompts/Prototype Creation Guide.md:496-509` (Step 3.5.3)
- Modify: `prompts/Prototype Creation Guide.md:1029` (after Step 8.5 check #6)

- [ ] **Step 1: Add item #6 to the Step 3.5.3 "must include" list**

Find the list in Step 3.5.3:

```
Each screen subagent's prompt MUST include ALL of the following — no exceptions:

1. **The CSS filename** — `<link rel="stylesheet" href="[product-slug].css">`
2. **The complete screen manifest** — all exact filenames (paste the full list)
3. **The sidebar nav HTML template** — paste the full `<nav>` block verbatim
4. **Which nav item is active** — specify which `<a>` tag gets `class="nav-item active"`
5. **The design system class names** available for use
```

Replace with:

```
Each screen subagent's prompt MUST include ALL of the following — no exceptions:

1. **The CSS filename** — `<link rel="stylesheet" href="[product-slug].css">`
2. **The complete screen manifest** — all exact filenames (paste the full list)
3. **The sidebar nav HTML template** — paste the full `<nav>` block verbatim
4. **Which nav item is active** — specify which `<a>` tag gets `class="nav-item active"`
5. **The design system class names** available for use
6. **The Design Token Contract** — all CSS variable names with values, component class inventory, and explicit theme mode (LIGHT/DARK). See Step 2.5 for the contract template. Subagents must use `var()` references for all colors — never hardcoded hex values.
```

- [ ] **Step 2: Update the explicit instruction quote**

Find:

```
**Explicit instruction to include in every subagent prompt:**
> "Use ONLY filenames from the manifest for all href links. Do NOT rename, abbreviate, or invent alternative filenames. Paste the sidebar nav HTML VERBATIM — only add 'active' to your screen's nav item."
```

Replace with:

```
**Explicit instruction to include in every subagent prompt:**
> "Use ONLY filenames from the manifest for all href links. Do NOT rename, abbreviate, or invent alternative filenames. Paste the sidebar nav HTML VERBATIM — only add 'active' to your screen's nav item. Use var(--variable-name) for ALL colors — never hardcode hex values. Use component classes from the Design Token Contract instead of writing custom styles."
```

- [ ] **Step 3: Add Step 8.5 check #7 — Visual Consistency Check**

Find the end of Step 8.5 check #6 (the "Quick Smoke Test" section). After the line that reads:

```
**This is the authoritative quality gate for prototypes.** Other quality checklists in this file and in `Shared Standards.md` cover design and functional quality; this step covers structural integrity.
```

Add the following new section immediately after:

```
#### 7. Visual Consistency Check (Theme Coherence)

Scan all `Screen_*.html` files for hardcoded colors in `<style>` blocks that conflict with the shared CSS theme:

**a. Extract theme mode from `[product-slug].css`:**
- If `--surface-bg` is a light color (#F4F7FB, #FFFFFF, etc.) → app is **LIGHT** mode
- If `--surface-bg` is a dark color (#1a1a2e, #0d1117, etc.) → app is **DARK** mode

**b. Grep each screen's `<style>` block for hardcoded hex values:**
```bash
grep -oE '#[0-9a-fA-F]{3,8}' documents/Screen_*.html
```

**c. Flag violations:**
- LIGHT mode app with dark backgrounds (#1a1a2e, #0d0d0d, #111, etc.) in cards/content areas
- DARK mode app with light backgrounds (#fff, #f4f7fb, etc.) in cards/content areas
- Any hardcoded color that has a CSS variable equivalent in the shared CSS

**d. Count var() vs hardcoded hex references per screen:**
```bash
# var() references (should be high):
grep -c 'var(--' documents/Screen_[Name].html

# Hardcoded hex in <style> blocks only (should be low):
# Extract <style> block, then count hex values
```
- Flag any screen where hardcoded hex count > var() count

**e. Fix violations:** Replace hardcoded values with their `var()` equivalents from the shared CSS. If no equivalent variable exists, add the variable to `[product-slug].css` first, then reference it.
```

- [ ] **Step 4: Commit**

```bash
git add "prompts/Prototype Creation Guide.md"
git commit -m "Prototype Guide: add token contract to Step 3.5.3, add visual consistency check to Step 8.5"
```

---

### Task 6: CLAUDE.md — Update Prototype Structure with strict build order

**Files:**
- Modify: `CLAUDE.md:56-68` (Prototype Structure section)

- [ ] **Step 1: Replace the Prototype Structure section**

Find the current section:

```
## Prototype Structure

**Create MODULAR files, not a single monolithic HTML:**
- `[product-slug].css` - Shared CSS file (create FIRST, `.css` extension required)
- `DesignSystem_[Product]_[YYYY-MM-DD].html` - Visual reference page (documents colors, components, typography)
- `Screen_[Name]_[Product]_[YYYY-MM-DD].html` - One file per screen (links to `.css` via `<link rel="stylesheet">`)
- `ScreenIndex_[Product]_[YYYY-MM-DD].html` - Navigation hub (use template at `prompts/ScreenIndex_Template.html`)

**CSS Architecture:**
- Shared styles MUST use `.css` extension — browsers reject `.html` files loaded via `<link rel="stylesheet">` (MIME type mismatch)
- Screen files link to shared CSS: `<link rel="stylesheet" href="[product-slug].css">`
- Screen-specific overrides allowed in `<style>` blocks (< 50 lines)
- ClickablePrototype is exempt (single-file, all CSS inline is fine)
```

Replace with:

```
## Prototype Structure

**Create MODULAR files, not a single monolithic HTML.**

**Build order (STRICT — each step depends on the previous):**
1. `[product-slug].css` — Shared CSS file (create FIRST, `.css` extension required)
2. `DesignSystem_[Product]_[YYYY-MM-DD].html` — Visual reference page (BEFORE any screens)
3. Design Token Contract — extracted from CSS for subagent prompts (theme mode, var names, class inventory)
4. Screen manifest + sidebar nav template — exact filenames, verbatim nav HTML
5. `Screen_[Name]_[Product]_[YYYY-MM-DD].html` — One file per screen (links to `.css`, uses token contract)
6. `ScreenIndex_[Product]_[YYYY-MM-DD].html` — Navigation hub (LAST, use template at `prompts/ScreenIndex_Template.html`)

**CSS Architecture:**
- Shared styles MUST use `.css` extension — browsers reject `.html` files loaded via `<link rel="stylesheet">` (MIME type mismatch)
- Screen files link to shared CSS: `<link rel="stylesheet" href="[product-slug].css">`
- Screen-specific overrides allowed in `<style>` blocks (< 50 lines), must use `var()` for colors
- ClickablePrototype is exempt (single-file, all CSS inline is fine)
```

- [ ] **Step 2: Verify surrounding content**

Read lines 55–72 of the modified file. Confirm the ScreenIndex placeholders section follows immediately after.

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "CLAUDE.md: update prototype build order to enforce Design System before screens"
```

---

### Task 7: Kiro prototype-guide.md — Reorder output files, add hard gate, add validation

**Files:**
- Modify: `.kiro/steering/prototype-guide.md:288-407` (Output Files and Screen Manifest sections)
- Modify: `.kiro/steering/prototype-guide.md:428-468` (Post-Build Validation section)

- [ ] **Step 1: Update the Output Files section header and reorder**

Find:

```
### 0. Shared CSS File (create FIRST — REQUIRED)
```

Add a build order note immediately before this line:

```
**Build order (STRICT — each step depends on the previous):**
1. Shared CSS file (`[product-slug].css`)
2. Design System reference page (`DesignSystem_[Product]_[Date].html`) — BEFORE any screens
3. Design Token Contract — extracted from CSS (theme mode, var names, class inventory)
4. Screen manifest + sidebar nav template
5. Individual screen files (`Screen_[Name]_[Product]_[Date].html`)
6. Screen Index (`ScreenIndex_[Product]_[Date].html`) — LAST

```

- [ ] **Step 2: Add Design Token Contract to Screen Manifest pass-through**

Find the line in the Screen Manifest section:

```
**Step 3: Pass to every screen builder:** Each screen's prompt MUST include the CSS filename, the complete manifest, the sidebar nav template, which nav item is active, and available CSS class names.
```

Replace with:

```
**Step 3: Pass to every screen builder:** Each screen's prompt MUST include the CSS filename, the complete manifest, the sidebar nav template, which nav item is active, available CSS class names, and the **Design Token Contract** (all CSS variable names with values, component class inventory, and explicit theme mode — LIGHT or DARK). Subagents must use `var()` for all colors — never hardcoded hex.
```

- [ ] **Step 3: Add hard gate before the Screen Manifest section**

Find:

```
### Screen Manifest (REQUIRED — Create Before Building Screens)
```

Add immediately before this line:

```
### HARD GATE — Before Building Any Screens

Do NOT create any `Screen_*.html` files until ALL of these exist in `./documents/`:
- [ ] `[product-slug].css` — shared stylesheet
- [ ] `DesignSystem_[Product]_[Date].html` — visual reference page
- [ ] Design Token Contract block — extracted from CSS (theme mode, CSS variables, component classes)
- [ ] Screen manifest with exact filenames
- [ ] Sidebar nav HTML template
- [ ] Brand assets block (if building for a known company)

```

- [ ] **Step 4: Add visual consistency check to Post-Build Validation**

Find the end of the "### 4. File Size Check" section (after the file size limits). Add a new section:

```
### 5. Visual Consistency Check (Theme Coherence)

Scan all `Screen_*.html` files for hardcoded colors that conflict with the shared CSS theme:

1. **Determine theme mode** from `[product-slug].css`: light if `--surface-bg` is light, dark if dark
2. **Grep `<style>` blocks** for hardcoded hex values: `grep -oE '#[0-9a-fA-F]{3,8}' Screen_*.html`
3. **Flag violations:**
   - Dark colors (#1a1a2e, #0d0d0d, #111) in a light-mode app's cards/content
   - Light colors (#fff, #f4f7fb) in a dark-mode app's cards/content
   - Any hardcoded color with a CSS variable equivalent in the shared CSS
4. **Count per screen:** var(--) references vs hardcoded hex — flag any screen where hardcoded > var()
5. **Fix:** Replace hardcoded values with `var()` equivalents. Add missing variables to CSS first if needed.

```

- [ ] **Step 5: Renumber existing checks 5 and 6**

The existing "### 5. Logo & Brand Verification" becomes "### 6. Logo & Brand Verification" and "### 6. Quick Smoke Test" becomes "### 7. Quick Smoke Test".

- [ ] **Step 6: Commit**

```bash
git add .kiro/steering/prototype-guide.md
git commit -m "Kiro prototype-guide: add build order, hard gate, token contract, visual consistency check"
```

---

### Task 8: Kiro specialist-prototype.md — Add CSS Variable Usage mandate

**Files:**
- Modify: `.kiro/steering/specialist-prototype.md:56-70` (after the Design Philosophy section)

- [ ] **Step 1: Add CSS Variable Usage section**

Find the "## Design Philosophy: NO AI SLOP" section. After its "**ALWAYS use:**" list (ending with "- Modular file structure"), add a new section:

```

## CSS Variable Usage (MANDATORY)

When writing screen-specific `<style>` overrides:
- **ALL colors** must use `var(--variable-name)` from the shared CSS — never hardcode hex values
- **Use component classes** from the shared CSS (`.card`, `.stat-card`, `.page-content`, etc.) instead of writing custom card/button/table styles
- Screen-specific `<style>` blocks must be < 50 lines
- **Match the app's theme mode** (LIGHT or DARK) as declared in the shared CSS — do not independently choose a different theme
- If the Design Token Contract was provided in your prompt, follow it exactly

**Why:** When screens are built in parallel, each subagent independently choosing colors creates a visual mashup — dark cards on light backgrounds, inconsistent text colors. Using shared CSS variables ensures every screen belongs to the same app.

```

- [ ] **Step 2: Commit**

```bash
git add .kiro/steering/specialist-prototype.md
git commit -m "Kiro specialist-prototype: add mandatory CSS variable usage section"
```

---

### Task 9: Kiro specialist-design-system.md — Add build order note

**Files:**
- Modify: `.kiro/steering/specialist-design-system.md:10-12` (after the intro paragraph)

- [ ] **Step 1: Add build order note**

Find:

```
**CRITICAL: You create TWO files:**
1. **`[product-slug].css`** — The shared stylesheet that all screens link to via `<link rel="stylesheet">`. Must use `.css` extension (browsers reject `.html` loaded as stylesheets due to MIME type mismatch). Use a stable filename without date suffix.
2. **`DesignSystem_[Product]_[Date].html`** — A visual reference page that documents your design tokens and components for human review. This page links to the `.css` file for its own styling.
```

Replace with:

```
**CRITICAL: You create TWO files, and they MUST exist BEFORE any Screen_*.html files are built:**
1. **`[product-slug].css`** — The shared stylesheet that all screens link to via `<link rel="stylesheet">`. Must use `.css` extension (browsers reject `.html` loaded as stylesheets due to MIME type mismatch). Use a stable filename without date suffix.
2. **`DesignSystem_[Product]_[Date].html`** — A visual reference page that documents your design tokens and components for human review. This page links to the `.css` file for its own styling.

**Build order:** The Design System is a **governing specification**, not post-hoc documentation. All screen builders (including parallel subagents) reference it as the single source of truth for theme mode, color variables, and component classes. If screens are built before this file exists, they will make independent aesthetic decisions that conflict with each other.
```

- [ ] **Step 2: Commit**

```bash
git add .kiro/steering/specialist-design-system.md
git commit -m "Kiro specialist-design-system: add build order mandate"
```

---

### Task 10: Kiro hooks.json — Update phase transition and enhance Screen Validator

**Files:**
- Modify: `.kiro/hooks.json` (two hooks to update)

- [ ] **Step 1: Update the "Phase Transition: PRD → Prototype" hook prompt**

Find:

```json
"prompt": "PRD complete! Ask the user:\n\n'Ready for the next phase? Options:\n1. Review the PRD\n2. Proceed to Prototype (create DesignSystem first, then Screen files)\n3. Make changes first'\n\nWait for their response."
```

Replace with:

```json
"prompt": "PRD complete! Ask the user:\n\n'Ready for the next phase? Options:\n1. Review the PRD\n2. Proceed to Prototype (strict build order: shared CSS → Design System → Design Token Contract → screen manifest → screens)\n3. Make changes first'\n\nWait for their response."
```

- [ ] **Step 2: Enhance the Screen Validator hook prompt**

Find the Screen Validator's prompt string. Locate this line within it:

```
- [ ] Uses CSS variables (not hardcoded colors)
```

Replace with:

```
- [ ] Uses CSS variables (not hardcoded colors):\n      * Count var(--) refs vs hardcoded # hex in <style> block\n      * Hardcoded hex count must be LESS than var() count\n      * No dark backgrounds (#1a1a2e, #0d0d0d, #111) in a light-mode app\n      * No light backgrounds (#fff, #f4f7fb, #fafafa) in a dark-mode app
```

- [ ] **Step 3: Verify JSON is valid**

```bash
python3 -c "import json; json.load(open('.kiro/hooks.json')); print('Valid JSON')"
```

Expected: `Valid JSON`

- [ ] **Step 4: Commit**

```bash
git add .kiro/hooks.json
git commit -m "Kiro hooks: update phase transition prompt, enhance Screen Validator with theme checks"
```

---

### Task 11: Final verification

- [ ] **Step 1: Grep for all "Design Token Contract" references across all modified files**

```bash
grep -rn "Design Token Contract" prompts/Orchestrator.md "prompts/Prototype Creation Guide.md" CLAUDE.md .kiro/steering/prototype-guide.md .kiro/steering/specialist-prototype.md .kiro/steering/specialist-design-system.md
```

Expected: References in all files except `hooks.json` (which uses the expanded checklist instead).

- [ ] **Step 2: Grep for "HARD GATE" in both Claude Code and Kiro steering files**

```bash
grep -rn "HARD GATE" prompts/Orchestrator.md .kiro/steering/prototype-guide.md
```

Expected: One match in each file.

- [ ] **Step 3: Grep for "var(--" in the Orchestrator subagent template to confirm hex values were replaced**

```bash
grep -n "var(--" prompts/Orchestrator.md
```

Expected: Multiple matches in the brand assets block and Design Token Contract template.

- [ ] **Step 4: Verify no raw hex values remain in the Brand Assets block**

```bash
grep -A5 "BRAND ASSETS" prompts/Orchestrator.md | grep -E "primary #|secondary #|accent #"
```

Expected: No matches (hex values should be replaced with var() references).

- [ ] **Step 5: Validate hooks.json is still valid JSON**

```bash
python3 -c "import json; json.load(open('.kiro/hooks.json')); print('Valid JSON')"
```

Expected: `Valid JSON`

- [ ] **Step 6: Commit any final fixes if needed, then do a final status check**

```bash
git status
git log --oneline -10
```

Expected: Clean working tree, 8-10 new commits on the branch.
