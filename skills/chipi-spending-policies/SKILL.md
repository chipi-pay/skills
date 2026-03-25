---
name: chipi-spending-policies
description: Set per-token spending caps on session keys — AI agent budgets, game limits, employee wallets. On-chain enforcement via CHIPI v33 wallet contract. Use when user says "spending policy", "spending caps", "budget control", "session key limits", "maxPerCall", "maxPerWindow", or "token allowance".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Spending Policies — Budget Control for Session Keys

Set per-token spending caps on session keys. The CHIPI v33 wallet contract enforces limits automatically during transaction execution. No backend enforcement needed.

Thanks to [Omar Espejel](https://x.com/espejelomar) for proposing this feature and contributing to [SNIP-163](https://github.com/starknet-io/SNIPs/pull/163).

## When to Use

After creating and registering a session key, before the session starts executing transactions autonomously. Use cases:

- AI agent with a daily USDC budget
- Game with per-trade and daily spending caps
- Employee wallet with transaction limits
- DeFi bot with max position size per swap

## Requirements

- CHIPI v33 wallet (v29 does not have spending policy entrypoints)
- Session key registered on-chain
- `@chipi-stack/nextjs@latest` (frontend) or `@chipi-stack/backend@latest` (server)

```bash
# Frontend (Next.js / React)
npm install @chipi-stack/nextjs@latest

# Backend (Node.js)
npm install @chipi-stack/backend@latest

# Python
pip install chipi-python
```

## Backend SDK

```typescript
import { ChipiServerSDK } from "@chipi-stack/backend";

const sdk = new ChipiServerSDK({
  apiPublicKey: process.env.CHIPI_PUBLIC_KEY!,
  apiSecretKey: process.env.CHIPI_SECRET_KEY!,
});

const USDC = "0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb";

// Set policy: max 1 USDC per call, 50 USDC per day
const txHash = await sdk.sessions.setSpendingPolicy({
  encryptKey: userEncryptKey,
  wallet: userWallet,
  spendingPolicyConfig: {
    sessionPublicKey: session.publicKey,
    token: USDC,
    maxPerCall: 1_000_000n,      // 1 USDC (6 decimals)
    maxPerWindow: 50_000_000n,   // 50 USDC
    windowSeconds: 86400,        // 24 hours
  },
}, bearerToken);

// Query current spend
const policy = await sdk.sessions.getSpendingPolicy({
  walletAddress: userWallet.publicKey,
  sessionPublicKey: session.publicKey,
  token: USDC,
});
console.log(`Spent ${policy.spentInWindow} of ${policy.maxPerWindow} this window`);

// Remove policy
await sdk.sessions.removeSpendingPolicy({
  encryptKey: userEncryptKey,
  wallet: userWallet,
  sessionPublicKey: session.publicKey,
  token: USDC,
}, bearerToken);
```

## React / Next.js Hook

```tsx
import { useChipiSession } from "@chipi-stack/nextjs";

const {
  setSpendingPolicy,
  getSpendingPolicy,
  removeSpendingPolicy,
  isSettingSpendingPolicy,
} = useChipiSession({ wallet, encryptKey: pin, getBearerToken: getToken });

// Set limits after registering session
await setSpendingPolicy({
  token: USDC,
  maxPerCall: 1_000_000n,
  maxPerWindow: 50_000_000n,
  windowSeconds: 86400,
});
```

## Python

```python
from chipi_sdk import SetSpendingPolicyParams, SpendingPolicyConfig

tx_hash = sdk.sessions.set_spending_policy(
    SetSpendingPolicyParams(
        encrypt_key=user_encrypt_key,
        wallet=user_wallet,
        spending_policy_config=SpendingPolicyConfig(
            session_public_key=session.public_key,
            token=USDC,
            max_per_call=1_000_000,
            max_per_window=50_000_000,
            window_seconds=86400,
        ),
    ),
    bearer_token=bearer_token,
)
```

## Validation Rules

- Token address must not be empty
- windowSeconds must be a positive integer within u64 range
- maxPerCall and maxPerWindow must be non-negative and fit in u256
- maxPerCall cannot exceed maxPerWindow
- Wallet must be CHIPI type

## On-Chain Enforcement

The contract enforces during `__execute__()` on session-signed transactions:
- **Tracked:** transfer, approve, increase_allowance (ERC-20)
- **Per-call:** reverts if amount > maxPerCall
- **Window:** reverts if spentInWindow + amount > maxPerWindow
- **Auto-reset:** window resets when duration passes
- **Owner bypass:** owner signatures skip enforcement entirely

## When in Doubt, Ask

If you're unsure about wallet type (CHIPI vs READY), class hash version (v29 vs v33), or whether the session key is registered, ask the user before proceeding. Don't guess env vars or wallet addresses.

## Next Steps

- Combine with [x402 payments](https://chipipay.com/guides/x402-payments) for automated API monetization
- See [session keys](https://chipipay.com/guides/session-keys) for the full session lifecycle
- [Upgrade to v33](https://chipipay.com/guides/wallet-upgrades) if your wallets are on v29

## Documentation

- [Spending Policies Guide](https://docs.chipipay.com/sdk/guides/spending-policies)
- [Spending Policy Methods](https://docs.chipipay.com/sdk/backend/methods/spending-policies)
- [x402 + Session Keys](https://docs.chipipay.com/sdk/guides/x402-sessions)
