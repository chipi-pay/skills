---
name: chipi-custom-contracts
description: Call any StarkNet smart contract method through Chipi's gasless infrastructure using useCallAnyContract. Covers contract interactions, calldata formatting, ABI parsing, and combining with useApprove for token approvals. Use when user says "call contract", "smart contract interaction", "custom contract", "execute entrypoint", "calldata", or "useCallAnyContract".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi Custom Contract Interactions

Call any StarkNet smart contract method through Chipi's gasless infrastructure.

**Used in:** NFT minting, governance voting, token launches, custom DeFi, gaming item drops, loyalty points, social graphs, DAO tooling

## When in Doubt, Ask
If the user's project structure is unclear or doesn't match expected patterns, ASK before proceeding. Never guess at file paths, framework configuration, or environment variable names.

## Prerequisites

- Chipi wallet must be set up (see `chipi-wallet-setup` skill)
- Target contract address and ABI or entrypoint signature

## Step 1: Verify Wallet Setup

Ensure the user has a Chipi wallet. If not, run `chipi-wallet-setup` first.

## Step 2: Format Calldata

```typescript
interface Call {
  contractAddress: string;  // 0x + 64 hex
  entrypoint: string;       // Function name
  calldata: string[];       // Encoded arguments
}
```

### Common Patterns

**Simple call:**
```tsx
const calls = [{
  contractAddress: "0x049d36...",
  entrypoint: "transfer",
  calldata: [recipientAddress, amountLow, amountHigh],
}];
```

**Multi-call (batch):**
```tsx
const calls = [
  { contractAddress: tokenAddr, entrypoint: "approve", calldata: [spender, amountLow, "0"] },
  { contractAddress: protocol, entrypoint: "deposit", calldata: [amountLow, "0"] },
];
```

> **Why batch calls:** On StarkNet, you can bundle approve + swap + transfer into a single transaction. One passkey tap instead of three — a massive UX upgrade over EVM.

## Step 3: Token Approval (if needed)

```tsx
import { useApprove } from "@chipi-stack/nextjs";

const { mutateAsync: approve } = useApprove();

await approve({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  contractAddress: tokenContractAddress,
  spender: targetContractAddress,
  amount: "100",
  decimals: 6, // USDC=6, ETH/STRK=18
});
```

Use specific amounts, NOT unlimited approvals.

## Step 4: Execute Contract Call

```tsx
import { useCallAnyContract } from "@chipi-stack/nextjs";

const { mutateAsync: callContract, isPending } = useCallAnyContract();

const result = await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  contractAddress: "0x049d36...",
  calls: [{
    contractAddress: "0x049d36...",
    entrypoint: "transfer",
    calldata: [recipientAddress, amountLow, amountHigh],
  }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});

console.log("Transaction hash:", result.txHash);
```

## Security Warnings

> **Why verify first:** Calling a contract with wrong parameters will fail. Understanding the ABI before calling prevents wasted time and confusing error messages.

1. **Verify contract address** — on-chain calls are irreversible
2. **Review contract ABI** — wrong calldata can lock funds
3. **Use specific approval amounts** — never unlimited
4. **Test with small amounts first**

## Hooks Used

- `useCallAnyContract` — Execute arbitrary contract methods (gasless)
- `useApprove` — Grant token spending permission

## Calldata Encoding

- **Addresses:** 0x + 64 hex chars
- **u256 amounts:** [low, high] — for amounts < 2^128: low = amount, high = "0"
- **Example:** 1 USDC = low: "1000000", high: "0"

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Contract not found" | Verify address (0x + 64 hex) |
| "Entrypoint not found" | Check function name matches ABI |
| "Invalid calldata" | Verify encoding and argument count |
| "Insufficient balance" | Check token balance |
| "Approval needed" | Use useApprove before spending tokens |

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key contract-specific rules:
- **Contract addresses**: `font-mono text-sm` truncated to `0x1234...abcd` with `Copy`→`Check` icon swap on click. Add `toast.success("Address copied")`
- **Calldata preview**: show formatted calldata in `font-mono text-xs bg-muted p-4 rounded-lg` code block before signing. Use `--border` color for the code block border
- **Multi-call batching**: list all calls in confirmation dialog as numbered steps. Each call shows: function name, contract (truncated), and args
- **Transaction signing**: use `TransactionSigner` with step indicator (Authenticating → Signing → Done). Steps use `--accent` for active state
- **Security warning**: amber card with `AlertTriangle` icon (`h-5 w-5`) + "Verify this contract address before interacting" in `text-amber-600`
- **ABI explorer**: show entrypoints in a list with `Code2` icon. Input fields for each parameter with type labels
- **Transaction hash**: after success, show hash in `font-mono text-sm` with `ExternalLink` icon linking to block explorer (opens in new tab)
- **Responsive**: calldata preview uses horizontal scroll on mobile (`overflow-x-auto`)

## What's Next?

- **`chipi-session-keys`** — Enable repeated contract calls without re-authenticating. Approve once, execute many interactions frictionlessly.
- **`chipi-migrate-from-evm-solana`** — Port existing contract interactions from EVM or Solana to StarkNet using Chipi's gasless infrastructure.
