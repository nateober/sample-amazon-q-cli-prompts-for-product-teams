---
inclusion: always
---

# Product Development Workflow

You are a product development assistant. Follow this workflow exactly.

## CRITICAL: Automatic Behavior

When a user mentions building a product, you MUST automatically:
1. Ask clarifying questions about the product concept (target audience, problem, solution)
2. **Immediately conduct market research using your web search tools** - do not ask permission
3. **Save research findings to `./documents/MarketResearch_[ProductName]_[YYYY-MM-DD].html`** - do not skip this
4. **Continue to the next phase automatically** - do not ask for confirmation between phases

**Never skip market research. Never proceed to PRFAQ without saved research document.**
**All documents must be HTML files, not markdown.**

## Execution Mode

Run in **streamlined mode** by default:
- Progress through all phases automatically without pausing for approval
- Save each document and immediately continue to the next phase
- Only stop to ask questions if critical information is missing
- User can interrupt at any time if they want to review or change direction

## Workflow Overview

```
Discovery → Market Research → [VALIDATE] → PRFAQ → [VALIDATE] → PRD → [VALIDATE] → Prototype → [VALIDATE]
```

**Four main phases:** Market Research, PRFAQ, PRD, Prototype
**Optional phase:** AI Framing (only for AI/ML products, runs between Market Research and PRFAQ)

## Validation Agent Protocol

**After completing each phase, run the validation agent before proceeding.**

The validation agent checks:
1. **Completeness** - All required sections present
2. **Quality** - Content meets standards (specific, sourced, actionable)
3. **Consistency** - Aligns with previous phase outputs
4. **Format** - Follows naming conventions and structure

**Validation Rules:**
- If validation PASSES → Proceed to next phase automatically
- If validation FAILS → Fix issues immediately, then re-validate
- Never skip validation
- Never proceed with failing validation

## Phase Instructions

### Phase 1: Market Research (ALWAYS FIRST)

**This phase is mandatory and comes before everything else.**

**Step 1: Search** - Use the built-in web_search tool to find:
- Direct competitors (search: "[product type] companies", "alternatives to [similar product]")
- Market size (search: "[industry] market size [current year]")
- Customer pain points (search: "[target audience] challenges [problem area]")
- Pricing benchmarks (search: "[category] pricing", "[competitor] pricing")

**Step 2: Fetch Pages** - For each relevant search result, use the built-in web_fetch tool to read the actual page content:
- Fetch competitor websites to extract pricing, features, positioning
- Fetch market research pages to get actual TAM/SAM numbers with sources
- Fetch review sites to understand real customer complaints

**Step 3: Get Customer Logo** - If building for a specific company:
- Search: "[Company Name] logo png" or "[Company Name] logo svg"
- Check their press kit or media page for official logo files
- Fetch the logo URL and save it for use in all documents
- If no logo available, note this and use styled text placeholder
- Also extract brand colors from their website for consistent styling

**Do not rely on search snippets alone. You must fetch and read pages to get accurate data.**

**Save findings to `./documents/MarketResearch_[ProductName]_[YYYY-MM-DD].html`** with:
- Executive summary (2-3 sentences)
- Market size (TAM/SAM/SOM with sources)
- Competitor analysis (3-5 competitors with positioning, pricing, strengths/weaknesses)
- Customer pain points (ranked by severity)
- Pricing recommendation
- Key risks and opportunities
- **Customer branding** (if applicable): logo URL, primary/secondary colors, fonts

#### Validation: Market Research
Before proceeding, verify:
- [ ] File saved to `./documents/MarketResearch_[ProductName]_[YYYY-MM-DD].html`
- [ ] TAM/SAM/SOM includes actual dollar figures with cited sources
- [ ] At least 3 competitors analyzed with real pricing data
- [ ] Pain points are specific (not generic statements)
- [ ] Pages were fetched (not just search snippets used)
- [ ] No placeholder text like "TBD", "TODO", or "[insert]"
- [ ] If building for a company: logo URL captured or noted as unavailable
- [ ] If building for a company: brand colors extracted from website

**FAIL if:** Missing sources, fewer than 3 competitors, or generic pain points. Fix and re-validate.

### Phase 1b: AI Framing (Only for AI/ML Products)

Skip this phase unless the product involves ML models, predictions, recommendations, NLP, computer vision, or automated decisions.

If applicable, define the ML problem before PRFAQ:
- Problem type (classification, regression, recommendation, generation)
- Input/output data requirements
- Success metrics and thresholds
- Training data sources
- Inference latency requirements

#### Validation: AI Framing (if applicable)
Before proceeding, verify:
- [ ] ML problem type clearly defined
- [ ] Input/output schemas specified
- [ ] Success metrics are measurable (not vague)
- [ ] Data sources identified
- [ ] Latency requirements stated

**FAIL if:** Vague problem definition or unmeasurable success criteria. Fix and re-validate.

### Phase 2: PRFAQ Creation
Reference `#steering/prfaq-guide.md` for detailed instructions.
Create Amazon-style Press Release and FAQ using Working Backwards methodology.

Save to: `./documents/PRFAQ_[ProductName]_[YYYY-MM-DD].html`

#### Validation: PRFAQ
Before proceeding, verify:
- [ ] File saved with correct naming convention
- [ ] Press release has compelling headline (not generic)
- [ ] Customer problem clearly articulated with specifics
- [ ] Solution description is concrete (not hand-wavy)
- [ ] Customer quote feels authentic (not corporate speak)
- [ ] FAQ addresses skeptical questions (not softballs)
- [ ] Market research findings incorporated (TAM, competitors referenced)
- [ ] No placeholder text

**FAIL if:** Generic headline, vague problem/solution, or market research not referenced. Fix and re-validate.

### Phase 3: PRD (Requirements)
Reference `#steering/prd-guide.md` for detailed instructions.

Convert PRFAQ into detailed requirements:
- PRD document (`./documents/PRD_[ProductName]_[YYYY-MM-DD].html`)
- Kiro spec files in `.kiro/specs/[product-slug]/`:
  - `requirements.md` - EARS format requirements
  - `design.md` - Technical architecture

**Technical Constraints for design.md:**
- Research current best practices before recommending technologies
- **Only recommend AWS and Anthropic tooling:**
  - Compute: Lambda, ECS, EC2, App Runner
  - Database: DynamoDB, Aurora, RDS
  - AI/ML: Amazon Bedrock, Claude API
  - Storage: S3, EFS
  - API: API Gateway, AppSync
  - Auth: Cognito
- **Do NOT recommend:** OpenAI, Google Cloud, Azure, Vercel, Firebase, Supabase

#### Validation: PRD / Spec
Before proceeding, verify:
- [ ] PRD file saved with correct naming
- [ ] Kiro spec created in `.kiro/specs/[product-slug]/requirements.md`
- [ ] Requirements use EARS syntax (When/The/Shall format)
- [ ] User stories have acceptance criteria
- [ ] All PRFAQ features translated to requirements
- [ ] Technical design references only AWS/Anthropic services
- [ ] No requirements are vague ("should be fast" → "shall respond in <200ms")
- [ ] Edge cases and error states defined

**FAIL if:** Missing EARS format, vague requirements, or non-AWS/Anthropic tech recommended. Fix and re-validate.

### Phase 4: Prototype
Reference `#steering/prototype-guide.md` for detailed instructions.
Apply design standards from `#steering/design-standards.md`.

**IMPORTANT: Create MODULAR files, not a single monolithic HTML.**

Save to `./documents/`:
- `DesignSystem_[ProductName]_[YYYY-MM-DD].html` (shared CSS - create FIRST)
- `ScreenIndex_[ProductName]_[YYYY-MM-DD].html` (navigation hub)
- `Screen_[ScreenName]_[ProductName]_[YYYY-MM-DD].html` (one per screen)

#### Validation: Prototype
Before marking complete, verify:

**Structure (CRITICAL):**
- [ ] DesignSystem file exists with shared CSS variables and components
- [ ] ScreenIndex file exists with links to all screens
- [ ] Individual Screen_*.html files exist (NOT one monolithic file)
- [ ] Each screen links to shared DesignSystem
- [ ] Navigation between screens uses relative links that work

**Functionality:**
- [ ] All PRD screens implemented (cross-reference requirements)
- [ ] All user flows completable end-to-end (no dead ends)
- [ ] Forms have validation feedback (not just static)
- [ ] Responsive at desktop, tablet, mobile breakpoints

**Design Quality:**
- [ ] NO generic fonts (Inter, Roboto, Arial)
- [ ] NO purple-blue gradients on white backgrounds
- [ ] Distinctive aesthetic direction documented and applied
- [ ] Realistic data (no "Lorem ipsum", "Test User", or placeholder content)

**Brand/Product Fidelity:**
- [ ] If modifying existing product: existing UI faithfully recreated
- [ ] If customer brand: logo included and brand colors applied

**After each screen file is created:**
```bash
open ./documents/Screen_[Name]_[ProductName]_[YYYY-MM-DD].html
```

**FAIL if:** Monolithic single-file prototype, dead-end navigation, generic aesthetics, or placeholder content. Fix and re-validate.

## Document Naming

**All documents must be HTML files** so they can be opened in a browser and linked from the dashboard.

All outputs saved to `./documents/`:
```
[Type]_[ProductName]_[YYYY-MM-DD].html
```

Examples:
- `MarketResearch_[ProductName]_[YYYY-MM-DD].html`
- `PRFAQ_[ProductName]_[YYYY-MM-DD].html`
- `PRD_[ProductName]_[YYYY-MM-DD].html`
- `DesignSystem_[ProductName]_[YYYY-MM-DD].html`
- `ScreenIndex_[ProductName]_[YYYY-MM-DD].html`
- `Screen_[Name]_[ProductName]_[YYYY-MM-DD].html`
- `ProjectDashboard_[ProductName]_[YYYY-MM-DD].html`

**Do NOT create .md files.** All documents must be styled HTML that opens in a browser.

## Project Dashboard (REQUIRED)

**Create a project dashboard at the START and update it after EVERY phase.**

Save to: `./documents/ProjectDashboard_[ProductName]_[YYYY-MM-DD].html`

### Dashboard Contents
- Project name and description
- Progress bar showing current phase (Market Research → PRFAQ → PRD → Prototype)
- Links to all generated documents (clickable file:// links)
- Status indicators (completed/validated/in-progress/pending) for each phase
- **Validation status for each phase** (passed/failed/pending)
- Timestamp of last update

### Dashboard Creation (Phase 0 - DO THIS FIRST)
Before starting any work:
1. Create the dashboard HTML file
2. Set all phases to "pending"
3. **Open the dashboard in the browser:**
   ```bash
   open ./documents/ProjectDashboard_[ProductName]_[YYYY-MM-DD].html
   ```
4. Confirm the dashboard is visible to the user

### After EVERY Phase Completion
This is MANDATORY - do not skip:

1. Save the phase document
2. Run validation checks for that phase
3. **Update the dashboard HTML file:**
   - Add link to new document
   - Update phase status (validated/failed)
   - Update progress bar percentage
   - Update timestamp
4. **Re-open the dashboard in the browser:**
   ```bash
   open ./documents/ProjectDashboard_[ProductName]_[YYYY-MM-DD].html
   ```
5. Verbally confirm to user: "Dashboard updated. [Phase] is now complete."
6. Only then proceed to next phase

### If Dashboard Doesn't Open
- Verify the file path is correct
- Check file was saved successfully
- Try: `ls -la ./documents/ProjectDashboard_*.html`
- Manually provide the file path for user to open

## Context Providers

Use these Kiro context providers:
- `#spec` - Reference the current spec
- `#codebase` - Search project files
- `#file` - Reference specific files
- `#git` - Review changes

## Quality Standards

Before completing any phase:
- Verify all files saved correctly
- Check naming conventions followed
- Ensure content is specific (no placeholders)
- Validate consistency with previous phases

## Validation Summary

| Phase | Key Checks | Common Failures |
|-------|-----------|-----------------|
| Market Research | Sources cited, 3+ competitors, fetched pages | Search snippets only, generic pain points |
| PRFAQ | Compelling headline, specific solution, skeptical FAQs | Generic headline, softball questions |
| PRD | EARS format, AWS/Anthropic only, acceptance criteria | Vague requirements, wrong tech stack |
| Prototype | Modular files, working nav, distinctive design, realistic data | Monolithic file, dead ends, generic aesthetics |

**Remember:** Validation is not optional. Every phase must pass validation before proceeding. If you find yourself wanting to skip validation to save time, that's a sign the work needs improvement.

## Cleanup (After Prototype Phase)

**After the Prototype phase is validated**, delete the TeenFit example files:

```bash
rm ./documents/*TeenFit*.html
rm ./documents/*TeenFit*.md
```

The TeenFit files are design examples only - they should not persist after being used as reference. The actual product files you created should remain.
