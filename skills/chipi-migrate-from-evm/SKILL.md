---
name: chipi-migrate-from-evm
description: Migrate an existing EVM (Ethereum, Base, Polygon, Arbitrum, Optimism) dapp to StarkNet with Chipi. Maps familiar concepts (ethers.js, viem/wagmi, MetaMask, WalletConnect) to Chipi equivalents. Use when user says "migrate from Ethereum", "port my EVM app", "convert from Solidity", "StarkNet equivalent of", or "migrate my dapp".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.2.0
  mcp-server: chipi-registry
---

# Migrate from EVM to StarkNet with Chipi

Help builders who already have working apps on EVM chains (Ethereum, Base, Polygon, Arbitrum, Optimism) port them to StarkNet using Chipi's infrastructure. They understand crypto — StarkNet is just unfamiliar territory.

## Used in

DeFi protocols, DEX frontends, NFT marketplaces, payment apps, token dashboards, staking platforms, DAO tools — any EVM dapp that wants gasless UX, native account abstraction, and ZK security.

> **Migrating from Solana instead?** Load the `chipi-migrate-from-solana` skill — it has deep Solana-specific mappings for Anchor, wallet-adapter, PDAs, SPL tokens, and the account model.

## When in Doubt, Ask

If the user's existing codebase uses patterns, libraries, or contract interactions you're unsure about, ASK before mapping them to StarkNet equivalents. It's better to confirm than to map incorrectly. Never guess at contract addresses or function signatures.

## Why Migrate?

> **Why this matters:** Builders on EVM already have working apps, users, and revenue. They're not starting from zero — they're upgrading. StarkNet + Chipi gives them superpowers their current chain can't match.

- **StarkNet:** 10-100x cheaper gas through ZK rollup validity proofs
- **Chipi:** Gasless UX — your users pay zero gas, ever
- **Native account abstraction** — no MetaMask, no seed phrases, passkey auth built in
- **Session keys** — one auth, many txs (impossible on raw EVM without ERC-4337)
- **Cairo > Solidity** for provable computation (ZK-native)

## The Biggest Mental Shifts

These are the concepts that trip up EVM developers the most. Read this section carefully.

### 1. EOAs Don't Exist

On EVM, most users have EOAs (externally owned accounts) — a raw key pair. On StarkNet, every account is a smart contract. This means account abstraction is native, not bolted on via ERC-4337. No bundlers, no paymasters, no EntryPoint contract.

```text
EVM:      EOA (key pair) → signs tx → submits to mempool
StarkNet: Smart contract account → executes tx → proven by ZK
```

### 2. No Chain Switching

On EVM, your app might support Ethereum, Polygon, Arbitrum, Base, Optimism — with chain switching, multi-chain configs, and "wrong network" errors. On StarkNet via Chipi, there's one network. No `wallet_switchEthereumChain`. No chain ID checks. No multi-chain config.

### 3. Gasless Is Native, Not a Hack

On EVM, gasless UX requires relayers (OpenGSN), meta-transactions (EIP-2771), Permit2, or ERC-4337 paymasters — all complex infrastructure. On Chipi, gasless is the default. Every transaction is sponsored. No setup needed.

### 4. Multicall Is Free

On EVM, batching calls requires a multicall contract (Multicall3) or separate transactions. On StarkNet, every account natively supports multicall — pass an array of calls. Via Chipi, approve + swap = 1 transaction, 0 gas, 1 passkey tap.

### 5. No ABI Files on Frontend

On EVM, you need ABI JSON files to interact with contracts (ethers.Contract, viem's readContract/writeContract). With Chipi's `useCallAnyContract`, you pass the entrypoint name and calldata directly — no ABI import needed.

### 6. uint256 = Two Felts

EVM's uint256 is a single 256-bit value. StarkNet's u256 is two felt252 values: `[low_128_bits, high_128_bits]`. For amounts under 2^128 (almost all practical values), pass `[amount, "0"]`.

## Concept Mapping

### EVM → StarkNet

| EVM Concept | StarkNet Equivalent | Chipi Abstraction |
|---|---|---|
| MetaMask / WalletConnect | Native account abstraction | `useCreateWallet` (passkey, no extension) |
| EOA (externally owned account) | Every account is a smart contract | CHIPI wallet type |
| ERC-20 transfer | ERC-20 transfer (same standard) | `useTransfer` (gasless) |
| ERC-20 approve + swap | approve + call | `useCallAnyContract` (gasless, batched in 1 tx) |
| ethers.js / viem | starknet.js | Chipi SDK (wraps starknet.js) |
| Solidity | Cairo | Deploy on StarkNet, call via `useCallAnyContract` |
| Gas fees (user pays) | Gas fees (Chipi sponsors) | All txs gasless |
| ERC-4337 (account abstraction) | Native — every account is a contract | Built-in, no bundler needed |
| Permit2 / gasless relayer | Not needed | Chipi handles gas natively |
| msg.sender | get_caller_address() | Handled by SDK |
| block.timestamp | get_block_timestamp() | Handled by SDK |
| Hardhat / Foundry | Scarb / Starknet Foundry | For contract devs only |
| uint256 | u256 (felt252 pair: low, high) | SDK handles encoding |
| bytes / calldata | felt252 array | SDK handles encoding |
| events (emit) | events (emit) | Same pattern, different syntax |
| Multicall3 contract | Native multicall (account abstraction) | `calls[]` array in `useCallAnyContract` |
| EIP-712 typed data signing | Not needed for common use cases | Gasless native, standard auth |
| ENS (.eth names) | StarkNet ID (.stark names) | Not handled by SDK — use raw addresses |
| Chain switching (switchChain) | Single network (StarkNet) | No chain config needed |

> **Why batch calls matter:** On EVM, approve + swap = 2 separate transactions (2 gas fees, 2 confirmations). On StarkNet via Chipi, approve + swap = 1 transaction (0 gas, 1 passkey tap). This alone is a massive UX upgrade.

## Step 1: Audit the Existing App

Analyze what the current app does. Ask the user about:
- **Token operations** — transfers, approvals, swaps, staking
- **Contract interactions** — which functions, what calldata, which contracts
- **Auth flow** — MetaMask? WalletConnect? Email + wallet?
- **User data** — stored on-chain vs off-chain
- **Dependencies** — ethers.js? viem? web3.js? wagmi?
- **Provider stack** — RainbowKit? ConnectKit? Web3Modal/AppKit?

> **Why audit first:** Migrating blind leads to missed features and broken flows. Map everything before changing anything.

## Step 2: Map Contracts

For each contract the app interacts with:

### Check if a StarkNet equivalent exists
Common tokens and protocols already on StarkNet (as of Feb 2026 — verify on [Starkscan](https://starkscan.co)):
- **USDC:** `0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb`
- **ETH:** `0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7`
- **STRK:** `0x04718f5a0fc34cc1af16a1cdee98ffb20c31f5cd61d6ab07201858f4287c938d`
- **VESU** (lending): Available via Chipi's `useStakeVesuUsdc`
- **Many DeFi protocols** have deployed on StarkNet — check ecosystem sites

### Do you need to deploy a Cairo contract?
If the app has custom Solidity logic:
- **Option A:** Find a StarkNet equivalent (many DeFi protocols have deployed)
- **Option B:** Rewrite in Cairo and deploy (load `chipi-custom-contracts` skill for integration)
- **Option C:** Use Chipi's built-in features if they cover the use case (payments, staking, SKUs)

### Can Chipi's built-in features replace it?
| If the app does... | Chipi replaces it with... |
|---|---|
| ERC-20 transfers | `useTransfer` (gasless) |
| Token approvals | Batched into `useCallAnyContract` calls |
| Staking / lending | `useStakeVesuUsdc` / `useWithdrawVesuUsdc` |
| Digital goods purchase | SKU marketplace (`usePurchaseSku`) |
| Frequent interactions | Session keys (`useChipiSession`) |

## Step 3: Replace Wallet Layer

**Remove:**
- ethers.js / viem / web3.js / wagmi
- MetaMask / WalletConnect integration
- RainbowKit / ConnectKit / Web3Modal / AppKit
- Seed phrase generation / storage
- Manual wallet connection flows
- Chain switching logic

**Replace with:**
- Chipi SDK (`useCreateWallet`, `useChipiWallet`)
- Users get passkey wallets — no browser extension needed
- One line to check wallet: `const { hasWallet, wallet } = useChipiWallet()`

> **Why this is better:** Your users go from "install MetaMask → create account → write down 12 words → connect wallet → approve site" to "tap fingerprint". That's the entire onboarding.

**Code mapping (ethers.js → Chipi):**

```tsx
// BEFORE (ethers.js):
const provider = new ethers.BrowserProvider(window.ethereum);
const signer = await provider.getSigner();
const address = await signer.getAddress();

// AFTER (Chipi):
const { wallet, hasWallet } = useChipiWallet();
const address = wallet?.publicKey;
// That's it. No provider, no signer, no window.ethereum.
```

**Code mapping (viem/wagmi → Chipi):**

```tsx
// BEFORE (wagmi):
import { useAccount, useConnect, useDisconnect } from "wagmi";
const { address, isConnected } = useAccount();
const { connect, connectors } = useConnect();
const { disconnect } = useDisconnect();

// AFTER (Chipi):
import { useChipiWallet, useCreateWallet } from "@chipi-stack/nextjs";
const { wallet, hasWallet } = useChipiWallet();
// No connectors, no chain switching, no window.ethereum detection.
```

**Full provider stack removal (RainbowKit example):**

```tsx
// REMOVE ALL OF THIS:
import "@rainbow-me/rainbowkit/styles.css";
import { RainbowKitProvider, ConnectButton, getDefaultConfig } from "@rainbow-me/rainbowkit";
import { WagmiProvider } from "wagmi";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { mainnet, polygon, arbitrum } from "wagmi/chains";

const config = getDefaultConfig({
  appName: "My App",
  projectId: "YOUR_WALLETCONNECT_ID",
  chains: [mainnet, polygon, arbitrum],
});

// REPLACE WITH:
// ChipiProvider wraps the app — see chipi-wallet-setup skill
// No RainbowKit. No wagmi. No QueryClient. No WalletConnect project ID.
```

> See `references/evm-code-migration-patterns.md` for full ConnectKit, Web3Modal/AppKit, and standalone wagmi removal patterns.

## Step 4: Replace Transaction Layer

**Remove:**
- Manual gas estimation (`estimateGas`)
- Nonce management
- Transaction signing (`signer.sendTransaction`)
- Gas price oracles
- Transaction receipt polling

**Replace with:**
- Chipi hooks (`useTransfer`, `useCallAnyContract`)
- All transactions are gasless and sponsored
- Built-in transaction status tracking

> **Why gasless matters:** On EVM, users need ETH to do anything — even send USDC. On Chipi, users never see gas. They pay in the token they're actually using (or nothing at all for sponsored txs).

**Code mapping:**

```tsx
// BEFORE (ethers.js ERC-20 transfer):
const usdc = new ethers.Contract(usdcAddress, erc20Abi, signer);
const tx = await usdc.transfer(recipient, ethers.parseUnits("10", 6));
await tx.wait();

// AFTER (Chipi):
const { mutateAsync: transfer } = useTransfer();
await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: 10,  // Human-readable, SDK handles decimals
  tokenAddress: "USDC",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Step 5: Replace Token Operations

**Remove:**
- ERC-20 ABI imports and Contract instances
- Manual `approve()` + `transferFrom()` patterns
- Token decimal conversion (`parseUnits`, `formatUnits`)
- Allowance checks (`allowance()`)

**Replace with:**
- `useTransfer` for token transfers (handles decimals automatically)
- `useGetTokenBalance` for balance checks
- `useCallAnyContract` with `calls[]` for approve + action in one tx

**ERC-20 approval flow → Chipi multicall:**

```tsx
// BEFORE (ethers.js — 2 txs, 2 gas fees):
const usdc = new ethers.Contract(usdcAddress, erc20Abi, signer);
await (await usdc.approve(dexAddress, amount)).wait();
const dex = new ethers.Contract(dexAddress, dexAbi, signer);
await (await dex.swap(tokenIn, tokenOut, amount)).wait();

// AFTER (Chipi — 1 tx, 0 gas):
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [
    { contractAddress: usdcAddress, entrypoint: "approve", calldata: [dexAddress, amountLow, "0"] },
    { contractAddress: dexAddress, entrypoint: "swap", calldata: [tokenIn, tokenOut, amountLow, "0"] },
  ],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

**Decimal handling:**

| Token | EVM Decimals | StarkNet Decimals | Chipi SDK |
|-------|-------------|-------------------|-----------|
| USDC | 6 | 6 | Pass human-readable (e.g., `10` for $10) |
| ETH | 18 (wei) | 18 (wei) | Pass human-readable (e.g., `0.1`) |
| STRK | N/A | 18 | Pass human-readable (e.g., `1.5`) |
| Custom | Varies | Varies | Use `otherToken.decimals` |

## Step 6: Replace Auth Layer

**Remove:**
- Wallet-based auth (sign message → verify signature)
- SIWE (Sign-In with Ethereum) / EIP-4361
- "Connect Wallet" as login
- Wallet state management
- EIP-712 typed data signing for auth

**Replace with:**
- Standard auth (Clerk/Firebase/Supabase) + Chipi wallet
- Passkey = auth + wallet in one step
- Users sign in with email/Google/passkey, wallet is created automatically

> **Why separate auth from wallet:** On EVM, your wallet IS your identity. On Chipi, your auth provider handles identity and Chipi handles the wallet. This means users can recover access through email, not seed phrases.

## Step 7: Handle Custom Contracts

If you have Solidity contracts with custom logic:

**Option A: Find StarkNet equivalent**
Many DeFi protocols, NFT standards, and utilities have StarkNet versions. Check the StarkNet ecosystem.

**Option B: Rewrite in Cairo and deploy**
- Cairo is StarkNet's native language (like Solidity for EVM)
- Use Scarb for project management, Starknet Foundry for testing
- Load the `chipi-custom-contracts` skill for integration guidance
- Once deployed, call via `useCallAnyContract`

**Option C: Use Chipi's built-in features**
If the custom contract just does transfers, approvals, staking, or marketplace operations — Chipi has hooks for all of these. No custom contract needed.

## Step 8: Address Format

EVM uses 40-character hex addresses with EIP-55 checksums. StarkNet uses 64-character hex felt252 values.

| Property | EVM | StarkNet |
|----------|-----|----------|
| Format | 0x + 40 hex chars | 0x + 64 hex chars |
| Example | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | `0x033068f6539f8e...` |
| Checksum | EIP-55 mixed-case | None (case-insensitive) |
| Validation | `/^0x[0-9a-fA-F]{40}$/` | `/^0x[0-9a-fA-F]{64}$/` |

**Common mistake:** Using an EVM address (40 chars) on StarkNet. Always validate address length.

```tsx
// Address detection helper
function detectAddressType(addr: string): "evm" | "starknet" | "invalid" {
  if (/^0x[0-9a-fA-F]{40}$/.test(addr)) return "evm";
  if (/^0x[0-9a-fA-F]{64}$/.test(addr)) return "starknet";
  return "invalid";
}
```

## Step 9: Test Migration

1. **Verify all token operations work** — transfers, balances, approvals
2. **Verify custom contract calls** return expected results
3. **Verify gasless UX** — users should never see gas prompts
4. **Verify no chain-switching UI remains** — no network dropdowns, no "wrong network" errors
5. **Verify address format** — no 40-char EVM addresses leaking into StarkNet calls
6. **Compare: old UX vs new UX** — should be dramatically simpler
7. **Check error handling** — map old error codes to new ones

> **Why compare UX:** The migration should feel like an upgrade to your users, not a lateral move. If the new UX isn't noticeably better, something is wrong.

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `Cannot find module 'ethers'` | Removed package but imports remain | Search for all `ethers`/`viem`/`wagmi` imports and remove |
| `window.ethereum is undefined` | Leftover MetaMask detection | Remove all `window.ethereum` checks — Chipi uses passkeys |
| EVM address (40 chars) rejected | Using Ethereum address on StarkNet | Convert to 0x + 64 hex chars |
| Chain switching errors | Leftover multi-chain config | Remove — StarkNet is the only network |
| `nonce too low` patterns in code | Leftover nonce management | Remove — Chipi handles nonces internally |
| Gas estimation errors | Leftover gas logic | Remove — all Chipi txs are gasless |
| `execution reverted` | Wrong calldata or entrypoint | Check u256 encoding [low, high] and entrypoint name |
| ERC-20 allowance errors | Forgot approve in multicall | Use `calls[]` to batch approve + action |
| BigNumber / bigint errors | Leftover ethers.js math | Use human-readable amounts (`amount: 10`) |
| Provider kit init errors | RainbowKit/ConnectKit config remains | Remove entire provider stack — use ChipiProvider |

> For detailed fixes with code examples, see `references/evm-common-errors.md`.

## Hooks Used

All hooks from `@chipi-stack/nextjs` (or `@chipi-stack/chipi-react` / `@chipi-stack/chipi-expo`):

| Chipi Hook | Replaces (EVM) |
|---|---|
| `useCreateWallet` | MetaMask connection, WalletConnect, RainbowKit ConnectButton |
| `useChipiWallet` | `useAccount` (wagmi), `provider.getSigner()` (ethers) |
| `useTransfer` | `contract.transfer()` (ethers), `writeContract` (viem/wagmi) |
| `useGetTokenBalance` | `contract.balanceOf()` (ethers), `useBalance`/`useReadContract` (wagmi) |
| `useCallAnyContract` | `contract.functionName()` (ethers), `writeContract` (viem/wagmi) |
| `useGetTransactionList` | `contract.on(event)` (ethers), `watchContractEvent` (viem) |
| `useChipiSession` | No EVM equivalent (ERC-4337 session keys are non-standard) |
| `useStakeVesuUsdc` | Custom staking contract calls |
| `usePurchaseSku` | Custom marketplace contract calls |

## Reference Files

For detailed code-level mappings, see:
- `references/evm-to-starknet-mapping.md` — Quick-reference ethers.js / viem → Chipi SDK mapping
- `references/evm-code-migration-patterns.md` — Deep patterns: v5/v6, wagmi, provider stack removal, multicall, NPM cleanup
- `references/evm-common-errors.md` — 18 common migration errors with fixes
- `references/token-bridge-guide.md` — How to bridge assets from EVM chains to StarkNet

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key migration-specific UI rules:
- Show a "migration checklist" component with checkmarks as each layer is replaced
- Use before/after code comparisons to validate the migration
- Display chain indicators ("Migrated from Ethereum" badge) during testing

## What's Next

- `chipi-session-keys` — Add frictionless transactions (one auth, many txs)
- `chipi-custom-contracts` — Call any StarkNet contract through Chipi
- `chipi-debug` — Diagnose errors, validate addresses, troubleshoot failed transactions
