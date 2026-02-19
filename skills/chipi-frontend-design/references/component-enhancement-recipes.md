# Component Enhancement Recipes

Before/after patterns for upgrading shadcn/ui components with the Chipi design system.

## Card — Glassmorphic + Hover

### Before (default shadcn)
```jsx
<Card className="border rounded-xl">
  <CardContent>Plain card</CardContent>
</Card>
```

### After (Chipi)
```jsx
<Card className="bg-white/70 dark:bg-black/50 backdrop-blur-xl border border-white/20 dark:border-white/10 shadow-lg hover:shadow-xl hover:-translate-y-0.5 transition-all duration-200">
  <CardContent>Glassmorphic card</CardContent>
</Card>
```

## Dialog — Slide-in + Frosted Backdrop

### Before
```jsx
<DialogOverlay className="bg-black/50" />
<DialogContent className="bg-background border" />
```

### After
```jsx
<DialogOverlay className="bg-black/40 backdrop-blur-sm" />
<DialogContent className="bg-white/80 dark:bg-neutral-900/80 backdrop-blur-xl border border-white/20 shadow-2xl animate-in slide-in-from-bottom-2 fade-in-0 duration-200" />
```

## Button — Press Effect + Brand Variant

### Before
```jsx
<Button>Default button</Button>
```

### After
```jsx
{/* All buttons get press effect via base styles */}
<Button className="active:scale-[0.98]">Tactile button</Button>

{/* Brand variant for primary CTAs */}
<Button variant="brand">
  Get Started
</Button>
```

### Brand Variant Definition
```ts
brand:
  "bg-[oklch(0.75_0.18_55)] text-white shadow-xs hover:bg-[oklch(0.55_0.20_55)] active:scale-[0.98]",
```

## Balance Display — Tabular Nums + Skeleton

### Before
```jsx
<p className="text-8xl">${balance}</p>
```

### After
```jsx
{isLoading ? (
  <div className="h-20 w-48 animate-shimmer rounded-lg bg-gradient-to-r from-muted via-muted-foreground/5 to-muted bg-[length:200%_100%]" />
) : (
  <p className="text-5xl font-extrabold font-mono tabular-nums">
    ${Number(balance).toFixed(2)}
  </p>
)}
```

## Hero Section — Gradient + Motion

### Before
```jsx
<section className="bg-orange-100 py-25 text-center">
  <h1 className="text-6xl font-bold">Title</h1>
</section>
```

### After
```jsx
<section className="relative overflow-hidden bg-gradient-to-br from-orange-50 via-white to-emerald-50 dark:from-orange-950/30 dark:via-neutral-900 dark:to-emerald-950/20 py-28 text-center">
  <h1 className="text-7xl sm:text-8xl font-extrabold tracking-tight animate-fade-in-up">
    Title
  </h1>
  <p className="animate-fade-in-up text-xl" style={{ animationDelay: '100ms' }}>
    Subtitle
  </p>
</section>
```

## Form Fields — Spacing + Currency Indicator

### Before
```jsx
<FormItem>
  <FormLabel>Amount (USDC)</FormLabel>
  <FormControl>
    <Input type="number" />
  </FormControl>
</FormItem>
```

### After
```jsx
<FormItem className="space-y-2">
  <FormLabel>Amount</FormLabel>
  <FormControl>
    <div className="relative">
      <span className="absolute left-3 top-1/2 -translate-y-1/2 text-muted-foreground font-mono">$</span>
      <Input
        type="number"
        inputMode="decimal"
        className="pl-7 font-mono tabular-nums"
        aria-label="Amount in USDC"
      />
      <span className="absolute right-3 top-1/2 -translate-y-1/2 text-xs text-muted-foreground font-medium">USDC</span>
    </div>
  </FormControl>
</FormItem>
```

## Step Progress Bar (Multi-step Dialogs)

### Recipe
```jsx
function StepProgress({ steps, currentStep }: { steps: string[]; currentStep: number }) {
  return (
    <div className="flex items-center justify-center gap-1 mb-6">
      {steps.map((label, i) => (
        <div key={label} className="flex items-center gap-1">
          <div
            className={`h-1.5 w-8 rounded-full transition-colors duration-300 ${
              i <= currentStep ? 'bg-accent' : 'bg-muted'
            }`}
            title={label}
          />
        </div>
      ))}
    </div>
  );
}
```

## Wallet Address — Copy with Flash

### Before
```jsx
<Button onClick={copy}>{shortAddress} <CopyIcon /></Button>
```

### After
```jsx
// Requires: npm install sonner
// Import: import { toast } from "sonner"
// Wrap app with <Toaster /> provider (see sonner docs)
const [copied, setCopied] = useState(false);

<Button
  variant="ghost"
  onClick={async () => {
    await navigator.clipboard.writeText(fullAddress);
    setCopied(true);
    setTimeout(() => setCopied(false), 1500);
    toast.success("Copied");
  }}
  className="font-mono text-sm"
>
  <span className={copied ? 'text-accent transition-colors' : 'transition-colors'}>
    {shortAddress}
  </span>
  {copied ? <CheckIcon className="h-4 w-4 text-accent" /> : <CopyIcon className="h-4 w-4" />}
</Button>
```
