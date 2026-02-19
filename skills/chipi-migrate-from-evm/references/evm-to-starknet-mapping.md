# EVM to StarkNet + Chipi — Quick-Reference Mapping

Concise function-level mapping of ethers.js / viem patterns to Chipi SDK equivalents.

> For deep patterns (ethers v5/v6 differences, wagmi hooks, full provider stack removal, multicall, event listening, SIWE, NPM cleanup), see `evm-code-migration-patterns.md`.

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

> **ethers.js v5 note:** v5 uses `new ethers.providers.Web3Provider(window.ethereum)` and `ethers.utils.formatEther()`. v6 uses `new ethers.BrowserProvider()` and `ethers.formatEther()`. Both map to the same Chipi hooks. See `evm-code-migration-patterns.md` for detailed v5 patterns.

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

## Common Token Addresses (StarkNet Mainnet — as of Feb 2026)

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
