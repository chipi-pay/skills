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
