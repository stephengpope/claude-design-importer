# Claude Design Importer

A Claude Code skill that turns a **static design mockup** — especially the
pixel-exact HTML that Claude's design tool (or a similar generator) produces —
into **production Tailwind / shadcn-on-Radix components that use a real token
system**, instead of freezing the mockup's magic pixel values into your codebase.

## The problem it fixes

Design mockups are visual artifacts: inline styles, hardcoded `font-size:13.5px`,
literal hex colors, raw `<div>`s. A naive import translates them 1:1 —
`font-size:13.5px` → `text-[13.5px]`, `padding:9px 11px` → arbitrary px padding,
`#7C5CFC` → `text-[#7C5CFC]`. The result *looks* identical but:

- ignores your type scale (scattered `text-[13.5px]`, `text-[11.5px]`, …)
- ignores your spacing scale (arbitrary px paddings)
- hardcodes colors that already have semantic tokens
- uses px instead of rem, so it ignores the user's root font size and reads tiny

## What the skill does instead

It treats the mockup as the **intended look, not the implementation**, and
re-expresses it through a token system:

1. **Preflight plan, then stop for approval** — inventories the mockup, detects your
   stack, and proposes a token mapping table *before* writing any code.
2. **A modular, rem-based type scale** — base 16px, a deliberate ratio (1.25 / 1.333),
   with per-step tuned line-height and letter-spacing. Every mockup px size snaps to
   the nearest step.
3. **Semantic color tokens, an 8px spacing scale, and shadcn/Radix primitives** — reused
   from your project, not reinvented.
4. **A self-check** — greps its own output for `[…px]`, raw hex, and inline styles.

The forced plan-first gate is the whole point: the px-soup happens when an importer
jumps straight to code and copies numbers.

## Install

```
/plugin marketplace add stephengpope/claude-design-importer
/plugin install claude-design-importer
```

Then hand Claude a mockup:

> "Build out this design into the app: ./Kanban.html"

The skill triggers on design-import / mockup-porting work and runs the plan-first flow.

## Why this is the right approach

Professional design systems (Radix Themes, IBM Carbon, Material 3, Atlassian) never
make a UI bigger by multiplying every value by a constant. They set a correct base and
let a token system carry it — rem-based type tokens with tuned line-heights, a separate
8px spacing scale, and discrete density/scale settings for user control. This skill
makes the import follow that discipline.

## License

MIT — see [LICENSE](./LICENSE).
