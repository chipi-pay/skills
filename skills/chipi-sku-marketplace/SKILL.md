---
name: chipi-sku-marketplace
description: Build a service marketplace using Chipi SKUs on Next.js, React, or Expo. Sell airtime, gift cards, bill pay, gaming credits with crypto. Currently available in Mexico only — more countries coming soon. Use when user says "add marketplace", "sell services", "SKU integration", "product listing", "telecom top-up", "gift cards", or "recargas".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi SKU Marketplace

Build a digital service marketplace with 14 categories: airtime, gift cards, gaming credits, and more.

**Used in:** super-apps, fintech wallets, loyalty reward redemption, mobile top-up services, gift card platforms

## Geographic Availability

**Mexico only.** More countries coming soon. If the user is building for other markets, warn them that SKU purchasing will not work outside Mexico. Suggest wallet and payment features as alternatives that work globally.

## Prerequisites

- Chipi providers + wallet must be set up (see `chipi-wallet-setup` skill)
- Users need a Chipi wallet with USDC balance to purchase SKUs

## When in Doubt, Ask
If the user's project structure is unclear or doesn't match expected patterns, ASK before proceeding. Never guess at file paths, framework configuration, or environment variable names.

## Step 1: Verify Setup

Check that ChipiProvider, auth, and wallet are configured. If not, run `chipi-wallet-setup` first.

**VERIFY:** Wallet setup is complete.

## Step 2: Get SKU Listing Component

If MCP is connected, call: `get_component_code("list-skus")`

Install dependencies:
```bash
npx shadcn@latest add pagination --y
```

**VERIFY:** `/skus` page exists with SKU grid.

> **Why categories matter:** With 14 categories and hundreds of SKUs, users need filtering to find what they need quickly. Don't show everything at once.

## Step 3: Get SKU Purchase Component

If MCP is connected, call: `get_component_code("buy-sku")`

Install dependencies:
```bash
npx shadcn@latest add pagination card dialog form input alert label input-otp button --y
```

**VERIFY:** `/skus/[id]` page exists with purchase dialog.

## Step 4: Implement SKU Listing

Use `useGetSkuList` for paginated listing:

```tsx
import { useGetSkuList } from "@chipi-stack/nextjs";

const ITEMS_PER_PAGE = 12;

export default function SkusPage() {
  const [page, setPage] = useState(1);
  const [category, setCategory] = useState<string>();

  const { data, isLoading } = useGetSkuList({
    page,
    limit: ITEMS_PER_PAGE,
    category,
    bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
  });

  return (/* Category filter + SKU grid + Pagination */);
}
```

### SKU Categories (14)

telephony, electricity, internet, streaming, gift cards, gaming, transport, toll payments, TV, mobility, government services, insurance, education, health

## Step 5: Implement SKU Purchase

```tsx
import { useGetSku, usePurchaseSku } from "@chipi-stack/nextjs";

const { data: sku } = useGetSku({ skuId, bearerToken });
const { mutateAsync: purchase } = usePurchaseSku();

await purchase({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  skuId: sku.id,
  reference: phoneNumber, // validated by SKU-specific regex
  amount: sku.price,
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Step 6: Dynamic Reference Validation

Each SKU has a specific regex pattern for the reference field (phone number, account ID, etc.). Validate the reference against the SKU's `referencePattern` before submitting.

> **Why dynamic validation:** Each SKU type requires different reference data — a phone top-up needs a phone number, an electricity payment needs a meter number. The SKU's regex pattern ensures valid data before purchasing.

## Step 7: Test

1. Navigate to `/skus` — verify grid loads with pagination
2. Filter by category — verify filtering works
3. Click a SKU — verify detail page loads
4. Test purchase flow (reference validation + wallet confirmation)

**VERIFY:** Full SKU flow works end-to-end.

## Hooks Used

- `useGetSkuList` — Paginated SKU listing
- `useGetSku` — Individual SKU details
- `usePurchaseSku` — Execute purchase with wallet signature
- `useGetSkuPurchase` — Track purchase status

## Troubleshooting

| Error | Solution |
|-------|----------|
| "No SKUs found" | SKU marketplace is Mexico only |
| "Invalid reference" | Reference doesn't match SKU's regex pattern |
| "Purchase failed" | Check wallet USDC balance |
| "SKU not available" | SKU may be temporarily out of stock |

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key marketplace-specific rules:
- **SKU cards**: glassmorphic style (`bg-white/70 dark:bg-black/50 backdrop-blur-xl border border-white/20`) with hover lift (`hover:-translate-y-0.5 hover:shadow-md transition-all duration-200`)
- **Category icons**: use Lucide React icons (`h-5 w-5`), never emoji. Map categories to icons consistently (e.g., `Phone` for telecom, `Zap` for utilities)
- **Price display**: `font-mono tabular-nums` with 2 decimal places (`$10.00`), always show `$` prefix. Use `--accent` color for price highlight
- **Purchase confirmation**: show SKU name, price (`font-mono tabular-nums`), and reference field. Reference input gets `aria-label` describing expected format
- **Loading**: skeleton card grid (`animate-pulse bg-muted rounded-xl h-48`) during fetch, not spinners. Show 6 skeleton cards in `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`
- **Pagination**: use shadcn Pagination component, show page numbers in `tabular-nums`
- **Empty state**: "No products available" with `ShoppingBag` icon, not blank page
- **Responsive**: single-column card grid on mobile, 2 columns at `md:`, 3 at `lg:`

## What's Next?

- **`chipi-payment-flow`** — Accept USDC payments from customers with merchant checkout and webhook notifications.
- **`chipi-wallet-setup`** — If not already done, set up wallet creation and balance display for your users.
