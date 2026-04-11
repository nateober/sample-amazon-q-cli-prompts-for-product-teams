# Design Token Contract & Visual Consistency Validation

**Date:** 2026-04-11
**Status:** Approved
**Trigger:** CloudReady Prototype RCA — light/dark mode mashup across 4 of 8 screens

## Problem

Two defects recur when prototype screens are built in parallel by subagents:

1. **Theme divergence** — Subagents independently choose light or dark card backgrounds because they lack Design System context. Each screen looks intentional in isolation; the mashup only manifests when screens are viewed together.
2. **Design System created after screens** — The Design System reference page is treated as documentation to generate, not a specification to follow. By the time it exists, all screens are already built.

Root cause: The subagent coordination contract covers filenames and navigation (Screen Manifest) but has no equivalent for design tokens. Subagents receive the CSS filename to link, and raw hex brand colors, but not the CSS variable names, component classes, or theme direction from the shared CSS.

## Solution

Three layers, applied to both Claude Code (`prompts/`) and Kiro (`.kiro/steering/`) steering files:

### Layer 1: Design Token Contract (prevention at source)

Mirror the Screen Manifest pattern. Before dispatching subagents, the main agent extracts a Design Token Contract from the shared CSS file and includes it in every subagent prompt.

**Contract template (steering files define the shape; values filled at build time):**

```
DESIGN TOKEN CONTRACT (use these — do NOT hardcode colors)
──────────────────────────────────────────────────────────
Theme: [THEME_MODE]

CSS Variables (use var() syntax, never raw hex):
  Surfaces:    [SURFACE_VARIABLES]
  Text:        [TEXT_VARIABLES]
  Brand:       [BRAND_VARIABLES]
  Borders:     [BORDER_VARIABLES]
  Semantic:    [SEMANTIC_VARIABLES]

Component Classes (use these instead of writing custom styles):
  [CLASS_INVENTORY]

RULES:
- Use var(--variable-name) for ALL colors — never hardcode hex values
- Use the component classes above — do NOT recreate card/button/table styles in <style>
- Screen-specific <style> overrides must be < 50 lines and use var() for colors
- This is a [THEME_MODE] mode app — all surfaces and text must match this theme
```

**How it gets filled at build time:**
1. Main agent creates `[product-slug].css`
2. Main agent reads the CSS back and extracts:
   - All `:root` CSS variable names and their values
   - All class names defined in the CSS (`.card`, `.stat-card`, `.page-content`, etc.)
   - Theme mode: light if `--surface-bg` is a light color, dark if dark
3. Main agent formats these into the contract template
4. Contract is pasted into every subagent prompt alongside the Screen Manifest and Brand Assets

**Subagent prompt rules (added to RULES section):**
- Use var(--variable-name) for ALL colors — never hardcode hex values
- Use the component classes from the contract — do NOT recreate card/button styles in `<style>`
- Screen-specific `<style>` overrides must be < 50 lines and must use var() for any colors
- This is a [THEME_MODE] mode app — all surfaces and text must match this theme

**Brand Assets block change:** Replace raw hex values with var() references. Instead of `primary #1B365D`, use `var(--brand-primary)` with hex noted only as a comment for reference.

### Layer 2: Build Order Enforcement (prevention via sequence)

Enforce that the Design System reference page and Design Token Contract exist BEFORE any screens are built.

**New Phase A order in Orchestrator:**
1. Create shared CSS file (`[product-slug].css`)
2. Resolve brand assets (logo, colors, fonts)
3. **Create Design System reference page** (`DesignSystem_[Product]_[Date].html`)
4. **Extract Design Token Contract** (read CSS, produce contract block)
5. Create screen manifest (exact filenames)
6. Create sidebar nav HTML template
7. Compile brand assets block (using var() names, not raw hex)
8. Compile Design Token Contract block (formatted for subagent prompts)

**Hard gate before Phase B (screen dispatch):**
```
HARD GATE — Do NOT dispatch any screen subagents until ALL of these exist:
[ ] [product-slug].css
[ ] DesignSystem_[Product]_[Date].html
[ ] Design Token Contract block (extracted from CSS)
[ ] Screen manifest with exact filenames
[ ] Sidebar nav HTML template
[ ] Brand assets block (if known company)
```

**Prototype Creation Guide step renumbering:**
- Step 1: Create Shared CSS File (unchanged)
- Step 1.1: Research Customer Brand (unchanged)
- Step 1.5: Establish Aesthetic Direction (unchanged)
- **Step 2: Create Design System Reference Page** (elevated from side note to numbered step)
- **Step 2.5: Extract Design Token Contract** (new step)
- Step 3: Information Architecture (unchanged)
- Step 3.5: Create Screen Manifest (unchanged, but Step 3.5.3 now includes token contract as item #6)
- Step 4: Build Individual Screens (unchanged)

**CLAUDE.md build order update:**
```
Build order (STRICT — each step depends on the previous):
1. [product-slug].css — Shared CSS file
2. DesignSystem_[Product]_[Date].html — Visual reference (BEFORE any screens)
3. Design Token Contract — extracted from CSS for subagent prompts
4. Screen manifest + sidebar nav template
5. Screen_[Name]_[Product]_[Date].html — Individual screens
6. ScreenIndex_[Product]_[Date].html — Navigation hub (LAST)
```

### Layer 3: Post-Build Visual Consistency Validation (detection)

Add a grep-based check to catch hardcoded colors that bypass the shared CSS.

**New Step 8.5 check #7 in Prototype Creation Guide:**

```
7. Visual Consistency Check (Theme Coherence)

a. Extract theme mode from [product-slug].css:
   - If --surface-bg is light (#F4F7FB, #FFFFFF, etc.) -> LIGHT mode
   - If --surface-bg is dark (#1a1a2e, #0d1117, etc.) -> DARK mode

b. Grep each screen's <style> block for hardcoded hex values:
   grep -oE '#[0-9a-fA-F]{3,8}' Screen_*.html

c. Flag violations:
   - LIGHT mode app with dark backgrounds in cards/content
   - DARK mode app with light backgrounds in cards/content
   - Any hardcoded color that has a CSS variable equivalent

d. Count var() vs hardcoded hex references per screen:
   - var() refs:     grep -c 'var(--' Screen_[Name].html
   - Hardcoded hex:  grep -c '#[0-9a-fA-F]' (in <style> blocks only)
   - Flag any screen where hardcoded hex count > var() count

e. Fix violations: replace hardcoded values with var() equivalents.
   If no equivalent exists, add the variable to the shared CSS first.
```

**Orchestrator Phase C addition (check #5):**
Scan `<style>` blocks in all screens for hardcoded hex values. Replace with var() equivalents. Flag any screen where hardcoded colors outnumber CSS variable references.

**Kiro hooks.json — enhance Screen Validator:**
Expand the existing "Uses CSS variables (not hardcoded colors)" checkbox to:
- Count var(--) references vs hardcoded # colors in `<style>` block
- Hardcoded hex count must be LESS than var() count
- No dark-themed colors (#1a1a2e, #0d0d0d) in a light-mode app
- No light-themed colors (#fff, #f4f7fb) in a dark-mode app

**Kiro specialist-prototype.md — add CSS Variable Usage rule:**
All colors in `<style>` blocks must use var(--variable-name). Never hardcode hex values for colors with CSS variable equivalents. Screen must match the app's declared theme mode.

## File Change Map

| File | Layer 1 (Token Contract) | Layer 2 (Build Order) | Layer 3 (Validation) |
|------|--------------------------|----------------------|---------------------|
| `prompts/Orchestrator.md` | New contract block in subagent template, var() in brand assets | Reorder Phase A, add hard gate | Add Phase C check #5 |
| `prompts/Prototype Creation Guide.md` | Update Step 3.5.3 item #6 | New Steps 2 and 2.5 | New Step 8.5 check #7 |
| `CLAUDE.md` | Brief mention of token contract | Update build order list | -- |
| `.kiro/steering/prototype-guide.md` | Add contract to screen builder pass-through | Reorder output files, add hard gate | Add validation check |
| `.kiro/steering/specialist-prototype.md` | -- | -- | Add CSS variable mandate section |
| `.kiro/steering/specialist-design-system.md` | -- | Add "create BEFORE screens" note | -- |
| `.kiro/hooks.json` | -- | Update phase transition prompt | Enhance Screen Validator |

## Design Decisions

**Why mirror the Screen Manifest pattern?** It already proved effective at eliminating broken cross-links (the previous #1 defect). Same coordination problem, same solution shape.

**Why prohibit raw hex in subagent prompts?** When a subagent has both `var(--brand-primary)` and `#0F48B8` available, it may choose whichever produces the most visually striking result in isolation. Removing the option eliminates the temptation.

**Why both prevention and detection?** The Design Token Contract prevents the defect at source. The validation gate catches anything that slips through. This belt-and-suspenders approach is the same pattern used for navigation (manifest prevents broken links, post-build link audit catches stragglers).

**Why not define CSS in steering files?** Steering files define the contract template (shape and placeholders), not actual CSS values. The main agent fills in real values at build time by reading the shared CSS file. This keeps steering files product-agnostic.
