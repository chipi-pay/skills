# Animation Recipes

CSS-first animations using `tw-animate-css` plus custom keyframes.

## Prerequisites

- **Tailwind CSS v4+** (uses `@theme inline` syntax for registering custom animations)
- **tw-animate-css**: Install with `npm install tw-animate-css`, then import in your CSS:
  ```css
  @import "tailwindcss";
  @import "tw-animate-css";
  ```

## Custom Keyframes (add to globals.css)

```css
@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(12px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes checkmark-draw {
  0% {
    stroke-dashoffset: 24;
  }
  100% {
    stroke-dashoffset: 0;
  }
}

@keyframes scale-bounce {
  0% { transform: scale(0); }
  60% { transform: scale(1.15); }
  100% { transform: scale(1); }
}

@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

@keyframes copy-flash {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

@keyframes grain {
  0%, 100% { transform: translate(0, 0); }
  10% { transform: translate(-5%, -10%); }
  30% { transform: translate(3%, -15%); }
  50% { transform: translate(12%, 9%); }
  70% { transform: translate(9%, 4%); }
  90% { transform: translate(-1%, 7%); }
}
```

## Tailwind Utility Classes

Register in `globals.css` under `@theme`:
```css
@theme inline {
  --animate-fade-in-up: fade-in-up 0.4s ease-out both;
  --animate-checkmark: checkmark-draw 0.3s ease-out 0.2s both;
  --animate-scale-bounce: scale-bounce 0.4s ease-out both;
  --animate-shimmer: shimmer 1.5s infinite;
  --animate-copy-flash: copy-flash 0.3s ease-out;
}
```

## Page Load — Staggered Fade-In

```jsx
{/* Stagger cards on page load */}
<div className="animate-fade-in-up" style={{ animationDelay: '0ms' }}>
  <Card>First card</Card>
</div>
<div className="animate-fade-in-up" style={{ animationDelay: '100ms' }}>
  <Card>Second card</Card>
</div>
<div className="animate-fade-in-up" style={{ animationDelay: '200ms' }}>
  <Card>Third card</Card>
</div>
```

## Dialog Entrance

```jsx
{/* DialogOverlay */}
<div className="fixed inset-0 bg-black/50 backdrop-blur-sm animate-in fade-in-0" />

{/* DialogContent */}
<div className="animate-in fade-in-0 slide-in-from-bottom-2 duration-200" />
```

## Success Celebration — Checkmark Draw

```jsx
{/* SVG Checkmark with draw animation */}
<div className="animate-scale-bounce">
  <svg className="h-16 w-16" viewBox="0 0 24 24" fill="none">
    <circle cx="12" cy="12" r="10" className="fill-emerald-500/20 stroke-emerald-500" strokeWidth="2" />
    <path
      d="M8 12.5l2.5 2.5 5-5"
      className="stroke-emerald-500 animate-checkmark"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
      style={{ strokeDasharray: 24, strokeDashoffset: 24 }}
    />
  </svg>
</div>
```

## Card Hover

```jsx
<Card className="hover:shadow-md hover:-translate-y-0.5 transition-all duration-200" />
```

## Button Press

```jsx
<Button className="active:scale-[0.98] transition-transform" />
```

## Loading Skeleton

```jsx
{/* Skeleton placeholder for data regions */}
<div className="animate-shimmer rounded-lg bg-gradient-to-r from-muted via-muted-foreground/5 to-muted bg-[length:200%_100%]">
  <div className="h-12 w-32 invisible">$0.00</div>
</div>
```

## Copy-to-Clipboard Flash

```jsx
const [copied, setCopied] = useState(false);

const handleCopy = async () => {
  await navigator.clipboard.writeText(value);
  setCopied(true);
  setTimeout(() => setCopied(false), 300);
};

<span className={copied ? 'animate-copy-flash' : ''}>
  {displayValue}
</span>
```

## Step Progress Indicator

```jsx
const steps = ['Verify', 'Register', 'Encrypt', 'Done'];
const currentStep = 2;

<div className="flex items-center gap-2">
  {steps.map((label, i) => (
    <div key={label} className="flex items-center gap-2">
      <div className={`
        w-8 h-8 rounded-full flex items-center justify-center text-sm font-semibold transition-all
        ${i < currentStep ? 'bg-accent text-white' :
          i === currentStep ? 'bg-accent/20 text-accent border-2 border-accent' :
          'bg-muted text-muted-foreground'}
      `}>
        {i < currentStep ? '✓' : i + 1}
      </div>
      {i < steps.length - 1 && (
        <div className={`h-0.5 w-8 ${i < currentStep ? 'bg-accent' : 'bg-muted'}`} />
      )}
    </div>
  ))}
</div>
```

## Grain Texture Overlay

Adds animated SVG noise texture to hero/feature sections for visual depth.

### CSS Utility (add to globals.css `@layer utilities`)
```css
.grain-overlay {
  position: relative;
  isolation: isolate;
}
.grain-overlay::before {
  content: "";
  position: absolute;
  inset: -50%;
  width: 200%;
  height: 200%;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.65' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");
  background-repeat: repeat;
  background-size: 256px 256px;
  animation: grain 8s steps(10) infinite;
  pointer-events: none;
  z-index: 1;
  opacity: 0.5;
}
.dark .grain-overlay::before {
  opacity: 0.3;
}
```

### Usage
```jsx
{/* Content must use relative z-10 to sit above the grain */}
<section className="grain-overlay bg-gradient-to-br from-orange-50 via-white to-emerald-50">
  <h1 className="relative z-10 text-7xl font-extrabold">Hero Title</h1>
  <p className="relative z-10 text-xl">Subtitle</p>
</section>
```

## Responsive Animation Rules

### Reduced Motion
Always respect `prefers-reduced-motion`:
```css
@media (prefers-reduced-motion: reduce) {
  .animate-fade-in-up {
    animation: none !important;
    opacity: 1 !important;
  }
  .grain-overlay::before {
    animation: none !important;
  }
}
```

### Mobile Adjustments
- **Stagger delays**: keep to 100ms increments max (avoid > 300ms total delay on mobile — users won't wait)
- **Hover effects**: only on `md:` and above — mobile doesn't hover. Card lift is fine because `transition-all` handles touch gracefully
- **Hero animations**: `text-5xl` on mobile, `md:text-7xl` on tablet+. Animation stays the same
- **Step progress**: reduce dot width to `w-6` on mobile if more than 4 steps
- **Dialog entrance**: `slide-in-from-bottom-4` on mobile (more dramatic), `slide-in-from-bottom-2` on desktop
- **Touch targets**: all animated interactive elements must be minimum 44x44px

### Performance
- All animations are CSS-only — no JavaScript animation libraries
- `will-change` is NOT needed for these simple transforms
- Grain overlay uses `steps(10)` to reduce GPU compositing
- `backdrop-blur` is GPU-intensive — limit to one layer per screen (dialog OR card, not both stacked)
