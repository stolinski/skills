---
name: graffiti-best-practices
description: Use when generating or refactoring Graffiti UI markup so output is class-first, semantic, accessible, responsive, and aligned with current Graffiti capabilities.
metadata:
  version: 1.7.0
---

# Graffiti Best Practices

Generate high-quality Graffiti markup using current library constraints.

Graffiti is the baseline design layer when this skill is active: it provides the default system for elements, layout primitives, utilities, components, and color/theme tokens.

This skill extends Graffiti patterns; it does not create a parallel styling system.

This skill is strict about class-first composition, minimal inline style usage, semantic HTML, and repeatable quality checks.

It is also strict about template-first adaptation: when a matching hosted template baseline exists, start from that template and customize it.

Audience: developers using Graffiti in real applications and websites.

## Package Structure

- `SKILL.md` - activation rules, boundaries, and execution workflow
- `references/OUTPUT_CONTRACT.md` - required response structure and scoring gates
- `references/GRAFFITI_SYSTEM.md` - Graffiti identity, token/variable model, and source-of-truth lookup rules
- `references/COMPONENT_INTENT_MATRIX.md` - role boundaries for core components (use-when, do-not-use, fallback)
- `references/CANONICAL_SNIPPETS.md` - critical pattern snippets that must be used as first-choice baselines
- `references/RECIPES_LAYOUTS.md` - page shell and layout recipes
- `references/RECIPES_SECTIONS.md` - section-level recipes by intent
- `references/RECIPES_COMPONENTS.md` - component-level recipe snippets
- `references/ANTI_PATTERNS.md` - failure mode catalog and detection heuristics
- `references/RECOVERY_TRANSFORMS.md` - deterministic rewrite transforms
- `references/EXAMPLES.md` - prompt/response behavior examples
- `references/TROUBLESHOOTING.md` - ambiguous prompt handling and recovery guidance

## Trigger Conditions

Activate this skill when the request includes one or more of these signals:

- Build or refactor a page/section using Graffiti classes
- Convert inline-styled markup to class-first markup
- Produce template-like structures (landing, dashboard, blog, settings, chat, forms, navigation)
- Improve markup quality, semantics, responsiveness, or consistency for Graffiti outputs

## Do Not Activate For

- Non-UI/backend-only tasks
- Requests to design a brand new CSS framework or token system
- Requests where Graffiti is not the target styling system

## Hard Boundaries (Refusal + Redirect Rules)

1. Do not invent class names not present in current Graffiti CSS/docs.
2. Do not invent CSS custom properties (`--*`) that are not part of Graffiti's documented token/component contracts.
3. Use Graffiti-first composition: exhaust existing classes/tokens before adding custom CSS.
4. Custom CSS is allowed when needed, but apply it systemically: create reusable project-level classes/utilities instead of one-off component-local overrides.
5. Do not hardcode raw color literals (`#hex`, `rgb()`, `oklch(...)`) inline unless explicitly requested.
6. Do not add JS/Svelte state for purely static structure; add state only when interaction or data requirements actually need it.
7. If a requested pattern is not supported by current classes/tokens, return the closest canonical fallback and clearly note the limitation plus an optional reusable extension path.
8. If a matching hosted template baseline exists for the requested intent, use it as the baseline. Do not generate an unrelated structure from scratch unless the user explicitly asks for a fresh build.
9. Do not re-implement built-in Graffiti primitives with bespoke wrappers/CSS/JS. This is a hard fail for critical patterns including dialog/modal, card/card-link, bubble/chat, callout, input-group, form rows/actions.
10. Do not create a duplicate design system (new ad-hoc utility framework, component framework, or token family) when Graffiti primitives already satisfy the request.

## Graffiti Identity (Non-Negotiable)

Treat these as fixed truths for this skill:

- Graffiti is a class-first drop-in CSS system. Markup should compose existing classes instead of rebuilding primitives with inline CSS.
- Graffiti is the baseline layer for page styling in projects using this skill (elements + layouts + utilities + components + full token system).
- The source of truth is hosted Graffiti docs/templates plus local package evidence, in this order:
  1. `https://graffiti-ui.com/base`, `https://graffiti-ui.com/utilities`, `https://graffiti-ui.com/elements`, `https://graffiti-ui.com/ui-blocks` requested with `Accept: text/markdown`
  2. `https://graffiti-ui.com/templates/landing`, `https://graffiti-ui.com/templates/dashboard`, `https://graffiti-ui.com/templates/blog`, `https://graffiti-ui.com/templates/settings`, `https://graffiti-ui.com/templates/ai-chat`
  3. Installed Graffiti stylesheet/package exports in the target project (actual selectors and variable contracts)
  4. Skill recipes in `references/*` (guidance layer, never higher authority than source files)
- Graffiti variables are contracts, not free-form style knobs. Use documented tokens/overrides only.
- If guidance in skill references conflicts with source files, source files win.

## System-First Preflight (Required)

Before writing or editing markup, run this preflight in order:

1. Variables: identify required tokens from `https://graffiti-ui.com/base` markdown and installed Graffiti CSS contracts.
2. Theme: confirm theme/color intent can be solved with Graffiti tokens/classes first.
3. Layout: map top-level structure to existing layout primitives.
4. Utilities: map text/state/alignment behavior to existing utilities.
5. Components: map each requested UI piece to canonical built-in components.

If any category cannot be mapped, choose a documented fallback and record it before writing final markup.

## Component Intent Steering (Required)

Classes are role-bearing primitives, not generic visual wrappers.

Before selecting any component class, verify three fits against `references/COMPONENT_INTENT_MATRIX.md`:

1. **Intent fit**: component role matches the requested job.
2. **Semantic fit**: host element semantics match the pattern.
3. **Shape fit**: content shape (single metric, repeating article, form control row, notice, etc.) matches the primitive.

Hard boundaries:

- `.card` and `.card.featured` are for repeating content units (article/product/plan-like records), not generic section/page wrappers.
- `.card.linked` is for card-as-link preview units, not nav containers or generic anchor wrappers.
- `.stat-card` is for KPI/metric values, not long-form copy or feature marketing blurbs.
- `.feature-card` is for feature-list entries, not arbitrary content blocks.
- If a neutral wrapper is needed, use layout/utilities/neutral surfaces (`layout-*`, `stack`, `cluster`, `split`, `box`, `surface`) instead of role-specific components.

## Variable System Rules

When using inline custom properties or token references:

1. Prefer core tokens documented on `https://graffiti-ui.com/base` markdown and implemented in Graffiti CSS contracts.
2. Prefer semantic status classes before color token overrides (for example `.tag.success` before `--tag-color`).
3. Use component override vars only where documented (for example `--button-color`, `--tag-color`, `--bubble-*`, `--toggle-color`, `--callout-*`).
4. Do not invent new `--*` variable names in markup unless the user explicitly requests a project extension path.
5. For spacing/layout, prefer existing layout classes first; if override is still needed, use documented variables such as `--gap`, `--layout-gap`, `--min-card-width`, `--max-width`.

## Required Workflow

Follow this sequence every time.

1. **Classify intent**
   - Map request to one or more intents: layout shell, neutral container, nav, form, repeating content unit, table/data, chat, utility/text.

2. **Resolve hosted template baseline first**
   - Check `https://graffiti-ui.com/templates/*` for the closest intent match.
   - If a match exists, treat it as the required starter and preserve its canonical section/component composition unless the user asked to replace it.
   - Record the selected template path in the output contract.

3. **Run system-first preflight**
   - Complete variable/theme/layout/utilities/components checks from `references/GRAFFITI_SYSTEM.md`.

4. **Resolve source-of-truth class and variable contracts**
   - For requested components/sections, read `https://graffiti-ui.com/base`, `https://graffiti-ui.com/utilities`, `https://graffiti-ui.com/elements`, and `https://graffiti-ui.com/ui-blocks` with `Accept: text/markdown` and use documented classes/examples as canonical patterns.
   - Use `https://graffiti-ui.com/base` markdown for global token names and categories.
   - If uncertain about availability, confirm against selectors/variables in installed Graffiti CSS.

5. **Create primitive mapping before coding**
   - For each requested component/interaction, map user intent to an existing Graffiti primitive and cite source file.
   - If no direct primitive exists, map to closest fallback and record limitation.

6. **Run component intent fit check**
   - Validate each selected component class against `references/COMPONENT_INTENT_MATRIX.md` before writing markup.
   - If a component only matches visual styling (but not role), remap to the proper primitive.
   - Treat role mismatch as a blocking error, not a stylistic preference.

7. **Resolve canonical snippets for critical patterns**
   - For dialog/modal, card/link, bubble/chat, form actions/options, input-group, and callouts, start from `references/CANONICAL_SNIPPETS.md` baselines.

8. **Select canonical recipe path**
   - Choose the closest reference pattern from:
     - `references/OUTPUT_CONTRACT.md`
     - `references/RECIPES_LAYOUTS.md`, `references/RECIPES_SECTIONS.md`, `references/RECIPES_COMPONENTS.md`, `references/CANONICAL_SNIPPETS.md`
   - Recipes refine the baseline template; they do not replace it when a baseline exists.

9. **Build a class plan before writing markup**
   - Identify layout classes, component classes, utility classes, semantic wrappers, and ARIA/state attributes.
   - Record why each role-specific component was selected and why neutral wrappers were rejected.
   - Build a variable plan: list every `--*` you intend to use and cite its source file.

10. **Apply class-first decision tree**
    - If a class exists, use it.
    - Inline style is only allowed for approved token overrides or bounded layout exceptions (see output contract).
    - If custom CSS is required, prefer reusable extension classes over local one-off overrides.

11. **Write semantic structure first, then style with classes**
    - Use landmarks and semantic tags (`header`, `nav`, `main`, `section`, `article`, `aside`, `footer`, lists, tables, labels).

12. **Run accessibility minimum checks**
    - Labels, heading order, keyboard-relevant semantics, `aria-current`/state attributes, table semantics, media alt text.

13. **Run responsiveness checks**
    - Ensure composition degrades correctly at mobile widths using existing layout primitives.

14. **Run class and variable validation checks**
    - Every class in output must be traceable to hosted docs markdown, hosted templates, or installed Graffiti CSS.
    - Every inline `--*` override must map to documented Graffiti token/override names.
    - Every role-specific component must pass intent-fit validation from `references/COMPONENT_INTENT_MATRIX.md`.
    - Confirm no built-in primitives were re-implemented with custom wrappers/CSS/JS.
    - Confirm no duplicate utility/component/token system was introduced.

15. **Emit output using required contract**
    - Response must follow `references/OUTPUT_CONTRACT.md` section order.
    - Must include a post-edit compliance report proving no duplicate system was created.

16. **Handle ambiguity with deterministic defaults**
    - Apply troubleshooting defaults from `references/TROUBLESHOOTING.md`.
    - Prefer safe class-first fallback plus explicit limitation note over invented classes.

## Hosted Template Baseline Map

Use this map before writing markup:

- Landing/marketing pages -> `https://graffiti-ui.com/templates/landing`
- Dashboard/admin pages -> `https://graffiti-ui.com/templates/dashboard`
- Blog/content pages -> `https://graffiti-ui.com/templates/blog`
- Settings/account pages -> `https://graffiti-ui.com/templates/settings`
- AI chat pages -> `https://graffiti-ui.com/templates/ai-chat`

If no direct match exists, use recipes as primary source and state "No baseline template match found" in the output contract.

## Class-First Decision Tree

1. Is there an existing Graffiti class or class combination for this requirement?
   - **Yes:** use class-based implementation.
   - **No:** go to step 2.

2. Can this be represented as an approved token override?
   - **Yes:** use inline custom properties only (for example `--gap`; use `--tag-color` only for custom categories when semantic tag variants do not fit).
   - **No:** go to step 3.

3. Is it a bounded layout exception (for example one-off max width wrapper)?
   - **Yes:** allow minimal inline declaration and record rationale in verification.
   - **No:** return closest class-based fallback and document limitation + reusable extension option.

Status defaults:

- Prefer semantic tag variants for status labels: `.tag.success`, `.tag.warning`, `.tag.error`, `.tag.info`.
- Use `--tag-color` as a fallback for non-status category colors.
- Prefer `.card.linked` for card-as-link patterns instead of inline link reset styles.
- Prefer `.form-option-row` for checkbox/radio label rows instead of inline `display: inline-flex` recipes.
- Prefer `.row` inside forms/fieldsets for field wrappers (label + input + help text) instead of repeated `stack` + `--gap` compositions.
- Prefer `.form-actions` for submit/cancel rows instead of bare `cluster` compositions (provides responsive stacking).
- Prefer native `<dialog>` + `.close` for modal flows before custom modal wrappers or JS toggles.

## Accessibility Minimums (Must Pass)

- Landmark structure is present for page-level outputs.
- Heading hierarchy is logical (no skipped levels without rationale).
- Every form control has an accessible label.
- Interactive nav/menu patterns include state signals (`aria-current`, `open`, checked state).
- Tables use proper `table`, `thead`, `tbody`, `th`, and `td` semantics.

## Compatibility Policy

- Default output is framework-agnostic HTML.
- If user asks for Svelte, keep markup-first output and only add Svelte logic when interaction or data-binding is explicitly required.
- For static structure in Svelte files, prefer markup-first output without unnecessary script/state blocks.

## Pass/Fail Gates

Treat output as failed if any hard fail occurs:

- Unknown class names
- Unknown custom property names
- Re-implementation of built-in Graffiti primitives
- Role-specific component used outside its intended role boundary (for example `.card` as generic wrapper)
- Duplicate styling system introduced instead of Graffiti composition
- Disallowed inline styles
- Missing required semantic/accessibility structure
- Response does not follow output contract sections
- Matching hosted template exists but output does not use it as baseline (unless user explicitly requested a fresh build)

Treat output as pass only if all sections in `references/OUTPUT_CONTRACT.md` pass their checks.

## References

- `references/OUTPUT_CONTRACT.md`
- `references/GRAFFITI_SYSTEM.md`
- `references/COMPONENT_INTENT_MATRIX.md`
- `references/CANONICAL_SNIPPETS.md`
- `references/RECIPES_LAYOUTS.md`
- `references/RECIPES_SECTIONS.md`
- `references/RECIPES_COMPONENTS.md`
- `references/ANTI_PATTERNS.md`
- `references/RECOVERY_TRANSFORMS.md`
- `references/EXAMPLES.md`
- `references/TROUBLESHOOTING.md`
