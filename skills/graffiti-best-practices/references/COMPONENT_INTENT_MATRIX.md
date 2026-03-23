# Component Intent Matrix

Use this matrix to prevent class selection by appearance alone.

Core rule: component classes are role-bearing primitives, not generic wrappers.

If a class does not match the requested job, do not use it even if it looks visually close.

## How to Apply

For every role-specific class in output:

1. Confirm the requested intent matches the class role.
2. Confirm semantic host element fits the pattern.
3. Confirm content shape fits (metric, article preview, form row, notice, message, etc.).
4. If any check fails, remap to a neutral layout/surface primitive.

## Matrix

| Primitive                                | Use when                                                                                       | Typical semantic host                                     | Do not use for                                                            | Prefer instead when mismatch                               |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------- |
| `.card`                                  | Repeating content records (article/product/plan/summary units) with title/body/meta/actions    | `<article class="card">`                                  | Generic page/section wrappers, shell spacing, arbitrary grouping          | `box`, `surface`, `stack`, `layout-*`                      |
| `.card.featured`                         | One emphasized item in a group of otherwise standard cards (for example featured pricing tier) | `<article class="card featured">`                         | Standalone highlight container with no card context, generic accent block | `.card` (non-featured), `callout`, `surface`               |
| `.card.linked`                           | Card-as-link preview unit where the full card is one destination                               | `<a class="card linked" href="...">`                      | Nav menus, button groups, wrappers around multiple different links        | Standard links/lists, `sidebar-nav`, `cluster`             |
| `.feature-card`                          | Marketing/feature-list item with short explanatory copy                                        | `<article class="feature-card">`                          | KPI metrics, article feeds, pricing plan cards, generic shells            | `stat-card`, `card`, `box`                                 |
| `.stat-card`                             | Single KPI/metric readout (label + value + delta/status)                                       | `<article class="stat-card">`                             | Long-form descriptive content, multi-action cards, article previews       | `card`, `feature-card`, `table`                            |
| `.callout` + variants                    | Contextual notice/warning/success/error message blocks                                         | `<div class="callout ...">`                               | Main content cards, persistent section wrappers, pricing/feature tiles    | `card`, `box`, semantic sections                           |
| `.row` (form context)                    | Label + control + optional help text grouping inside forms/fieldsets                           | `<div class="row">` inside `<form>`/`<fieldset>`          | Generic horizontal layout wrappers outside form semantics                 | `split`, `cluster`, `stack`                                |
| `.form-option-row`                       | Compact checkbox/radio option rows                                                             | `<label class="form-option-row">`                         | Arbitrary icon/text rows, non-form alignment wrappers                     | `cluster`, `split`, `row`                                  |
| `.form-actions`                          | Submit/cancel action row at form end with responsive stacking behavior                         | `<div class="form-actions">`                              | Header toolbars, card CTA rows not tied to form submission                | `cluster`, `split`                                         |
| `.input-group`                           | Single input coupled with appended button/control (search, invite, subscribe)                  | `<div class="input-group">`                               | Full multi-field forms, unrelated controls in one row                     | `row`, `stack`, `cluster`                                  |
| `.chat-thread` + `.chat-row` + `.bubble` | Sequential chat conversation transcript (assistant/user messages)                              | `<section class="chat-thread">` with `.chat-row` children | Generic comments/feed list, notices, FAQ accordions                       | `stack`, `card`, `callout`, semantic list/article patterns |
| `.toc`                                   | On-page heading index for a long document                                                      | `<nav class="toc" aria-label="Table of contents">`        | Primary site navigation, sidebar app menus                                | `nav`, `sidebar-nav`                                       |
| `.table` (wrapper)                       | Real tabular datasets needing native table semantics and overflow handling                     | `<div class="table"><table>...`                           | Card grids, marketing layouts, non-tabular content                        | `layout-card`, `layout-three-col`, `stack`                 |

## Neutral Wrapper Defaults

When the user asks for "a container/wrapper/section" without a clear component role, default to neutral composition:

- Layout: `layout-*`, `stack`, `cluster`, `split`
- Surface wrappers: `box`, `surface`
- Semantic structure: `section`, `article`, `aside`, `header`, `footer` (without forcing component classes)

## Card-Specific Guardrails

- `.card` implies a record-like content unit, usually part of a repeated set.
- If content is not record-like, do not use `.card` only for border/padding aesthetics.
- If the whole unit is clickable, use `.card.linked` with root anchor semantics.
- If only one card is "special", use `.card.featured` in the context of a card group.

## Ambiguity Defaults

When uncertain, prefer the lower-risk choice:

1. Use a neutral wrapper (`box`/`surface` + layout classes).
2. Keep semantic HTML intact.
3. Document the unresolved intent in limitations instead of forcing a mismatched component.
