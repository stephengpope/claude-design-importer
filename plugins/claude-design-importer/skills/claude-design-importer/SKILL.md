---
name: claude-design-importer
description: Use when given an HTML mockup, design file, or screenshot and asked to implement, build, or recreate it in a Tailwind/shadcn codebase — even a bare "build this" with no mention of quality. Converts inline styles and magic values like text-[13.5px] into tokenized components. Also use to clean up an already-imported, px-littered design.
---

# Claude Design Importer

A design mockup — from Claude's design tool, a design file, or a screenshot — is a
**pixel-exact visual artifact**: inline `style="..."`, hardcoded `font-size:13.5px`,
literal hex colors, raw `<div>`s. It shows the intended *look*. It is not a component
system. Your job is to rebuild that look as idiomatic, quality **Tailwind v4 +
shadcn/Radix** components.

## The golden rule

The mockup is the intended **look, not the implementation**. Reproduce the look by
re-expressing it through the project's component primitives and design tokens — never
by transliterating inline styles and magic numbers 1:1. Treat every literal value as
an *approximation to snap onto the nearest existing token*. Half-pixel sizes like
`13.5px` are design-tool noise, not intent.

## Build from primitives, not raw divs

A styled `<div>` that looks like a button is not a button. Map the mockup's elements
to components the project already has:

- shadcn/Radix primitives — `Button`, `Card`, `Dialog`, `Input`, `Select`, `Badge`,
  `Avatar`, `Tabs`, `DropdownMenu`, `Tooltip`, …
- Reuse their variants (CVA) and `data-slot` styling hooks; extend them, don't rebuild.
- You get accessibility, keyboard handling, and focus management for free.

## Snap to the existing token system — don't invent one

Importing is mechanical, not greenfield design. Map each value to the nearest token
the framework/project already provides. Do **not** design a new modular type scale or
hand-tune line-heights — the defaults already ship tuned values. Snapping is
deterministic: compute and apply it; don't stall asking which token to use.

- **Type:** `text-[Npx]` → nearest named `text-*` (tie → the larger, for legibility).
  e.g. `13.5px → text-sm`, `27px → text-3xl`.
- **Spacing / size:** px → the spacing scale. On-grid (÷4) is pixel-identical
  (`w-[272px] → w-68`); off-grid snaps to the nearest step.
- **Color:** hex/rgba → existing semantic tokens (`text-foreground`, `bg-card`,
  `primary`, …). Drive everything from CSS variables so light/dark mode work; never
  hardcode hex. If a color has no token, add one in `@theme` — don't inline it.
- **Radii / shadows / letter-spacing:** → the radius / shadow / tracking tokens.

## Make it real quality, not just tokenized

- **Semantic, accessible markup:** real headings and landmarks, labeled controls,
  visible `focus-visible` states, correct ARIA (Radix handles most of it).
- **Responsive:** the mockup is one fixed width — make the layout hold up across sizes.
- **Theme-aware by construction:** because colors are tokens, dark mode follows for
  free. Confirm it actually does.
- **Compose and reuse:** factor repeated patterns into components with props/variants
  instead of copy-pasting blocks.

## Cover every category; know what legitimately stays arbitrary

Tokenize **all** systematic arbitrary values — type, spacing, color, radii, shadow,
letter-spacing — and sweep until each is gone.

Leave genuinely one-off values; these are correct Tailwind, not debt:
- viewport units (`max-h-[86vh]`), `calc()`, hairline `border-[1.5px]`, custom grid
  templates (`grid-cols-[1fr_148px_44px]`).

Never rewrite **variant selectors** — `data-[state=open]`, `has-[…]`, `group-[…]`,
`aria-[…]` are not values; changing them breaks behavior.

Vendored shadcn `ui/*` source is yours to tokenize too. The one thing to avoid:
redefining a **shared scale token** (e.g. `--radius`) blindly — that cascades to every
component. Map onto the existing scale instead of redefining it.

## A design import isn't done until you've seen it rendered

A "no `[px]` left" grep does not prove it looks right. Run the app and verify:

- Screenshot the affected surfaces **before and after**, and compare.
- Check the states that hide regressions: **dark mode**, **populated** (not empty)
  data, **sub-tabs / modals / dropdowns**, and **auth-gated routes** (log in for a
  session cookie and drive a headless browser — don't skip a page because it needs
  auth).
- Confirm the framework actually **generates** the utilities you emit (compile the CSS
  and grep for the rule; CSS escapes dots, so `.w-0\.5`).
- Run the **production build** and the **linter**.

Report what you verified — and, plainly, anything you couldn't (e.g. a state with no
data to render).
