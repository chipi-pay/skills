---
name: chipi-monetize-api
description: Earn USDC from every API call with x402 + session keys + spending policies. Server middleware returns 402, clients pay stablecoins on StarkNet. Use when user says "monetize API", "API payments", "pay per request", "x402 server", "earn from API", "API revenue", or "charge per call".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Monetize Your API with Stablecoin Micropayments

Earn USDC from every API call. Servers return HTTP 402, clients pay with stablecoins on StarkNet, requests are fulfilled automatically. No subscriptions, no billing dashboards, no invoices. AI agents pay per request via session keys.

## When to Use

- You have an API endpoint and want to charge per request in USDC
- AI agents need to consume your API autonomously
- You want revenue without Stripe, billing pages, or subscription management
- You're building data feeds, AI inference, premium content, or compute services

## How It Works

1. Client sends request to protected endpoint
2. Server returns `402 Payment Required` with price and recipient
3. Client signs USDC payment on StarkNet (session key signs automatically)
4. Server verifies payment via x402 facilitator and fulfills request

No signups. No API keys for billing. Just HTTP with built-in payments.

## Server Setup (TypeScript / Express)

```bash
npm install @chipi-stack/backend@latest express
```

```typescript
import express from "express";
import { x402Middleware } from "@chipi-stack/backend";

const app = express();
app.use(express.json());

app.use("/api/inference", x402Middleware({
  amount: "0.003",                            // USDC per request
  recipient: process.env.MERCHANT_WALLET!,    // your StarkNet wallet
  facilitatorUrl: "https://x402.chipipay.com",
  network: "starknet-mainnet",
  asset: "USDC",
}));

app.post("/api/inference", async (req, res) => {
  const result = await runInference(req.body.prompt);
  res.json(result);
});

app.listen(3000);
```

## Server Setup (Python / FastAPI)

```bash
pip install chipi-python fastapi uvicorn
# chipi-python is the Python SDK (import as chipi_sdk)
```

```python
from fastapi import FastAPI, Depends
from chipi_sdk import x402_middleware

app = FastAPI()

paywall = x402_middleware(
    amount="0.003",
    recipient="0xYOUR_WALLET",
    facilitator_url="https://x402.chipipay.com",
    network="starknet-mainnet",
    asset="USDC",
)

@app.post("/api/inference", dependencies=[Depends(paywall)])
async def inference(prompt: str):
    result = await run_inference(prompt)
    return {"result": result}
```

## Client: AI Agent with Budget

```typescript
import { ChipiServerSDK } from "@chipi-stack/backend";

const sdk = new ChipiServerSDK({
  apiPublicKey: process.env.CHIPI_PUBLIC_KEY!,
  apiSecretKey: process.env.CHIPI_SECRET_KEY!,
});

const USDC = "0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb";

// 1. Create + register session
const session = sdk.sessions.createSessionKey({ encryptKey, durationSeconds: 86400 });
await sdk.sessions.addSessionKeyToContract({
  encryptKey, wallet: agentWallet,
  sessionConfig: { sessionPublicKey: session.publicKey, validUntil: session.validUntil, maxCalls: 10000, allowedEntrypoints: ["transfer"] },
}, bearerToken);

// 2. Set spending cap: $0.01/call, $50/day
await sdk.sessions.setSpendingPolicy({
  encryptKey, wallet: agentWallet,
  spendingPolicyConfig: { sessionPublicKey: session.publicKey, token: USDC, maxPerCall: 10_000n, maxPerWindow: 50_000_000n, windowSeconds: 86400 },
}, bearerToken);

// 3. Agent calls API — payments are automatic via session key
```

## Pricing Guide

| Model | Price | Use Case |
|-------|-------|----------|
| Per-request | $0.001 - $0.01 | Data APIs, price feeds |
| Per-inference | $0.003 - $0.05 | AI models, LLM calls |
| Per-compute | $0.01 - $0.10 | Heavy processing |
| Free tier | $0 | Health checks, discovery |

## Three Layers of Protection

1. **Client-side:** `maxPaymentAmount` on X402Client (early rejection, saves gas)
2. **Session limit:** `maxCalls` on session key (total transaction count)
3. **Spending policy:** `maxPerCall` + `maxPerWindow` (USDC amount, enforced on-chain)

## When in Doubt, Ask

If you're unsure about the merchant wallet address, facilitator URL, pricing model, or whether to use session keys, ask the user before proceeding. Don't guess env vars or wallet addresses.

## Documentation

- [x402 Protocol](https://docs.chipipay.com/sdk/guides/x402-introduction)
- [x402 Server Setup](https://docs.chipipay.com/sdk/guides/x402-server)
- [Spending Policies](https://docs.chipipay.com/sdk/guides/spending-policies)
- [x402 Facilitator](https://x402.chipipay.com) — live on StarkNet mainnet
