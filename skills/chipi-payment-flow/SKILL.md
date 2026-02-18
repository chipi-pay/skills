---
name: chipi-payment-flow
description: Add USDC payment acceptance to a Next.js, React, or Expo app using Chipi. Supports Chipi wallet and external wallet payments with webhook notifications. Use when user says "accept payments", "add checkout", "USDC payments", "merchant payments", "pay with crypto", or "crypto checkout".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi Payment Flow

Add USDC payment acceptance with two checkout paths: Chipi wallet (passkey/PIN) and external wallet redirect.

## Prerequisites

- Chipi providers must be set up (see `chipi-wallet-setup` skill)
- Merchant wallet address from dashboard.chipipay.com
- Chipi secret key (sk_prod_) for webhook verification

## Step 1: Verify Wallet Setup

Check that ChipiProvider and auth are configured. If not, run the `chipi-wallet-setup` skill first.

**VERIFY:** ChipiProvider wraps the app, auth provider is configured.

## Step 2: Set Merchant Wallet

Add to `.env.local`:
```env
NEXT_PUBLIC_MERCHANT_WALLET=0xYOUR_MERCHANT_WALLET_ADDRESS
```

**VERIFY:** `.env.local` has `NEXT_PUBLIC_MERCHANT_WALLET`.

## Step 3: Get Payment Component

If MCP is connected, call: `get_component_code("pay-with-crypto-button")`

This provides:
- `PayWithCryptoCard` — Main payment UI with both paths
- `PayWithChipiButton` — Pay with Chipi wallet (passkey/PIN)
- `PayWithExternalWalletButton` — Redirect to pay.chipipay.com
- `TransactionSigner` — Universal auth dialog (auto-detects passkey vs PIN)

Install shadcn dependencies:
```bash
npx shadcn@latest add button card dialog label input-otp --y
```

**VERIFY:** Payment components are in the project.

## Step 4: Add Payment Page

Create a payment page with the `PayWithCryptoCard` component:

```tsx
import { PayWithCryptoCard } from "@/components/pay-with-crypto-card";

export default function PaymentPage() {
  return (
    <PayWithCryptoCard
      amount="10.00"
      token="USDC"
      merchantWallet={process.env.NEXT_PUBLIC_MERCHANT_WALLET!}
      onSuccess={(txHash) => {
        console.log("Payment successful:", txHash);
      }}
    />
  );
}
```

Two payment paths:
1. **Chipi Wallet** — User confirms with passkey/PIN, transaction executes directly
2. **External Wallet** — User is redirected to pay.chipipay.com

**VERIFY:** Payment page renders with both payment options.

## Step 5: Configure Webhook Endpoint

Create an API route to receive payment notifications with HMAC-SHA256 verification:

```typescript
// app/api/webhooks/chipi/route.ts (Next.js)
import crypto from "crypto";

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get("chipi-signature");

  const expectedSignature = crypto
    .createHmac("sha256", process.env.CHIPI_SECRET_KEY!)
    .update(body)
    .digest("hex");

  if (signature !== expectedSignature) {
    return Response.json({ error: "Invalid signature" }, { status: 401 });
  }

  const payload = JSON.parse(body);
  // payload.event, payload.txHash, payload.amount, payload.token, payload.status
  // Update your order/subscription status

  return Response.json({ received: true });
}
```

**VERIFY:** Webhook endpoint exists and validates signatures.

## Step 6: Test

1. Open payment page
2. Test Chipi wallet payment (if wallet exists)
3. Test external wallet redirect
4. Verify webhook receives notification

**VERIFY:** Full payment flow works end-to-end.

## Key Rules

- USDC has 6 decimals — "10.00" displays as $10.00
- All transactions are gasless through Chipi
- `sk_prod_` keys are SECRET — never expose to frontend/client
- External wallet payments redirect to pay.chipipay.com
- Always verify webhook signatures with HMAC-SHA256

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Invalid signature" | Verify using sk_prod_ key, not pk_prod_ |
| "Merchant wallet not set" | Add NEXT_PUBLIC_MERCHANT_WALLET to .env.local |
| "Payment failed" | Check sender balance, verify amounts |
| "Webhook not receiving" | Ensure endpoint is publicly accessible |
