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

These map to Tailwind's `shadow-*` utilities. If defining as CSS custom properties:

```css
:root {
  /* Card default */
  --shadow-sm: 0 1px 2px oklch(0 0 0 / 5%);

  /* Card hover */
  --shadow-md: 0 4px 6px oklch(0 0 0 / 7%), 0 2px 4px oklch(0 0 0 / 5%);

  /* Dialog */
  --shadow-lg: 0 10px 15px oklch(0 0 0 / 10%), 0 4px 6px oklch(0 0 0 / 5%);

  /* Glassmorphic */
  --shadow-glass: 0 8px 32px oklch(0 0 0 / 8%);
}
```

## Breakpoints

| Name | Width | Usage |
|------|-------|-------|
| `sm` | 640px | Mobile landscape |
| `md` | 768px | Tablet |
| `lg` | 1024px | Desktop |
| `xl` | 1280px | Wide desktop |

Design mobile-first. Use `md:` prefix for tablet+ layouts.
