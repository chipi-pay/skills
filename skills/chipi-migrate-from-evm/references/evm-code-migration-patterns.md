# EVM Code Migration Patterns

Expanded code-level mappings from ethers.js (v5 & v6), viem, wagmi, and EVM provider stacks to Chipi SDK equivalents.

> For quick-reference function mappings, see `evm-to-starknet-mapping.md`. This file covers **deep patterns** including full provider stack removal, multicall, event listening, and auth.

## ethers.js v5 → Chipi

### Web3Provider & Signer

```tsx
// BEFORE (ethers.js v5)
import { ethers } from "ethers";

const provider = new ethers.providers.Web3Provider(window.ethereum);
await provider.send("eth_requestAccounts", []);
const signer = provider.getSigner();
const address = await signer.getAddress();
const balance = await provider.getBalance(address);
const formatted = ethers.utils.formatEther(balance);
```

```tsx
// AFTER (Chipi)
import { useChipiWallet, useGetTokenBalance } from "@chipi-stack/nextjs";

const { wallet, hasWallet } = useChipiWallet();
const address = wallet?.publicKey;
const { data: balance } = useGetTokenBalance({
  walletAddress: address,
  tokenAddress: "ETH",
});
// No provider. No signer. No window.ethereum.
```

### BigNumber Math

```tsx
// BEFORE (ethers.js v5 — BigNumber)
import { ethers } from "ethers";

const amount = ethers.BigNumber.from("1000000");
const doubled = amount.mul(2);
const formatted = ethers.utils.formatUnits(amount, 6);
const parsed = ethers.utils.parseUnits("10", 6);
```

```tsx
// AFTER (Chipi)
// No BigNumber needed. Pass human-readable amounts.
// SDK handles decimal conversion internally.
await transfer({
  amount: 10, // Human-readable — SDK converts to raw units
  tokenAddress: "USDC",
  // ...
});
```

### Contract Interaction with Filters

```tsx
// BEFORE (ethers.js v5 — contract + event filters)
const contract = new ethers.Contract(address, abi, signer);
const tx = await contract.swap(tokenIn, tokenOut, amountIn, minOut);
await tx.wait();

// Event listening with filters
const filter = contract.filters.Swap(null, wallet.address);
contract.on(filter, (sender, recipient, amountIn, amountOut) => {
  console.log(`Swap: ${amountIn} → ${amountOut}`);
});
```

```tsx
// AFTER (Chipi)
import { useCallAnyContract, useGetTransactionList } from "@chipi-stack/nextjs";

const { mutateAsync: callContract } = useCallAnyContract();
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [{
    contractAddress: "0x_STARKNET_DEX",
    entrypoint: "swap",
    calldata: [tokenIn, tokenOut, amountInLow, "0", minOutLow, "0"],
  }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});

// Event listening → transaction history polling
const { data: txs } = useGetTransactionList({ walletAddress: address });
```

## ethers.js v6 → Chipi

### BrowserProvider (replaces Web3Provider)

```tsx
// BEFORE (ethers.js v6)
import { ethers } from "ethers";

const provider = new ethers.BrowserProvider(window.ethereum);
const signer = await provider.getSigner();
const address = await signer.getAddress();
const balance = await provider.getBalance(address);
const formatted = ethers.formatEther(balance); // v6: top-level function
```

```tsx
// AFTER (Chipi) — same as v5 mapping
const { wallet } = useChipiWallet();
const address = wallet?.publicKey;
```

### Native BigInt (replaces BigNumber)

```tsx
// BEFORE (ethers.js v6 — native bigint)
const amount = 1000000n; // native BigInt
const formatted = ethers.formatUnits(amount, 6);
const parsed = ethers.parseUnits("10", 6); // returns bigint
const etherValue = ethers.parseEther("0.1"); // returns bigint
```

```tsx
// AFTER (Chipi)
// No bigint math needed on frontend.
// Pass human-readable amounts. SDK handles everything.
await transfer({ amount: 10, tokenAddress: "USDC", /* ... */ });
await transfer({ amount: 0.1, tokenAddress: "ETH", /* ... */ });
```

### parseUnits / formatUnits Changes

```tsx
// v5: ethers.utils.parseUnits("10", 6) → BigNumber
// v6: ethers.parseUnits("10", 6) → bigint
// Chipi: amount: 10 (human-readable, no parsing needed)
```

## viem Deep Patterns

### publicClient.readContract

```tsx
// BEFORE (viem)
import { createPublicClient, http } from "viem";
import { mainnet } from "viem/chains";

const client = createPublicClient({ chain: mainnet, transport: http() });

const balance = await client.readContract({
  address: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  abi: erc20Abi,
  functionName: "balanceOf",
  args: [userAddress],
});
```

```tsx
// AFTER (Chipi)
const { data: balance } = useGetTokenBalance({
  walletAddress: address,
  tokenAddress: "USDC",
});
// No ABI. No public client. No chain config.
```

### writeContract

```tsx
// BEFORE (viem)
import { createWalletClient, custom } from "viem";

const walletClient = createWalletClient({
  chain: mainnet,
  transport: custom(window.ethereum),
});

const hash = await walletClient.writeContract({
  address: contractAddress,
  abi: contractAbi,
  functionName: "swap",
  args: [tokenIn, tokenOut, amount],
});
```

```tsx
// AFTER (Chipi)
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [{
    contractAddress: "0x_STARKNET_CONTRACT",
    entrypoint: "swap",
    calldata: [tokenIn, tokenOut, amountLow, "0"],
  }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No ABI file. No wallet client. Gasless.
```

### multicall (readContract batch)

```tsx
// BEFORE (viem multicall)
const results = await client.multicall({
  contracts: [
    { address: usdcAddr, abi: erc20Abi, functionName: "balanceOf", args: [user] },
    { address: daiAddr, abi: erc20Abi, functionName: "balanceOf", args: [user] },
    { address: wethAddr, abi: erc20Abi, functionName: "balanceOf", args: [user] },
  ],
});
```

```tsx
// AFTER (Chipi) — query each balance
const { data: usdcBal } = useGetTokenBalance({ walletAddress: addr, tokenAddress: "USDC" });
const { data: ethBal } = useGetTokenBalance({ walletAddress: addr, tokenAddress: "ETH" });
// For write multicalls, use calls[] array in useCallAnyContract
```

### watchEvent / watchContractEvent

```tsx
// BEFORE (viem)
const unwatch = client.watchContractEvent({
  address: contractAddress,
  abi: contractAbi,
  eventName: "Transfer",
  onLogs: (logs) => {
    logs.forEach((log) => console.log(log.args));
  },
});
// Later: unwatch();
```

```tsx
// AFTER (Chipi) — poll transaction history
import { useGetTransactionList } from "@chipi-stack/nextjs";

const { data: transactions, refetch } = useGetTransactionList({
  walletAddress: address,
});
// Set up polling interval to detect new transactions
useEffect(() => {
  const interval = setInterval(() => refetch(), 10_000);
  return () => clearInterval(interval);
}, [refetch]);
```

## wagmi Deep Patterns

> **Important:** These patterns use wagmi v2 API names. If migrating from wagmi v1, note that `useContractRead` → `useReadContract`, `useContractWrite` → `useWriteContract`, `useWaitForTransaction` → `useWaitForTransactionReceipt`.

### useReadContract

```tsx
// BEFORE (wagmi v2)
import { useReadContract } from "wagmi";

const { data: balance } = useReadContract({
  address: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  abi: erc20Abi,
  functionName: "balanceOf",
  args: [userAddress],
});
```

```tsx
// AFTER (Chipi)
import { useGetTokenBalance } from "@chipi-stack/nextjs";

const { data: balance } = useGetTokenBalance({
  walletAddress: userAddress,
  tokenAddress: "USDC",
});
```

### useWriteContract + useWaitForTransactionReceipt

```tsx
// BEFORE (wagmi v2)
import { useWriteContract, useWaitForTransactionReceipt } from "wagmi";

const { writeContract, data: hash } = useWriteContract();
const { isLoading, isSuccess } = useWaitForTransactionReceipt({ hash });

writeContract({
  address: contractAddress,
  abi: contractAbi,
  functionName: "swap",
  args: [tokenIn, tokenOut, amount],
});
```

```tsx
// AFTER (Chipi)
import { useCallAnyContract } from "@chipi-stack/nextjs";

const { mutateAsync: callContract, isPending } = useCallAnyContract();
const result = await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [{
    contractAddress: "0x_STARKNET_CONTRACT",
    entrypoint: "swap",
    calldata: [tokenIn, tokenOut, amountLow, "0"],
  }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No separate receipt hook. Result returned directly. Gasless.
```

### useBalance

```tsx
// BEFORE (wagmi v2)
import { useBalance } from "wagmi";

const { data: ethBalance } = useBalance({ address: userAddress });
const { data: usdcBalance } = useBalance({
  address: userAddress,
  token: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
});
```

```tsx
// AFTER (Chipi)
const { data: ethBal } = useGetTokenBalance({ walletAddress: addr, tokenAddress: "ETH" });
const { data: usdcBal } = useGetTokenBalance({ walletAddress: addr, tokenAddress: "USDC" });
```

### useReadContracts (multicall)

```tsx
// BEFORE (wagmi v2)
import { useReadContracts } from "wagmi";

const { data } = useReadContracts({
  contracts: [
    { address: usdcAddr, abi: erc20Abi, functionName: "balanceOf", args: [user] },
    { address: usdcAddr, abi: erc20Abi, functionName: "totalSupply" },
  ],
});
```

```tsx
// AFTER (Chipi) — individual hooks
const { data: balance } = useGetTokenBalance({ walletAddress: user, tokenAddress: "USDC" });
// For custom reads, use useCallAnyContract with calls[]
```

### useToken (deprecated in wagmi v2, but common in v1 codebases)

```tsx
// BEFORE (wagmi v1)
import { useToken } from "wagmi";

const { data: tokenData } = useToken({
  address: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
});
// Returns: { name, symbol, decimals, totalSupply }
```

```tsx
// AFTER (Chipi)
// Token metadata not needed on frontend.
// Chipi SDK knows decimals for supported tokens (USDC=6, ETH=18, STRK=18).
// For custom tokens, use useCallAnyContract to read name/symbol/decimals.
```

## Full Provider Stack Removal

### RainbowKit

```tsx
// REMOVE ALL OF THIS:
import "@rainbow-me/rainbowkit/styles.css";
import { RainbowKitProvider, ConnectButton, getDefaultConfig } from "@rainbow-me/rainbowkit";
import { WagmiProvider } from "wagmi";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { mainnet, polygon, arbitrum, optimism, base } from "wagmi/chains";

const config = getDefaultConfig({
  appName: "My App",
  projectId: "YOUR_WALLETCONNECT_ID",
  chains: [mainnet, polygon, arbitrum, optimism, base],
});
const queryClient = new QueryClient();

function App({ children }) {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider>
          <ConnectButton />
          {children}
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}
```

```tsx
// REPLACE WITH (in layout.tsx):
// ChipiProvider wraps the app — see chipi-wallet-setup skill
// No RainbowKit. No wagmi. No QueryClient. No WalletConnect project ID.
// Just ChipiProvider + your auth provider (Clerk/Firebase/Supabase).
```

### ConnectKit

```tsx
// REMOVE ALL OF THIS:
import { ConnectKitProvider, ConnectKitButton } from "connectkit";
import { WagmiProvider, createConfig, http } from "wagmi";
import { mainnet } from "wagmi/chains";

const config = createConfig({
  chains: [mainnet],
  transports: { [mainnet.id]: http() },
});

function App({ children }) {
  return (
    <WagmiProvider config={config}>
      <ConnectKitProvider>
        <ConnectKitButton />
        {children}
      </ConnectKitProvider>
    </WagmiProvider>
  );
}
```

```tsx
// REPLACE WITH:
// Same as RainbowKit — ChipiProvider + auth provider.
// No ConnectKit. No wagmi config. No chain transports.
```

### Web3Modal / AppKit (WalletConnect)

```tsx
// REMOVE ALL OF THIS:
import { createWeb3Modal } from "@web3modal/wagmi/react";
import { defaultWagmiConfig } from "@web3modal/wagmi/react/config";
// or newer AppKit:
import { createAppKit } from "@reown/appkit/react";
import { WagmiAdapter } from "@reown/appkit-adapter-wagmi";

const projectId = "YOUR_WALLETCONNECT_PROJECT_ID";
const config = defaultWagmiConfig({ chains: [mainnet], projectId });
createWeb3Modal({ wagmiConfig: config, projectId });

// or AppKit:
const wagmiAdapter = new WagmiAdapter({ projectId, chains: [mainnet] });
createAppKit({ adapters: [wagmiAdapter], projectId });
```

```tsx
// REPLACE WITH:
// ChipiProvider. No WalletConnect project ID needed.
// No Web3Modal. No AppKit. No wagmi adapter.
```

### wagmi + QueryClient Config (standalone)

```tsx
// REMOVE:
import { WagmiProvider, createConfig, http } from "wagmi";
import { mainnet, polygon, arbitrum } from "wagmi/chains";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { injected, metaMask, walletConnect } from "wagmi/connectors";

const config = createConfig({
  chains: [mainnet, polygon, arbitrum],
  connectors: [injected(), metaMask(), walletConnect({ projectId: "..." })],
  transports: {
    [mainnet.id]: http(),
    [polygon.id]: http(),
    [arbitrum.id]: http(),
  },
});
```

```tsx
// REPLACE WITH:
// ChipiProvider handles everything. No chains, no connectors, no transports.
// One network (StarkNet). One wallet type (passkey). Zero config.
```

## EIP-712 Typed Data Signing → Not Needed

```tsx
// BEFORE (EIP-712 — common for permit, gasless approvals, off-chain orders)
const domain = { name: "MyApp", version: "1", chainId: 1, verifyingContract: "0x..." };
const types = { Order: [{ name: "amount", type: "uint256" }, { name: "nonce", type: "uint256" }] };
const value = { amount: 1000, nonce: 1 };

// ethers.js v6:
const signature = await signer.signTypedData(domain, types, value);
// viem:
const signature = await walletClient.signTypedData({ domain, types, primaryType: "Order", message: value });
```

```tsx
// AFTER (Chipi)
// EIP-712 is not needed. Common use cases it replaces:
// - Permit/Permit2 (gasless approvals) → Chipi is natively gasless
// - Off-chain order signing → use standard backend auth
// - SIWE (Sign-In with Ethereum) → use Clerk/Firebase/Supabase auth
// Delete all signTypedData, EIP-712 domain/types definitions.
```

## ENS Resolution → Not Applicable

```tsx
// BEFORE (ENS)
// ethers.js:
const address = await provider.resolveName("vitalik.eth");
const ensName = await provider.lookupAddress(address);
// viem:
const address = await client.getEnsAddress({ name: "vitalik.eth" });
const name = await client.getEnsName({ address: "0x..." });
```

```tsx
// AFTER (Chipi)
// ENS does not exist on StarkNet.
// StarkNet has StarkNet ID (starknet.id) for human-readable addresses.
// The Chipi SDK does not handle name resolution — use raw addresses.
// If you need name resolution, integrate starknet.id separately.
```

## NPM Packages to Remove

When migrating from EVM, remove these from `package.json`:

```json
{
  "dependencies": {
    // REMOVE ALL OF THESE (as applicable):
    "ethers": "^5.x or ^6.x",
    "viem": "^2.x",
    "wagmi": "^2.x",
    "@rainbow-me/rainbowkit": "^2.x",
    "connectkit": "^1.x",
    "@web3modal/wagmi": "^5.x",
    "@reown/appkit": "^1.x",
    "@reown/appkit-adapter-wagmi": "^1.x",
    "@tanstack/react-query": "^5.x",
    "web3": "^4.x",
    "@walletconnect/web3-provider": "^2.x",
    "@metamask/sdk": "^0.x",
    "@safe-global/safe-apps-sdk": "^9.x",

    // ADD:
    "@chipi-stack/nextjs": "latest"
    // (or @chipi-stack/chipi-react or @chipi-stack/chipi-expo)
  }
}
```

Run:
```bash
npm uninstall ethers viem wagmi @rainbow-me/rainbowkit connectkit @web3modal/wagmi @reown/appkit @reown/appkit-adapter-wagmi @tanstack/react-query web3 @walletconnect/web3-provider @metamask/sdk
npm install @chipi-stack/nextjs
```

> **Note:** Only remove `@tanstack/react-query` if it was installed solely for wagmi. If your app uses it for other data fetching, keep it.

## Address Format Validation

| Property | EVM | StarkNet |
|----------|-----|----------|
| Format | 0x + 40 hex chars | 0x + 64 hex chars |
| Example | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | `0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb` |
| Checksum | EIP-55 mixed-case checksum | No checksum (case-insensitive) |
| Validation | `/^0x[0-9a-fA-F]{40}$/` | `/^0x[0-9a-fA-F]{64}$/` |

### Address Detection Helper

```tsx
function detectAddressType(address: string): "evm" | "starknet" | "invalid" {
  if (/^0x[0-9a-fA-F]{40}$/.test(address)) return "evm";
  if (/^0x[0-9a-fA-F]{64}$/.test(address)) return "starknet";
  return "invalid";
}

// Usage: warn users who paste EVM addresses
const type = detectAddressType(input);
if (type === "evm") {
  showError("This looks like an Ethereum address. Please use your StarkNet address (0x + 64 chars).");
}
```

## Approve + Interact Multicall

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
// On EVM: 2 txs, 2 gas fees, 2 confirmations.
// On StarkNet via Chipi: 1 tx, 0 gas, 1 passkey tap.
```

## Event Listening → Transaction History

```tsx
// BEFORE (ethers.js v5/v6)
const contract = new ethers.Contract(address, abi, provider);
contract.on("Transfer", (from, to, amount) => {
  console.log(`Transfer: ${from} → ${to}: ${amount}`);
});

// BEFORE (viem)
const unwatch = client.watchContractEvent({
  address: contractAddress,
  abi,
  eventName: "Transfer",
  onLogs: (logs) => logs.forEach((log) => console.log(log.args)),
});
```

```tsx
// AFTER (Chipi)
import { useGetTransactionList } from "@chipi-stack/nextjs";

const { data: transactions, refetch } = useGetTransactionList({
  walletAddress: address,
});
// StarkNet events exist but Chipi SDK provides transaction history.
// Poll for new transactions instead of subscribing to events.
```

## SIWE (Sign-In with Ethereum) → Standard Auth

```tsx
// BEFORE (SIWE)
import { SiweMessage } from "siwe";

const message = new SiweMessage({
  domain: window.location.host,
  address: userAddress,
  statement: "Sign in to MyApp",
  uri: window.location.origin,
  version: "1",
  chainId: 1,
  nonce: await fetchNonce(),
});
const signature = await signer.signMessage(message.prepareMessage());
// Send { message, signature } to backend for verification
```

```tsx
// AFTER (Chipi) — standard auth, no wallet signing
// Use Clerk, Firebase, or Supabase for authentication.
// The wallet is created AFTER auth, not used FOR auth.
import { useUser } from "@clerk/nextjs";
const { user, isSignedIn } = useUser();
// User signs in with email/Google/passkey — wallet is separate.
// No SIWE library. No signature verification. No nonce management.
```
