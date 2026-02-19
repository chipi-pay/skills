# Chipi Design Tokens

## Color Palette (OKLCH)

### Brand Colors
```css
/* Primary accent — Chipi orange */
--accent:           oklch(0.75 0.18 55);
--accent-light:     oklch(0.92 0.06 55);    /* Background tints */
--accent-dark:      oklch(0.55 0.20 55);    /* Hover states */

/* Success — Emerald */
--success:          oklch(0.72 0.17 160);
--success-light:    oklch(0.92 0.06 160);
--success-dark:     oklch(0.55 0.17 160);

/* Primary text — Blue-slate */
--primary:          oklch(0.20 0.02 260);
--primary-light:    oklch(0.40 0.02 260);
--primary-foreground: oklch(0.985 0 0);

/* Neutrals — Slate */
--muted:            oklch(0.97 0.005 260);
--muted-foreground: oklch(0.50 0.02 260);
--background:       oklch(0.98 0.003 260);
--border:           oklch(0.90 0.01 260);
```

### Dark Mode Overrides
```css
.dark {
  --accent:           oklch(0.78 0.16 55);
  --accent-light:     oklch(0.25 0.08 55);
  --primary:          oklch(0.92 0.01 260);
  --primary-foreground: oklch(0.15 0.02 260);
  --background:       oklch(0.12 0.02 260);
  --muted:            oklch(0.18 0.02 260);
  --muted-foreground: oklch(0.65 0.02 260);
  --border:           oklch(1 0 0 / 10%);
}
```

## Typography Scale

### Font Families
```css
--font-sans: 'Plus Jakarta Sans', system-ui, sans-serif;
--font-mono: 'JetBrains Mono', 'Fira Code', monospace;
```

### Scale (Mobile → Desktop)

| Token | Mobile | Desktop | Usage |
|-------|--------|---------|-------|
| `text-xs` | 12px | 12px | Captions, labels |
| `text-sm` | 14px | 14px | Secondary text |
| `text-base` | 16px | 16px | Body text (minimum for readability) |
| `text-lg` | 18px | 18px | Card titles |
| `text-xl` | 20px | 20px | Section subtitles |
| `text-3xl` | 30px | 30px | Section headings |
| `text-5xl` | 48px | 48px | Page titles |
| `text-7xl` | 72px | 72px | Hero headlines |

### Weight Mapping

| Weight | Class | Usage |
|--------|-------|-------|
| 400 | `font-normal` | Body text |
| 500 | `font-medium` | Labels, nav items |
| 600 | `font-semibold` | Card titles, buttons |
| 700 | `font-bold` | Section headings |
| 800 | `font-extrabold` | Hero headlines |

### Financial Data Rules
```html
<!-- ALWAYS use for money/numbers -->
<span class="font-mono tabular-nums">$1,234.56</span>

<!-- Balance display -->
<p class="text-5xl font-extrabold font-mono tabular-nums">
  $12,450.00
</p>
```

## Spacing System

Base: 4px grid. Use Tailwind spacing utilities.

| Token | Value | Usage |
|-------|-------|-------|
| `gap-1` | 4px | Tight inline elements |
| `gap-2` | 8px | Related items |
| `gap-4` | 16px | Card content spacing |
| `gap-6` | 24px | Section padding |
| `gap-8` | 32px | Between sections |
| `gap-12` | 48px | Major section breaks |

### Component Padding
- Cards: `p-6` (24px)
- Dialogs: `p-6` (24px)
- Buttons: `px-4 py-2` (16px/8px) default, `px-6 py-3` (24px/12px) large
- Input fields: `px-3 py-2` (12px/8px)

## Border Radius

```css
--radius: 0.625rem;        /* 10px — base */
--radius-sm: 0.375rem;     /* 6px */
--radius-md: 0.5rem;       /* 8px */
--radius-lg: 0.625rem;     /* 10px */
--radius-xl: 0.875rem;     /* 14px */
```

## Shadows

Full shadow scale mapped to Tailwind's `shadow-*` utilities:

```css
:root {
  /* Subtle — input fields, badges */
  --shadow-xs: 0 1px 2px oklch(0 0 0 / 3%);

  /* Default — cards at rest */
  --shadow-sm: 0 1px 3px oklch(0 0 0 / 5%), 0 1px 2px oklch(0 0 0 / 3%);

  /* Hover — cards on hover, dropdowns */
  --shadow-md: 0 4px 6px oklch(0 0 0 / 7%), 0 2px 4px oklch(0 0 0 / 5%);

  /* Elevated — dialogs, popovers */
  --shadow-lg: 0 10px 15px oklch(0 0 0 / 10%), 0 4px 6px oklch(0 0 0 / 5%);

  /* Prominent — modals, toasts */
  --shadow-xl: 0 20px 25px oklch(0 0 0 / 10%), 0 8px 10px oklch(0 0 0 / 4%);

  /* Maximum — full-screen overlays */
  --shadow-2xl: 0 25px 50px oklch(0 0 0 / 15%);

  /* Glassmorphic — frosted glass components */
  --shadow-glass: 0 8px 32px oklch(0 0 0 / 8%);

  /* Inner — inset for input focus, pressed states */
  --shadow-inner: inset 0 2px 4px oklch(0 0 0 / 5%);
}
```

### Shadow Usage Guide

| Token | Tailwind Class | Usage |
|-------|---------------|-------|
| `--shadow-xs` | `shadow-xs` | Input fields, badges, chips |
| `--shadow-sm` | `shadow-sm` | Cards at rest |
| `--shadow-md` | `shadow-md` | Cards on hover, dropdowns |
| `--shadow-lg` | `shadow-lg` | Dialogs, popovers |
| `--shadow-xl` | `shadow-xl` | Modals, toast notifications |
| `--shadow-2xl` | `shadow-2xl` | Full-screen overlays |
| `--shadow-glass` | (custom) | Glassmorphic components |
| `--shadow-inner` | `shadow-inner` | Input focus, pressed buttons |

## Icon System

Use **Lucide React** exclusively. Never mix icon libraries.

```bash
npm install lucide-react
```

### Sizing Convention

| Context | Size | Class |
|---------|------|-------|
| Inline text | 16px | `h-4 w-4` |
| Button icon | 16px | `h-4 w-4` |
| Card / section | 20px | `h-5 w-5` |
| Feature highlight | 24px | `h-6 w-6` |
| Dialog status | 48px | `h-12 w-12` |
| Success celebration | 64px | `h-16 w-16` |

### Common Icons

| Purpose | Icon | Import |
|---------|------|--------|
| Security / encryption | `ShieldCheck` | `lucide-react` |
| Biometric / passkey | `Fingerprint` | `lucide-react` |
| PIN / key | `KeyRound` | `lucide-react` |
| Loading spinner | `Loader2` + `animate-spin` | `lucide-react` |
| Copy to clipboard | `Copy` / `Check` (toggle) | `lucide-react` |
| Send / transfer | `ArrowUpRight` | `lucide-react` |
| Warning | `AlertTriangle` | `lucide-react` |
| Success | `CheckCircle2` | `lucide-react` |
| External link | `ExternalLink` | `lucide-react` |

## Layout & Container

```css
/* Max content widths */
--max-w-content: 1280px;   /* Main content area */
--max-w-prose: 65ch;        /* Text-heavy sections */
--max-w-dialog: 28rem;      /* sm:max-w-md — dialogs */
--max-w-card: 24rem;        /* Individual card max */
```

### Grid Patterns

```tsx
{/* Card grid — responsive */}
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

{/* Feature grid — 2 column on tablet+ */}
<div className="grid grid-cols-1 md:grid-cols-2 gap-8">

{/* Centered content — text sections */}
<div className="mx-auto max-w-prose px-4 md:px-6">
```

## Breakpoints

| Name | Width | Usage |
|------|-------|-------|
| `sm` | 640px | Mobile landscape |
| `md` | 768px | Tablet |
| `lg` | 1024px | Desktop |
| `xl` | 1280px | Wide desktop |

Design mobile-first. Use `md:` prefix for tablet+ layouts.
