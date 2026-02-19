---
name: chipi-migrate-from-evm
description: Migrate an existing EVM (Ethereum, Base, Polygon, Arbitrum, Optimism) dapp to StarkNet with Chipi. Maps familiar concepts (ethers.js, viem, MetaMask, WalletConnect) to Chipi equivalents. Use when user says "migrate from Ethereum", "port my EVM app", "convert from Solidity", "StarkNet equivalent of", or "migrate my dapp".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.1.0
  mcp-server: chipi-registry
---

# Migrate from EVM to StarkNet with Chipi

Help builders who already have working apps on EVM chains (Ethereum, Base, Polygon, Arbitrum, Optimism) port them to StarkNet using Chipi's infrastructure. They understand crypto — StarkNet is just unfamiliar territory.

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

> **Why batch calls matter:** On EVM, approve + swap = 2 separate transactions (2 gas fees, 2 confirmations). On StarkNet via Chipi, approve + swap = 1 transaction (0 gas, 1 passkey tap). This alone is a massive UX upgrade.

## Step 1: Audit the Existing App

Analyze what the current app does. Ask the user about:
- **Token operations** — transfers, approvals, swaps, staking
- **Contract interactions** — which functions, what calldata, which contracts
- **Auth flow** — MetaMask? WalletConnect? Email + wallet?
- **User data** — stored on-chain vs off-chain
- **Dependencies** — ethers.js? viem? web3.js? wagmi?

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
- Seed phrase generation / storage
- Manual wallet connection flows

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

## Step 5: Replace Auth Layer

**Remove:**
- Wallet-based auth (sign message → verify signature)
- "Connect Wallet" as login
- Wallet state management

**Replace with:**
- Standard auth (Clerk/Firebase/Supabase) + Chipi wallet
- Passkey = auth + wallet in one step
- Users sign in with email/Google/passkey, wallet is created automatically

> **Why separate auth from wallet:** On EVM, your wallet IS your identity. On Chipi, your auth provider handles identity and Chipi handles the wallet. This means users can recover access through email, not seed phrases.

## Step 6: Handle Custom Contracts

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

## Step 7: Test Migration

1. **Verify all token operations work** — transfers, balances, approvals
2. **Verify custom contract calls** return expected results
3. **Verify gasless UX** — users should never see gas prompts
4. **Compare: old UX vs new UX** — should be dramatically simpler
5. **Check error handling** — map old error codes to new ones

> **Why compare UX:** The migration should feel like an upgrade to your users, not a lateral move. If the new UX isn't noticeably better, something is wrong.

## Reference Files

For detailed function-level mappings, see:
- `references/evm-to-starknet-mapping.md` — Detailed ethers.js / viem → Chipi SDK mapping
- `references/token-bridge-guide.md` — How to bridge assets from EVM chains to StarkNet

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key migration-specific UI rules:
- Show a "migration checklist" component with checkmarks as each layer is replaced
- Use before/after code comparisons to validate the migration
- Display chain indicators ("Migrated from Ethereum" badge) during testing
