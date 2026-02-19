# EVM to StarkNet + Chipi — Function-Level Mapping

Detailed mapping of ethers.js / viem patterns to their Chipi SDK equivalents.

## Provider & Signer

### ethers.js v6

```tsx
// BEFORE
import { ethers } from "ethers";

const provider = new ethers.BrowserProvider(window.ethereum);
const signer = await provider.getSigner();
const address = await signer.getAddress();
const balance = await provider.getBalance(address);
const network = await provider.getNetwork();
```

```tsx
// AFTER (Chipi)
import { useChipiWallet, useGetTokenBalance } from "@chipi-stack/nextjs";

const { wallet, hasWallet } = useChipiWallet();
const address = wallet?.publicKey;
const { data: balance } = useGetTokenBalance({
  walletAddress: address,
  tokenAddress: "USDC",
});
// Network is always StarkNet — no network switching needed
```

### viem

```tsx
// BEFORE
import { createWalletClient, custom } from "viem";
import { mainnet } from "viem/chains";

const client = createWalletClient({
  chain: mainnet,
  transport: custom(window.ethereum),
});
const [address] = await client.getAddresses();
```

```tsx
// AFTER (Chipi) — same as ethers.js mapping above
const { wallet } = useChipiWallet();
const address = wallet?.publicKey;
```

## Wallet Connection

```tsx
// BEFORE (MetaMask / WalletConnect)
await window.ethereum.request({ method: "eth_requestAccounts" });
// or
import { useConnect } from "wagmi";
const { connect, connectors } = useConnect();
await connect({ connector: connectors[0] });
```

```tsx
// AFTER (Chipi) — wallet creation replaces connection
import { useCreateWallet } from "@chipi-stack/nextjs";

const { mutateAsync: createWallet } = useCreateWallet();
await createWallet({
  walletType: "CHIPI",      // Self-custody wallet with session key support
  usePasskey: true,          // Biometric auth
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No extension needed. No connection popup. Just a fingerprint scan.
```

## ERC-20 Operations

### Transfer

```tsx
// BEFORE (ethers.js)
const usdc = new ethers.Contract(usdcAddress, erc20Abi, signer);
const tx = await usdc.transfer(recipient, ethers.parseUnits("10", 6));
const receipt = await tx.wait();
```

```tsx
// AFTER (Chipi)
import { useTransfer } from "@chipi-stack/nextjs";

const { mutateAsync: transfer } = useTransfer();
const result = await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: 10,  // Human-readable — SDK handles 6-decimal conversion
  tokenAddress: "USDC",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// Gasless. No gas estimation. No nonce management.
```

### Approve + Interact (multicall)

```tsx
// BEFORE (ethers.js) — 2 separate transactions, 2 gas fees
const usdc = new ethers.Contract(usdcAddress, erc20Abi, signer);
const approveTx = await usdc.approve(dexAddress, amount);
await approveTx.wait();
const dex = new ethers.Contract(dexAddress, dexAbi, signer);
const swapTx = await dex.swap(tokenIn, tokenOut, amount);
await swapTx.wait();
```

```tsx
// AFTER (Chipi) — 1 transaction, 0 gas
import { useCallAnyContract } from "@chipi-stack/nextjs";

const { mutateAsync: callContract } = useCallAnyContract();
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [
    {
      contractAddress: usdcAddress,     // approve
      entrypoint: "approve",
      calldata: [dexAddress, amountLow, "0"],
    },
    {
      contractAddress: dexAddress,       // swap
      entrypoint: "swap",
      calldata: [tokenIn, tokenOut, amountLow, "0"],
    },
  ],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

### Balance Check

```tsx
// BEFORE (ethers.js)
const usdc = new ethers.Contract(usdcAddress, erc20Abi, provider);
const balance = await usdc.balanceOf(address);
const formatted = ethers.formatUnits(balance, 6);
```

```tsx
// AFTER (Chipi)
import { useGetTokenBalance } from "@chipi-stack/nextjs";

const { data: balance } = useGetTokenBalance({
  walletAddress: address,
  tokenAddress: "USDC",
});
// Returns formatted balance — no manual decimal conversion
```

## Transaction Handling

### Send Transaction

```tsx
// BEFORE (ethers.js)
const tx = await signer.sendTransaction({
  to: recipient,
  value: ethers.parseEther("0.1"),
  gasLimit: 21000,
});
const receipt = await tx.wait();
console.log(`Tx hash: ${receipt.hash}`);
console.log(`Status: ${receipt.status === 1 ? "success" : "failed"}`);
```

```tsx
// AFTER (Chipi) — transfers are abstracted
import { useTransfer } from "@chipi-stack/nextjs";

const { mutateAsync: transfer } = useTransfer();
const result = await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: 0.1,
  tokenAddress: "ETH",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No gas limit. No gas estimation. Chipi handles everything.
```

### Event Listening

```tsx
// BEFORE (ethers.js)
const contract = new ethers.Contract(address, abi, provider);
contract.on("Transfer", (from, to, amount) => {
  console.log(`Transfer: ${from} → ${to}: ${amount}`);
});
```

```tsx
// AFTER (Chipi) — use transaction history hook
import { useGetTransactionList } from "@chipi-stack/nextjs";

const { data: transactions } = useGetTransactionList({
  walletAddress: address,
});
// Poll or refetch to detect new transactions
```

## Auth Patterns

### Sign-In with Ethereum (SIWE)

```tsx
// BEFORE
const message = `Sign in to MyApp\nNonce: ${nonce}`;
const signature = await signer.signMessage(message);
// Send to backend for verification
```

```tsx
// AFTER (Chipi) — standard auth, no wallet signing
// Use Clerk, Firebase, or Supabase for authentication
// The wallet is created AFTER auth, not used FOR auth
import { useUser } from "@clerk/nextjs";
const { user, isSignedIn } = useUser();
// User signs in with email/Google/passkey — wallet is separate
```

## Common Token Addresses (StarkNet Mainnet)

| Token | EVM Address | StarkNet Address |
|-------|-------------|-----------------|
| USDC | 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 | 0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb |
| ETH | Native / 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 | 0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7 |
| STRK | N/A | 0x04718f5a0fc34cc1af16a1cdee98ffb20c31f5cd61d6ab07201858f4287c938d |

## Key Differences to Remember

| Concept | EVM | StarkNet + Chipi |
|---------|-----|------------------|
| Address format | 0x + 40 hex chars | 0x + 64 hex chars |
| uint256 encoding | Single value | [low_128, high_128] pair |
| Gas | User pays ETH | Chipi sponsors (gasless) |
| Transaction batching | Not native (multicall contract) | Native (array of calls) |
| Account type | EOA or contract | Always a contract |
| Wallet creation | Instant (just a key pair) | On-chain deployment (Chipi handles) |
| Block time | ~12 seconds | ~15-30 seconds |
