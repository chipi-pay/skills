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

### Brand Variant Definition (via CSS variables)
```ts
brand:
  "bg-brand text-white shadow-xs hover:bg-brand-hover active:scale-[0.98]",
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

## Toast Notifications — Branded + Accessible

### Before
```jsx
alert("Transaction complete!");
```

### After
```jsx
// Requires: npm install sonner
// Wrap app with <Toaster /> (see sonner docs)
import { toast } from "sonner";

// Success
toast.success("Transaction confirmed", {
  description: "Sent $50.00 USDC to 0x1234...abcd",
});

// Error — actionable
toast.error("Transfer failed", {
  description: "Insufficient balance: you have $4.50",
  action: { label: "Check balance", onClick: () => router.push("/wallet") },
});
```

## Transaction Status Badge

### Before
```jsx
<span className="text-green-500">Confirmed</span>
```

### After
```jsx
const statusStyles = {
  pending:   "bg-amber-500/10 text-amber-600 border-amber-500/20",
  confirmed: "bg-success/10 text-success border-success/20",
  failed:    "bg-destructive/10 text-destructive border-destructive/20",
} as const;

<span className={`inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-medium border ${statusStyles[status]}`}>
  {status === "pending" && <Loader2 className="h-3 w-3 animate-spin" />}
  {status === "confirmed" && <CheckCircle2 className="h-3 w-3" />}
  {status === "failed" && <AlertTriangle className="h-3 w-3" />}
  {status.charAt(0).toUpperCase() + status.slice(1)}
</span>
```

## PIN Input — Masked + Accessible

### Before
```jsx
<input type="password" maxLength={4} placeholder="Enter PIN" />
```

### After
```jsx
import { InputOTP, InputOTPGroup, InputOTPSlot } from "@/components/ui/input-otp";

<div className="flex flex-col items-center space-y-3">
  <label className="text-sm uppercase tracking-wider text-muted-foreground">
    Enter PIN
  </label>
  <p className="text-xs text-muted-foreground text-center max-w-[250px]">
    Enter the 4-digit PIN you used to create your wallet.
  </p>
  <InputOTP maxLength={4} value={pin} onChange={setPin} disabled={isProcessing} autoFocus>
    <InputOTPGroup className="gap-2">
      <InputOTPSlot index={0} className="w-14 h-14 text-xl border-2" mask />
      <InputOTPSlot index={1} className="w-14 h-14 text-xl border-2" mask />
      <InputOTPSlot index={2} className="w-14 h-14 text-xl border-2" mask />
      <InputOTPSlot index={3} className="w-14 h-14 text-xl border-2" mask />
    </InputOTPGroup>
  </InputOTP>
</div>
```

## Loading Table — Skeleton Rows

### Before
```jsx
{isLoading && <p>Loading...</p>}
```

### After
```jsx
{isLoading ? (
  <div className="space-y-3">
    {Array.from({ length: 5 }).map((_, i) => (
      <div key={i} className="flex items-center gap-4 animate-pulse">
        <div className="h-10 w-10 rounded-full bg-muted" />
        <div className="flex-1 space-y-2">
          <div className="h-4 w-3/4 rounded bg-muted" />
          <div className="h-3 w-1/2 rounded bg-muted" />
        </div>
        <div className="h-5 w-20 rounded bg-muted" />
      </div>
    ))}
  </div>
) : (
  <TransactionList items={transactions} />
)}
```

## Hero Section — Grain Texture Overlay

### Before
```jsx
<section className="bg-gradient-to-br from-orange-50 to-white py-28 text-center">
  <h1 className="text-7xl font-extrabold">Title</h1>
</section>
```

### After
```jsx
{/* grain-overlay class adds animated SVG noise texture via ::before pseudo-element */}
<section className="grain-overlay relative overflow-hidden bg-gradient-to-br from-orange-50 via-white to-emerald-50 dark:from-orange-950/30 dark:via-neutral-900 dark:to-emerald-950/20 py-28 text-center">
  <h1 className="relative z-10 text-5xl md:text-7xl font-extrabold tracking-tight animate-fade-in-up">
    Title
  </h1>
  <p className="relative z-10 animate-fade-in-up text-xl" style={{ animationDelay: '100ms' }}>
    Subtitle
  </p>
</section>
```

## Tabs — Neo-Brutal Active State

### Before (default shadcn)
```jsx
import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs";

<Tabs defaultValue="tab1">
  <TabsList>
    <TabsTrigger value="tab1">Tab 1</TabsTrigger>
    <TabsTrigger value="tab2">Tab 2</TabsTrigger>
  </TabsList>
  <TabsContent value="tab1">Content 1</TabsContent>
  <TabsContent value="tab2">Content 2</TabsContent>
</Tabs>
```

### After (Chipi)
```jsx
import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs";

const TAB_CLASSES = "neo-border rounded-none neo-shadow-sm neo-push px-6 py-3 font-bold text-sm uppercase tracking-wider transition-all data-[state=active]:bg-[var(--neo-fg)] data-[state=active]:text-[var(--neo-bg)] data-[state=active]:translate-x-[4px] data-[state=active]:translate-y-[4px] data-[state=active]:shadow-none data-[state=inactive]:bg-[var(--neo-bg)] data-[state=inactive]:text-[var(--neo-fg)]";

<Tabs defaultValue="tab1">
  <TabsList className="flex flex-wrap gap-3 bg-transparent">
    <TabsTrigger value="tab1" className={TAB_CLASSES}>Tab 1</TabsTrigger>
    <TabsTrigger value="tab2" className={TAB_CLASSES}>Tab 2</TabsTrigger>
  </TabsList>
  <TabsContent value="tab1">Content 1</TabsContent>
  <TabsContent value="tab2">Content 2</TabsContent>
</Tabs>
```

## Alert Banner — Actionable + Branded

### Before (default shadcn)
```jsx
import { Alert, AlertDescription } from "@/components/ui/alert";

<Alert>
  <AlertDescription>Something happened.</AlertDescription>
</Alert>
```

### After (Chipi)
```jsx
import { Alert, AlertDescription } from "@/components/ui/alert";
import { AlertTriangle, ArrowRight } from "lucide-react";

<Alert className="neo-border rounded-none bg-amber-50 dark:bg-amber-950/30 border-amber-500/40">
  <AlertTriangle className="h-4 w-4 text-amber-600" />
  <AlertDescription className="flex items-center justify-between">
    <span className="font-medium text-amber-900 dark:text-amber-200">
      Session key expires in 15 minutes
    </span>
    <button className="inline-flex items-center gap-1 text-sm font-bold text-amber-700 hover:text-amber-900 underline underline-offset-4">
      Renew <ArrowRight className="h-3 w-3" />
    </button>
  </AlertDescription>
</Alert>
```

## Success Celebration — Checkmark Draw

### Before
```jsx
{success && <p className="text-green-500 animate-pulse">Success!</p>}
```

### After
```jsx
{success && (
  <div className="flex flex-col items-center space-y-4">
    <div className="w-24 h-24 rounded-full bg-emerald-500/20 border-2 border-emerald-500 flex items-center justify-center">
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
    </div>
    <p className="text-lg font-semibold">Transaction confirmed!</p>
    <p className="text-sm text-muted-foreground">Sent $50.00 USDC to 0x1234...abcd</p>
  </div>
)}
```
