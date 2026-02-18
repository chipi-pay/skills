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

## Geographic Availability

**Mexico only.** More countries coming soon. If the user is building for other markets, warn them that SKU purchasing will not work outside Mexico. Suggest wallet and payment features as alternatives that work globally.

## Prerequisites

- Chipi providers + wallet must be set up (see `chipi-wallet-setup` skill)
- Users need a Chipi wallet with USDC balance to purchase SKUs

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
