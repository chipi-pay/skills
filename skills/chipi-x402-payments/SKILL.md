---
name: chipi-x402-payments
description: Add HTTP-native pay-per-request micropayments to any API using x402 protocol. Servers return 402 Payment Required, clients pay with USDC on StarkNet, requests are fulfilled automatically. Use when user says "x402", "pay per request", "API micropayments", "402 payment required", "monetize API", "paywall API", or "HTTP payments".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi x402 Pay-Per-Request Payments

Add HTTP-native micropayments to any API endpoint. Instead of API keys, subscriptions, or billing dashboards, payments happen inline with every HTTP request using the x402 protocol.

**Used in:** premium APIs, AI inference endpoints, data feeds, content paywalls, streaming services, real-time data, SaaS usage-based billing

## How x402 Works

1. Client sends a request to a protected endpoint
2. Server returns `402 Payment Required` with price, asset, and recipient
3. Client signs a USDC payment on StarkNet and retries with an `X-PAYMENT` header
4. Server verifies the payment via a facilitator and fulfills the request

No signups. No invoices. No billing disputes. Just HTTP requests with built-in payments.

## When in Doubt, Ask
If the user's project structure is unclear or doesn't match expected patterns, ASK before proceeding. Never guess at file paths, framework configuration, or environment variable names.

## Step 1: Choose Your Side

x402 has two sides. Determine what the user needs:

- **Client-side** — Calling a paid API (React/Next.js app consuming a 402-gated endpoint)
- **Server-side** — Protecting an API with payments (Express, Next.js API route, or FastAPI)
- **Both** — Full end-to-end setup

## Step 2: Client-Side Setup (React / Next.js)

### 2a. Install

```bash
npm install @chipi-stack/nextjs@latest
```

### 2b. Use the Hook

```tsx
import { useX402Payment } from "@chipi-stack/nextjs";

function PremiumContent() {
  const { payFetch, totalSpent, isPaying } = useX402Payment({
    wallet: userWallet,
    encryptKey: passkeyCredential,
    bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
    maxAmount: "1.00", // Safety cap per request in USDC
  });

  const loadData = async () => {
    // If the server returns 402, the hook pays and retries automatically
    const response = await payFetch("https://api.example.com/premium-data");
    const data = await response.json();
  };

  return (
    <button onClick={loadData} disabled={isPaying}>
      {isPaying ? "Processing payment..." : "Load Premium Data"}
    </button>
  );
}
```

### Hook Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| wallet | object | Yes | Chipi wallet object |
| encryptKey | string | Yes | PIN or passkey-derived key |
| bearerToken | string | Yes | Chipi API key (`pk_prod_`) |
| facilitatorUrl | string | No | Default: `https://x402.org/facilitator` |
| maxAmount | string | No | Max USDC per request (safety cap). Default: `"1.00"` |

### Hook Returns

| Field | Type | Description |
|-------|------|-------------|
| payFetch | function | Drop-in `fetch()` replacement that handles 402 automatically |
| lastPayment | object \| null | `{ amount, recipient, txHash }` of last payment |
| totalSpent | string | Cumulative USDC spent this session |
| isPaying | boolean | Payment in progress |

## Step 3: Server-Side Setup

### Option A: Express Middleware

```bash
npm install @chipi-stack/x402
```

```typescript
import express from "express";
import { x402Middleware } from "@chipi-stack/x402";

const app = express();

// Protect an endpoint with x402
app.use("/api/premium", x402Middleware({
  amount: "0.10",                              // USDC per request
  recipient: process.env.MERCHANT_WALLET!,     // Your StarkNet wallet
  facilitatorUrl: "https://x402.org/facilitator",
  network: "starknet-mainnet",
  asset: "USDC",
}));

app.get("/api/premium/data", (req, res) => {
  // Only reached after payment is verified and settled
  res.json({ data: "premium content" });
});
```

### Option B: Next.js API Route

```typescript
import { x402Middleware } from "@chipi-stack/x402";

const paywall = x402Middleware({
  amount: "0.05",
  recipient: process.env.MERCHANT_WALLET!,
  facilitatorUrl: "https://x402.org/facilitator",
  network: "starknet-mainnet",
  asset: "USDC",
});

export async function GET(req: Request) {
  const paymentResult = await paywall(req);
  if (paymentResult) return paymentResult; // Returns 402 if unpaid
  return Response.json({ data: "premium content" });
}
```

### Option C: FastAPI (Python)

```bash
pip install chipi-x402
```

```python
import os
from fastapi import FastAPI
from chipi_x402 import x402_middleware

app = FastAPI()
app.add_middleware(
    x402_middleware,
    amount="0.10",
    recipient=os.environ["MERCHANT_WALLET"],
    facilitator_url="https://x402.org/facilitator",
    network="starknet-mainnet",
    asset="USDC",
)

@app.get("/api/premium")
async def premium():
    return {"data": "premium content"}
```

### Middleware Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| amount | string | Yes | USDC price per request (e.g. `"0.10"`) |
| recipient | string | Yes | Your StarkNet wallet address |
| facilitatorUrl | string | Yes | `https://x402.org/facilitator` |
| network | string | Yes | `starknet-mainnet` |
| asset | string | Yes | `USDC` |

## Step 4: Manual Verification (Advanced)

For custom server logic, use `X402Facilitator` directly:

```typescript
import { X402Facilitator } from "@chipi-stack/x402";

const facilitator = new X402Facilitator({
  rpcUrl: "https://starknet-mainnet.infura.io/v3/YOUR_KEY",
  usdcAddress: "0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb",
});

// In your route handler:
const paymentHeader = request.headers.get("X-PAYMENT");

if (!paymentHeader) {
  return Response.json({
    price: "0.10",
    asset: "USDC",
    network: "starknet-mainnet",
    recipient: process.env.MERCHANT_WALLET,
    facilitatorUrl: "https://x402.org/facilitator",
  }, { status: 402 });
}

const result = await facilitator.verify({
  paymentHeader,
  expectedAmount: "0.10",
  expectedRecipient: process.env.MERCHANT_WALLET!,
});

if (result.valid) {
  const txHash = await facilitator.settle(result.payment);
  // Fulfill the request
}
```

## Step 5: SNIP-12 Typed Data Signing (Advanced)

For building and signing x402 payment data manually using SNIP-12 (StarkNet's EIP-712 equivalent):

```typescript
import { buildX402TypedData } from "@chipi-stack/core";
import { signTypedData } from "@chipi-stack/core";

// Build the payment typed data
const typedData = buildX402TypedData({
  recipient: merchantAddress,
  amount: "0.10",         // Human-readable USDC
  nonce: crypto.randomUUID(),
  expiry: 300,            // Seconds until payment expires
});

// Sign with the wallet's private key
const signature = await signTypedData({
  encryptedPrivateKey: wallet.encryptedPrivateKey,
  encryptKey: passkeyCredential,
  typedData,
});

// Base64-encode signature and send as X-PAYMENT header
```

## Step 6: Combine with Session Keys

Eliminate the passkey prompt on every request. After a one-time session registration, all subsequent x402 payments are signed automatically.

```tsx
import { useX402Payment } from "@chipi-stack/nextjs";
import { useChipiSession } from "@chipi-stack/nextjs";

function AutomatedPremiumAPI() {
  const { hasActiveSession, registerSession } = useChipiSession();
  const { payFetch } = useX402Payment({
    wallet: userWallet,
    encryptKey: passkeyCredential,
    bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
  });

  const loadStream = async () => {
    // After session registration, payFetch signs payments
    // with the session key — no passkey prompt per request
    const response = await payFetch("https://api.example.com/stream");
    const data = await response.json();
  };

  return <button onClick={loadStream}>Load Stream</button>;
}
```

This is ideal for streaming APIs, real-time data feeds, or any endpoint called frequently.

## Step 7: Environment Variables

**Client-side (.env.local):**
```bash
NEXT_PUBLIC_CHIPI_API_KEY=pk_prod_YOUR_KEY
```

**Server-side (.env):**
```bash
MERCHANT_WALLET=0xYOUR_STARKNET_WALLET_ADDRESS
```

## Step 8: Test

1. Start the server with x402 middleware on an endpoint
2. Call the endpoint without payment — verify 402 response with payment details
3. Call with `payFetch` — verify automatic payment and 200 response
4. Check `totalSpent` reflects the charge
5. If using session keys, verify no auth prompt after registration

## Key Rules

- Payments are in USDC on StarkNet mainnet (6 decimals)
- The facilitator at `https://x402.org` handles verification and settlement
- All transactions are gasless for the payer
- Each payment has a unique nonce for replay protection
- `maxAmount` on the client prevents unexpectedly large charges
- USDC address: `0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb`

## Troubleshooting

| Error | Solution |
|-------|----------|
| "402 Payment Required but no X-PAYMENT header" | Client isn't handling the 402 flow — use `payFetch` instead of `fetch` |
| "Payment amount exceeds maximum" | Lower the server's `amount` or raise the client's `maxAmount` |
| "Unsupported network/scheme/asset" | Verify both client and server use `starknet-mainnet` and `USDC` |
| "Nonce already used (replay)" | Each payment needs a unique nonce — check for duplicate requests |
| "Facilitator verification failed" | Ensure `facilitatorUrl` matches on client and server |
| "Insufficient USDC balance" | Payer wallet needs enough USDC for the request amount |

## What's Next?

- **`chipi-session-keys`** — Combine with session keys for automated payments without per-request auth.
- **`chipi-custom-contracts`** — Gate custom contract calls behind x402 payments.
