---
name: chipi-wallet-setup
description: End-to-end Chipi StarkNet wallet integration for Next.js, React, or Expo apps. Sets up ChipiProvider, auth, wallet creation with passkey (biometric), balance display, and USDC transfers. Defaults to passkey auth — warn about PIN security risks if user requests PIN. Use when user says "set up Chipi wallet", "integrate Chipi", "add wallet to my app", "create Starknet wallet", or "Chipi wallet integration".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi Wallet Setup

Set up a complete Chipi wallet integration with passkey authentication, balance display, and USDC transfers.

**Used in:** payment apps, gaming wallets, social tipping, loyalty programs, NFT marketplaces, DeFi dashboards, super-apps

## When in Doubt, Ask
If the user's project structure is unclear or doesn't match expected patterns, ASK before proceeding. Never guess at file paths, framework configuration, or environment variable names.

## Prerequisites

- A Next.js, React (Vite/CRA), or Expo project
- An auth provider (Clerk recommended, also supports Firebase, Supabase, Better Auth)
- A Chipi API key from dashboard.chipipay.com

## Step 1: Detect Framework & Install SDK

Detect the user's framework from their project files:
- `next.config.*` -> Next.js -> `npm install @chipi-stack/nextjs`
- `vite.config.*` -> React -> `npm install @chipi-stack/chipi-react`
- `app.json` or `expo` in package.json -> Expo -> `npx expo install @chipi-stack/chipi-expo`

**VERIFY:** package.json contains the correct `@chipi-stack/*` package.

## Step 2: Install Auth Provider

Recommended: Clerk
```bash
npm install @clerk/nextjs
```

Alternatives: Firebase (`firebase`), Supabase (`@supabase/supabase-js`), Better Auth (`better-auth`)

**VERIFY:** Auth provider package is in package.json.

## Step 3: Configure Environment Variables

Create `.env.local`:
```env
NEXT_PUBLIC_CHIPI_API_KEY=pk_prod_YOUR_KEY
CHIPI_SECRET_KEY=sk_prod_YOUR_KEY
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_YOUR_CLERK_KEY
CLERK_SECRET_KEY=sk_YOUR_CLERK_KEY
```

**VERIFY:** `.env.local` exists with all required keys.

## Step 4: Install Providers

If MCP is connected, call: `get_component_code("chipi-providers")`

Otherwise, wrap your app's layout/root with:
1. Auth provider (ClerkProvider, FirebaseProvider, etc.)
2. ChipiProvider from the SDK

For Next.js, add to `app/layout.tsx`. For React, add to `main.tsx`. For Expo, add to `App.tsx`.

**VERIFY:** Root layout has ChipiProvider wrapping the app.

> **Why this matters:** The provider initializes Chipi's SDK context so all child components can use wallet hooks. Without it, hooks like useCreateWallet won't work.

## Step 5: Add Wallet Creation

If MCP is connected, call: `get_component_code("create-wallet-dialog")`

Install shadcn dependencies:
```bash
npx shadcn@latest add button dialog form input-otp alert --y
```

### CRITICAL: Authentication Defaults

**ALWAYS default to passkey/biometric authentication:**
- Set `usePasskey: true` in `useCreateWallet`
- Set `walletType: "CHIPI"` (enables session keys)

```tsx
import { useCreateWallet } from "@chipi-stack/nextjs"; // or chipi-react/chipi-expo

const { mutateAsync: createWallet, isPending } = useCreateWallet();

await createWallet({
  encryptKey: passkeyCredential,
  externalUserId: user.id,
  chain: "STARKNET",
  walletType: "CHIPI",  // DEFAULT — Chipi's own account contract, supports session keys
  usePasskey: true,       // DEFAULT — passkey/biometric auth (RECOMMENDED)
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

### If User Requests PIN Authentication

**WARN the user:**
- PINs are 4 digits only — very limited entropy
- Vulnerable to shoulder-surfing and brute force
- Must be entered on every transaction
- Recommend passkey instead for stronger security and better UX

If they insist on PIN, proceed but suggest future migration via the `chipi-pin-to-passkey-migration` skill, which uses the `migrate-to-passkey-dialog` component:
```
Call get_component_code("migrate-to-passkey-dialog")
```

### Wallet Types

| Type | Description | Session Keys |
|------|-------------|-------------|
| **CHIPI** (default) | Chipi's own account contract — **always use this** | Yes |
| READY | Argent X compatible | No |

Only use `"READY"` if the user explicitly needs Argent X wallet app compatibility.

**VERIFY:** Wallet creation component exists with CHIPI + passkey defaults.

> **Why CHIPI wallet type:** This is Chipi's own account contract that supports session keys, meaning users won't need to re-authenticate every transaction. It's the most feature-rich option.

## Step 6: Add Wallet Summary

If MCP is connected, call: `get_component_code("wallet-summary")`

Install dependencies:
```bash
npx shadcn@latest add button card --y
```

The wallet summary displays:
- Wallet public key (truncated with copy button)
- USDC balance via `useGetTokenBalance`

**VERIFY:** Wallet summary component exists.

> **Why show balance:** Users need to see their wallet address (to receive funds) and current balance. This builds trust and confirms the wallet is working.

## Step 7: Add USDC Transfer

If MCP is connected, call: `get_component_code("send-usdc-dialog")`

Install dependencies:
```bash
npx shadcn@latest add button dialog form input label input-otp --y
```

Uses `useTransfer` hook for gasless USDC transfers.

**VERIFY:** Send USDC dialog component exists.

## Step 8: Wire Wallet Page

Create a wallet page using `useChipiWallet` (all-in-one hook):

```tsx
import { useChipiWallet } from "@chipi-stack/nextjs";

export default function WalletPage() {
  const {
    wallet, hasWallet, formattedBalance,
    createWallet, isCreating, isLoadingWallet
  } = useChipiWallet({
    externalUserId: user.id,
    getBearerToken: async () => process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
    defaultToken: "USDC",
  });

  if (isLoadingWallet) return <Loading />;
  if (!hasWallet) return <CreateWalletDialog />;

  return (
    <>
      <WalletSummary wallet={wallet} balance={formattedBalance} />
      <SendUsdcDialog wallet={wallet} />
    </>
  );
}
```

**VERIFY:** Wallet page conditionally renders create or summary+send.

## Step 9: Verification Checklist

- [ ] Correct SDK package installed for the framework
- [ ] Auth provider configured and working
- [ ] Environment variables set
- [ ] ChipiProvider wraps the app
- [ ] Wallet creation defaults to CHIPI + passkey
- [ ] Wallet summary shows public key + USDC balance
- [ ] USDC transfer works
- [ ] Dev server runs without errors

## Hooks Used

- `useCreateWallet` — Deploy wallet (CHIPI + passkey default)
- `useChipiWallet` — All-in-one wallet management
- `useGetWallet` — Fetch wallet by user ID
- `useTransfer` — Transfer USDC/ETH/STRK
- `useGetTokenBalance` — Fetch on-chain balance
- `migrate-to-passkey-dialog` — Component to convert PIN wallet to passkey (see `chipi-pin-to-passkey-migration` skill)

## Troubleshooting

| Error | Solution |
|-------|----------|
| "User not authenticated" | Ensure auth provider is configured and user is signed in |
| "Wallet not found" | User hasn't created a wallet yet — show create dialog |
| "Transaction fails" | Check balance, verify recipient address format (0x + 64 hex) |
| "Session keys not working" | Verify wallet is CHIPI type (not READY) |
| "PIN security warning" | Recommend passkey migration via `migrate-to-passkey-dialog` component |

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key wallet-specific rules:
- **Trust indicators**: `ShieldCheck` icon (lucide-react, `h-5 w-5`) near encryption steps. Passkey: emerald border + "Hardware-backed, 256-bit encryption". PIN: default border + "4-digit encryption"
- **Balance display**: `text-4xl md:text-5xl font-extrabold font-mono tabular-nums` — always 2 decimal places, use `--accent` color for the `$` prefix
- **Wallet address**: truncate to `0x1234...abcd` in `font-mono text-sm`, copy-to-clipboard with `Copy`→`Check` icon swap (1.5s) and `toast.success("Copied")`
- **Wallet creation success**: SVG checkmark-draw animation (`animate-checkmark` + `animate-scale-bounce`), never `animate-pulse`
- **Step progress**: use `StepProgress` component with steps `["Detect", "Choose", "Secure", "Create"]`
- **Loading**: skeleton placeholder (`animate-pulse bg-muted rounded-lg`) for balance, not spinner
- **Empty state**: when no wallet exists, show CTA card with `Fingerprint` icon + "Create your wallet" in `text-lg font-semibold`
- **Responsive**: dialog uses `max-w-[calc(100vw-2rem)] sm:max-w-md`

## What's Next?

- **`chipi-payment-flow`** — Accept USDC payments from customers with merchant checkout and webhook notifications.
- **`chipi-session-keys`** — Enable background transactions so users don't need to re-authenticate every action.
- **`chipi-defi-staking`** — Let users stake their USDC to earn yield directly from your app.
