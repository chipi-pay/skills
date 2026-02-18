# Contract Interaction Patterns

## Pattern 1: Simple Call

```tsx
import { useCallAnyContract } from "@chipi-stack/nextjs";

const { mutateAsync: callContract } = useCallAnyContract();

await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  contractAddress: target,
  calls: [{ contractAddress: target, entrypoint: "my_function", calldata: ["arg1", "arg2"] }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Pattern 2: Approve + Call

When interacting with DeFi protocols that spend your tokens:

```tsx
import { useApprove, useCallAnyContract } from "@chipi-stack/nextjs";

// Step 1: Approve
const { mutateAsync: approve } = useApprove();
await approve({
  encryptKey: passkeyCredential, wallet: userWallet,
  contractAddress: usdcAddress, spender: defiAddress, amount: "100", decimals: 6,
});

// Step 2: Call protocol
const { mutateAsync: callContract } = useCallAnyContract();
await callContract({
  encryptKey: passkeyCredential, wallet: userWallet,
  contractAddress: defiAddress,
  calls: [{ contractAddress: defiAddress, entrypoint: "deposit", calldata: ["100000000", "0"] }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Pattern 3: Multi-Call (Batch)

Multiple operations in one transaction:

```tsx
await callContract({
  encryptKey: passkeyCredential, wallet: userWallet, contractAddress: target,
  calls: [
    { contractAddress: tokenA, entrypoint: "approve", calldata: [router, amountA, "0"] },
    { contractAddress: tokenB, entrypoint: "approve", calldata: [router, amountB, "0"] },
    { contractAddress: router, entrypoint: "add_liquidity", calldata: [tokenA, tokenB, amountA, "0", amountB, "0"] },
  ],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Pattern 4: Read Before Write

```tsx
import { Contract, RpcProvider } from "starknet";

// Read (free, no gas, no Chipi needed)
const provider = new RpcProvider({ nodeUrl: "https://starknet-mainnet.public.blastapi.io" });
const contract = new Contract(abi, contractAddress, provider);
const balance = await contract.balanceOf(walletAddress);

// Write (through Chipi, gasless)
await callContract({
  encryptKey: passkeyCredential, wallet: userWallet, contractAddress,
  calls: [{ contractAddress, entrypoint: "transfer", calldata: [recipient, balance.low.toString(), balance.high.toString()] }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Security Checklist

- [ ] Contract address verified on starkscan.co
- [ ] ABI reviewed
- [ ] Amounts are specific (no unlimited approvals)
- [ ] Calldata encoding matches contract expectations
- [ ] Tested with small amounts first
