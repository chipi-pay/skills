# Anti-Slop Fintech UX Guardrails

Rules to prevent generic AI-generated UI patterns in financial applications.

## Generic Pattern Blocklist

Patterns that scream "AI-generated". NEVER use any of these:

- **NEVER** purple-to-blue gradient from top-left to bottom-right (`bg-gradient-to-br from-purple-500 to-blue-500`)
- **NEVER** `rounded-full` avatar with gradient border ring (`ring-2 ring-gradient`)
- **NEVER** 3-column card grid with `rounded-3xl` + `shadow-2xl` + identical padding — the default "features section" slop
- **NEVER** hero with centered `text-6xl` + `text-gray-500` subtitle + single purple CTA button
- **NEVER** `bg-white dark:bg-gray-900` as your only color tokens — commit to a real palette
- **NEVER** generic "Get Started" / "Learn More" / "Sign Up Free" button text without context
- **NEVER** decorative blob SVGs (`absolute -z-10 blur-3xl opacity-30`) as background interest — use grain texture or halftone patterns instead
- **NEVER** `animate-bounce` on scroll indicators or CTAs — use `animate-fade-in-up` with staggered delays
- **NEVER** icon + title + description card repeated 3-6 times in a grid as your only layout pattern

## Money Display

### Rules
1. **Always 2 decimal places**: `$10.00` not `$10` or `$10.0`
2. **Always `tabular-nums font-mono`**: digit widths must be consistent for scanning
3. **Always prefix with `$`**: never display raw numbers for currency
4. **Thousands separator for > $999**: `$1,234.56`
5. **Right-align** amounts in tables and lists
6. **Negative amounts**: use red text + minus sign, never parentheses: `-$50.00`

### Code Pattern
```tsx
function formatUSDC(amount: number): string {
  return `$${amount.toLocaleString('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  })}`;
}

<span className="font-mono tabular-nums">{formatUSDC(balance)}</span>
```

## Transaction States

Every transaction must show its current state. Never leave the user wondering "did it work?"

### Required States
| State | UI | Duration |
|-------|-----|----------|
| Idle | Form visible, CTA enabled | — |
| Authenticating | Biometric/PIN dialog, spinner | 1-5s |
| Signing | "Signing transaction..." with spinner | 2-10s |
| Pending | "Transaction submitted" with hash link | 15-30s |
| Confirmed | Green checkmark, celebration | Show 3s then dismiss |
| Failed | Red alert, actionable error message | Until dismissed |

### Step Indicator Pattern
```tsx
type TxStep = 'authenticate' | 'sign' | 'pending' | 'done';

const stepLabels: Record<TxStep, string> = {
  authenticate: 'Authenticating',
  sign: 'Signing',
  pending: 'Confirming',
  done: 'Done',
};
```

## Error Messages

### Bad (generic)
- "Error"
- "Something went wrong"
- "Transaction failed"

### Good (actionable)
- "Transfer failed — recipient address not found on StarkNet"
- "Insufficient balance: you have $4.50 but tried to send $10.00"
- "PIN verification failed — please re-enter your 4-digit PIN"
- "Passkey authentication cancelled — tap Retry to try again"

### Pattern
```tsx
function getTransferErrorMessage(error: Error): string {
  if (error.message.includes('insufficient')) {
    return `Insufficient balance. Check your USDC balance and try a smaller amount.`;
  }
  if (error.message.includes('invalid address')) {
    return `Invalid recipient address. StarkNet addresses start with 0x followed by 64 hex characters.`;
  }
  return `Transfer failed: ${error.message}. Please try again.`;
}
```

## Security Indicators

### Required Visual Cues
- **Encryption**: show ShieldCheck icon + "Encrypted" label near sensitive operations
- **Passkey**: show Fingerprint icon when biometric auth is active
- **PIN entry**: mask all digits, show dots not numbers
- **Wallet address**: always truncate to `0x1234...abcd` (6 + 4), full address on copy only
- **Transaction signing**: show what's being signed (recipient, amount) before auth prompt

### Trust Hierarchy
```
Most trusted (green border/icon):
  ✓ Passkey — "Hardware-backed, 256-bit encryption"

Adequate (default border):
  ✓ PIN — "4-digit encryption"

Warning (amber):
  ⚠ Session key — "Auto-signs for 6 hours"
```

## Loading States

### Rules
1. **Data regions > 100px**: use skeleton, not spinner
2. **Button actions**: show inline spinner + disabled state
3. **Page transitions**: show nothing (instant nav) or full skeleton
4. **Never**: empty white space while loading

### Skeleton Pattern
```tsx
function BalanceSkeleton() {
  return (
    <div className="animate-pulse space-y-2">
      <div className="h-4 w-20 rounded bg-muted" />
      <div className="h-12 w-36 rounded bg-muted" />
    </div>
  );
}
```

## Confirmation Patterns

### When to Require Explicit Confirmation
- Sending funds (always)
- Staking/unstaking (always)
- Revoking session keys (always)
- Changing auth method (always)
- Purchasing SKUs (always)

### Confirmation Dialog Structure
```
┌─────────────────────────┐
│  ⚠ Confirm Transfer     │
│                         │
│  Send $50.00 USDC to   │
│  0x1234...abcd          │
│                         │
│  Network fee: gasless   │
│                         │
│  [Cancel]    [Confirm]  │
└─────────────────────────┘
```

## Accessibility

1. **Touch targets**: minimum 44x44px on mobile
2. **Color contrast**: 4.5:1 for text, 3:1 for large text
3. **Icon buttons**: always include `aria-label`
4. **Form fields**: always include `<label>` or `aria-label`
5. **Status changes**: use `aria-live="polite"` for dynamic updates
6. **Focus management**: trap focus in dialogs, return focus on close

## Responsive Rules

1. **Mobile-first**: default styles for 375px, enhance with `md:` and `lg:`
2. **Hero text**: `text-5xl md:text-7xl` — never overflow on mobile
3. **Cards**: single column on mobile, `md:grid-cols-2` on tablet+
4. **Dialogs**: full-width minus padding on mobile (`max-w-[calc(100%-2rem)]`)
5. **Tables**: horizontal scroll wrapper on mobile
6. **Balance display**: `text-4xl md:text-5xl` — scale down for mobile
