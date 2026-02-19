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

Use the `chipi-frontend-design` skill for full design system guidance. Key contract-specific rules:
- Contract addresses: `font-mono text-sm` truncated display with copy button
- Calldata preview: show formatted calldata before signing in a code block
- Multi-call batching: list all calls in confirmation dialog before auth
- Transaction signing: step indicator (Authenticating > Signing > Done)
- Security: show ShieldCheck icon and "Verify contract before interacting" warning
