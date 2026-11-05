---
name: redesigning-vue-saas-layouts
description: Use when redesigning a Vue 3 app's UI into a modern SaaS-style interface, replacing a top navigation bar with a vertical left sidebar, or adding a collapsible icons-only sidebar.
---

# Redesigning Vue SaaS Layouts

## Overview

A layout redesign changes how the app looks, not what it does. The finished diff contains shell structure and styling, and every existing behaviour still works the same way.

## When to Use

- Moving a top nav into a left sidebar
- Restyling an app into a consistent SaaS look
- Adding a collapsible or icons-only sidebar

Skip this for new features, or for visual changes to a single component.

## Scope Contract

Each changed line belongs to one of these categories:

| Category | Examples |
|---|---|
| Shell structure | Sidebar, main column, moving existing menus |
| Styling | Tokens, spacing, cards, tables, states |
| Requested sidebar behaviour | Collapse toggle, saved preference, responsive rail |

Keep these exactly as they are: routes, API calls, props and emits, element types, event handlers, data loading, and the text of existing labels. Hardcoded labels stay hardcoded.

Put any behaviour improvement you notice, such as closing a menu on outside click, in a **Suggested follow-ups** list in your final report.

## Steps

1. **Map the shell.** Find the header, sticky or fixed elements, z-index values, and pixel offsets tied to the old header height.
2. **Add tokens.** Define colours, spacing, radii, z-index and sidebar widths in `:root` in the global stylesheet.
3. **Build the sidebar.** Use `position: fixed`, give the main column a matching `margin-left` from the same width token, and keep `min-width: 0` on it.
4. **Reset offsets.** Each sticky element that sat below the header now uses `top: 0`.
5. **Layer correctly.** Use the order bar, sidebar, modal. Check existing modals sit above the sidebar.
6. **Add collapse.** Store the preference in `localStorage`, wrapping each read and write in `try/catch`. Use a CSS or `matchMedia` breakpoint for the rail.
7. **Make the rail usable.** Use inline SVG with `aria-hidden`, a `title` and accessible name on each link, and `aria-expanded` on the toggle. Check that menus opening from the rail stay on screen.
8. **Translate new strings.** If the app has locale files, add each new key to every locale in the same change. Use only the strings the sidebar needs, such as the toggle labels.
9. **Verify in a browser.** Check each route expanded, collapsed and at narrow width, in each language, with each modal open, and compare the console with its state before the redesign.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Stale `top: 70px` on a filter bar | Reset it to `top: 0` |
| Sidebar width repeated in several rules | Use one token |
| `overflow: hidden` on the sidebar clips menus | Set overflow on the nav list only |
| New key added to one locale | Add it to all locale files |
| Reporting success from a build alone | Render every route in a browser |
