---
name: chipi-private-balances
description: Add confidential balances (STRK20 privacy pool) to a Chipi-powered app — shield/unshield USDC so only the user can see what they keep. Use when the user says "private balance", "confidential", "hide balance", "shield", "unshield", "privacy pool", or "modo privado". Self-custodial, gasless for the end user.
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi Private Balances (shield / unshield)

Let users move their own USDC into StarkWare's STRK20 confidential pool ("private balance") and back. Production-proven in Chipi's consumer wallet (Modo privado, mainnet since 2026-07-07).

**Mental model to give users:** checking/savings. Visible balance = spending; private balance = what you keep, invisible to everyone but you. Spending requires unshielding first (~2–3 min proving).

## Non-negotiables (read before writing code)

1. **Self-custody**: privacy keys derive CLIENT-SIDE from the user's owner-independent anchor (passkey encryptKey / PRF output). Cache the companion ADDRESS; NEVER persist or transmit the keys. If your design sends a key to a server, stop — that breaks the product's core promise.
2. **Every shield/unshield needs one passkey gesture.** It cannot be silent — that IS the security model. Set the expectation in copy: "You confirm with your fingerprint or Face ID — same as when you send money."
3. **Browser apps MUST set `gatewaySubmitUrl`** to a same-origin server proxy: the Starknet sequencer gateway sends no CORS headers. Forward the POST server-side to `https://alpha-mainnet.starknet.io/gateway/add_transaction`.
4. **StarkWare ToS**: capture user acceptance at the first shield (durable record). Link https://privacy.starknet.io/terms-of-service.pdf.
5. **Fees**: the pool charges a fixed 4 STRK per op. Chipi's gas tank fronts ALL STRK (your user never holds it), metered per API key. Show the cost in USD via `getShieldFee(strkUsdPrice)`.

## Step 1: Derive the privacy account (client-side)

```tsx
import { derivePrivacyAccount } from "@chipi-stack/nextjs"; // or chipi-react

const keys = await derivePrivacyAccount({
  anchorSecret,                    // 32 bytes from the passkey encryptKey/PRF
  seedDomainTag: wallet.address,   // per-wallet domain separation
  classHash: COMPANION_CLASS_HASH, // standard SNIP-6 account class
});
// persist keys.companionAddress (public); keep keys in memory only
```

**VERIFY:** same anchor → same `companionAddress` on every run (deterministic).

## Step 2: Shield

```tsx
import { useShield } from "@chipi-stack/nextjs";

const { shieldAsync } = useShield();
await shieldAsync({
  config: {
    apiPublicKey: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
    getBearerToken: () => getToken(),          // FRESH token source, not a string
    gatewaySubmitUrl: "/api/starknet-gateway", // your CORS proxy (browsers)
  },
  keys,
  token: USDC_ADDRESS,       // native USDC
  amount: 1_000_000n,        // base units (6 decimals)
  fundDeposit: (companion, amount) => transferGasless(companion, amount), // YOUR rail
  onStep: setPhase,          // "prepare" | "fund" | "deposit" | "prove" | "finish"
});
```

- `getBearerToken` is a FUNCTION on purpose: the flow takes 1–3 minutes and short-lived JWTs die mid-flight; the SDK re-fetches per request and retries 401s.
- Drive a REAL progress UI from `onStep` (proving is the long phase, ~60–90s). Never a fake timer; rotate reassuring copy during "prove" so it doesn't read as hung.
- Failure ≠ lost money: a failed shield leaves funds in the companion; the next shield sweeps them in. Say so in your error copy ("your money is safe").

**VERIFY:** the result contains a tx hash AND the SDK verified the receipt + pool event (it throws otherwise — no phantom successes).

## Step 3: Unshield

```tsx
const { unshieldAsync } = useUnshield();
await unshieldAsync({ config, keys, token: USDC_ADDRESS, amount, deliverTo: wallet.address });
```

`deliverTo` is REQUIRED: withdrawals land in the companion and the SDK delivers onward to the user's real wallet (awaited — they're waiting for their money).

## Step 4: Show the balance

```tsx
import { usePrivateBalance } from "@chipi-stack/nextjs";

// Passive display (instant, render-safe): net YOUR recorded events
const { byToken } = usePrivateBalance({ mode: "ledger", events });

// Explicit "verify on-chain" action (authoritative, needs the derived keys)
const { byToken } = usePrivateBalance({ mode: "onchain", config, keys });
```

Record every shield/unshield in YOUR store (direction, token, amount, txHash) — the chain won't cleanly surface confidential moves, so your ledger is the display source of truth.

## Failure modes you WILL hit (all handled, know the shapes)

- `gas_funding_failed_4xx`: business rejection (over the per-call cap / key not privacy-enabled) — don't retry, surface it.
- `prove 401`: stale JWT — the SDK already re-fetches + retries once; if persistent, your `getBearerToken` is returning cached tokens.
- Gateway `Block hash mismatch … stored block hash: 0`: prover base-block race — server-side concern (Chipi's prover pins head-12); report if seen.
- Dust: shields under ~1 USDC cost more in pool fees than they protect; gate or warn.
- REVERTED with `Result::unwrap failed`: masked inner revert (usually a missing approval) — the SDK pre-approves; if you bypass `execShield`, you own this.

## When in Doubt, Ask

If the user's auth provider, transfer rail (`fundDeposit`), or key-anchor source is unclear, ASK before writing code. Never invent env var names or file paths.
