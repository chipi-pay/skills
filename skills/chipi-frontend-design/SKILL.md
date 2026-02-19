---
name: chipi-frontend-design
description: Design system and UX guidance for Chipi-powered applications — typography, color, motion, and fintech guardrails.
author: Chipi Pay
version: 1.0.0
tags: [design, ux, frontend, fintech]
---

# Chipi Frontend Design System

Apply these rules to every UI you generate for Chipi-powered applications.

## Typography

- **NEVER** use Inter, Roboto, or system-ui as display fonts
- **Default pairing**: Plus Jakarta Sans (display/body) + JetBrains Mono (code/data)
- Headings: `text-5xl` minimum for hero, `text-3xl` for section titles. Use `font-bold` or `font-extrabold`
- Financial figures: `tabular-nums font-mono` — always. Inconsistent digit widths break scanning
- Body: `text-base` (16px) minimum. Never smaller than `text-sm` for readable content

## Color — CSS Variables (OKLCH)

Use the Chipi brand palette via CSS custom properties:

| Token | Value | Usage |
|-------|-------|-------|
| `--accent` | `oklch(0.75 0.18 55)` | Chipi orange — CTAs, active states, links |
| `--success` | `oklch(0.72 0.17 160)` | Emerald — confirmations, positive amounts |
| `--primary` | `oklch(0.20 0.02 260)` | Blue-slate — headings, primary text |
| `--muted` | `oklch(0.97 0.005 260)` | Slate tint — backgrounds, cards |

- **NEVER** flat `bg-white` for page backgrounds. Use `bg-[var(--muted)]` or subtle gradients
- Success states: emerald. Error states: existing `--destructive`. Warnings: amber
- Dark mode: invert lightness, keep chroma. Orange stays orange

## Motion — CSS-first (tw-animate-css)

- Page load: stagger `animate-fade-in-up` on cards with `delay-[100ms]`, `delay-[200ms]`, etc.
- Dialogs: `animate-in slide-in-from-bottom-2` + `backdrop-blur-sm` on overlay
- Success: draw-checkmark SVG animation or scale-bounce on icon
- Hover: `hover:shadow-md hover:-translate-y-0.5 transition-all duration-200` on cards
- Press: `active:scale-[0.98]` on buttons
- **NEVER** `animate-bounce` on non-playful UI. **NEVER** animation longer than 500ms

## Glassmorphic Components

Apply to cards, dialogs, and overlays:
```
bg-white/70 dark:bg-black/50 backdrop-blur-xl
border border-white/20 dark:border-white/10
shadow-lg
```
Glass adapts to any background — gradients, images, solid colors.

## Backgrounds

- Hero sections: gradient backgrounds (`bg-gradient-to-br from-orange-50 via-white to-emerald-50`)
- **NEVER** flat `#fff` or `bg-white` for full-page backgrounds
- Add depth with layered elements or subtle noise textures

## Fintech UX Guardrails

- Amounts: always 2 decimal places (`$10.00` not `$10`), `tabular-nums`
- Destructive actions: require explicit confirmation dialog
- Transaction states: show step indicators (Authenticating > Signing > Done)
- Loading: use skeleton screens, not spinners, for data regions
- Errors: actionable messages ("Transfer failed — check recipient address" not "Error")
- Copy-to-clipboard: show brief flash feedback (`animate-pulse` for 300ms)

## Anti-Slop Rules

**NEVER:**
- Rounded-full avatars with gradient borders (generic AI pattern)
- Excessive emoji in UI text
- "Lorem ipsum" placeholder content
- Gray-on-white text below 4.5:1 contrast ratio
- Spinners as the only loading indicator for regions > 100px

**ALWAYS:**
- Brand orange (`accent`) for primary CTAs
- Monospace + `tabular-nums` for any number
- Step indicators for multi-step flows
- `aria-label` on icon-only buttons
- Minimum touch target: 44x44px on mobile

## Reference Files

For detailed implementation recipes, see:
- `references/design-tokens.md` — Full color palette, typography scale, spacing system
- `references/animation-recipes.md` — Copy-paste CSS keyframes and Tailwind classes
- `references/component-enhancement-recipes.md` — Before/after shadcn upgrades
- `references/anti-slop-fintech.md` — Comprehensive fintech UX patterns
