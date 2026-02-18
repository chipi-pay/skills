---
name: chipi-defi-staking
description: Add VESU USDC staking and withdrawal to earn yield on StarkNet. Use when user says "stake USDC", "VESU staking", "earn yield", "DeFi staking", "withdraw staking", or "USDC yield".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi DeFi Staking

Add VESU USDC staking and withdrawal to earn yield on StarkNet.

## Prerequisites

- Chipi providers + wallet must be set up (see `chipi-wallet-setup` skill)
- User needs a Chipi wallet with USDC balance
- VESU is a lending protocol on StarkNet — deposits earn yield automatically

## Step 1: Verify Wallet Setup

Ensure ChipiProvider, auth, and wallet are configured with USDC balance.

**VERIFY:** Wallet is set up with USDC balance.

## Step 2: Create Stake Form

```tsx
import { useStakeVesuUsdc } from "@chipi-stack/nextjs"; // or chipi-react/chipi-expo

export function StakeForm({ wallet }: { wallet: any }) {
  const [amount, setAmount] = useState("");
  const { mutateAsync: stake, isPending } = useStakeVesuUsdc();

  const handleStake = async () => {
    if (Number(amount) <= 0) return;

    await stake({
      encryptKey: passkeyCredential,
      wallet,
      amount,
      bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
    });
  };

  return (
    <div>
      <h2>Stake USDC in VESU</h2>
      <input type="number" value={amount} onChange={(e) => setAmount(e.target.value)} min="0" step="0.01" />
      <button onClick={handleStake} disabled={isPending}>
        {isPending ? "Staking..." : "Stake USDC"}
      </button>
    </div>
  );
}
```

## Step 3: Create Withdraw Form

```tsx
import { useWithdrawVesuUsdc } from "@chipi-stack/nextjs";

export function WithdrawForm({ wallet }: { wallet: any }) {
  const [amount, setAmount] = useState("");
  const { mutateAsync: withdraw, isPending } = useWithdrawVesuUsdc();

  const handleWithdraw = async () => {
    if (Number(amount) <= 0) return;

    await withdraw({
      encryptKey: passkeyCredential,
      wallet,
      amount,
      bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
    });
  };

  return (
    <div>
      <h2>Withdraw USDC from VESU</h2>
      <input type="number" value={amount} onChange={(e) => setAmount(e.target.value)} min="0" step="0.01" />
      <button onClick={handleWithdraw} disabled={isPending}>
        {isPending ? "Withdrawing..." : "Withdraw USDC"}
      </button>
    </div>
  );
}
```

## Step 4: Test

1. Verify stake form validates amount > 0
2. Test staking flow with passkey confirmation
3. Test withdrawal flow
4. Verify balance changes

**VERIFY:** Both stake and withdraw work end-to-end.

## Hooks Used

- `useStakeVesuUsdc` — Stake USDC (auto-handles token approval)
- `useWithdrawVesuUsdc` — Withdraw USDC from VESU

## Key Rules

- VESU is a lending protocol — deposits earn yield automatically
- Auto-handles ERC-20 approval before staking
- Amounts must be greater than 0
- USDC has 6 decimals
- Withdrawals depend on VESU pool liquidity

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Insufficient balance" | Check USDC balance before staking |
| "Amount must be > 0" | Validate amount input |
| "Withdrawal pending" | VESU pool may have low liquidity — wait and retry |
