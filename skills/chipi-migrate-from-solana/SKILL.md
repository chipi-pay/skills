---
name: chipi-migrate-from-solana
description: Migrate an existing Solana dapp to StarkNet with Chipi. Deep mappings for Anchor, @solana/web3.js, wallet-adapter, PDAs, SPL tokens, and the Solana account model. Use when user says "migrate from Solana", "port my Solana app", "convert from Anchor", "replace Phantom", "Solana to StarkNet", or "move from Solana".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Migrate from Solana to StarkNet with Chipi

Help builders who already have working apps on Solana port them to StarkNet using Chipi's infrastructure. They understand Solana's account model, PDAs, Anchor, and SPL tokens — StarkNet is just unfamiliar territory.

## Used in

DeFi protocols, NFT marketplaces, payment apps, token projects, gaming, social apps — any Solana dapp that wants gasless UX, native account abstraction, and ZK security.

## When in Doubt, Ask

If the user's existing codebase uses patterns, libraries, or program interactions you're unsure about, ASK before mapping them to StarkNet equivalents. It's better to confirm than to map incorrectly. Never guess at contract addresses, PDAs, or function signatures.

## Why Migrate?

> **Why this matters:** Builders on Solana already have working apps, users, and revenue. They're upgrading to get features Solana can't offer natively.

- **Gasless UX** — your users pay zero gas through Chipi (vs ~$0.0008–$0.005 per tx on Solana depending on priority fees and network conditions, which still requires SOL)
- **Native account abstraction** — no Phantom, no seed phrases, passkey auth built in
- **No rent** — deploy once, stays forever (Solana charges rent-exemption deposits, typically ~0.002 SOL for token accounts but varies by data size)
- **Session keys** — one auth, many txs (no Solana equivalent without custom programs)
- **ZK security** — validity proofs guarantee correctness (unlike optimistic rollups; Solana uses deterministic execution on L1)
- **Cairo > Rust** for provable computation (ZK-native language)

## The Biggest Mental Shifts

These are the concepts that trip up Solana developers the most. Read this section carefully.

### 1. Account Model → Contract Storage

On Solana, programs are stateless and data lives in separate accounts. You pass accounts into instructions. On StarkNet, contracts have their own storage — like Ethereum but with native AA.

```text
Solana:   Program + Data Account + PDA → instruction with accounts[]
StarkNet: Contract with storage → function call with calldata[]
```

### 2. PDAs Don't Exist

There is no `findProgramAddressSync` on StarkNet. PDAs are Solana's way of letting programs own data. On StarkNet, contracts store data directly in storage variables. No seeds, no bumps, no derivation.

### 3. No Rent

Solana charges rent for storing data on-chain. Accounts must be rent-exempt (minimum ~0.002 SOL). On StarkNet, there's no rent — deploy a contract, it stays forever. No ongoing costs for your users.

### 4. No ATAs (Associated Token Accounts)

On Solana, every token requires an ATA per wallet. You call `getOrCreateAssociatedTokenAccount`. On StarkNet, tokens use the ERC-20 standard — just call `balanceOf(address)`. No token accounts to create or manage.

### 5. No Account Declarations in Transactions

Solana transactions must declare every account they touch upfront (for Sealevel parallelism). StarkNet transactions just specify the function and calldata. No account lists.

### 6. Sealevel Parallel → ZK Sequential Proofs

Solana's runtime executes transactions in parallel (Sealevel). StarkNet batches transactions and generates ZK proofs. Different performance model, but you don't need to think about it — Chipi abstracts the execution layer.

## Concept Mapping

| Solana Concept | StarkNet Equivalent | Chipi Abstraction |
|---|---|---|
| Phantom / wallet-adapter | Native account abstraction | `useCreateWallet` (passkey, no extension) |
| Program (stateless) | Contract (with storage) | Call via `useCallAnyContract` |
| SPL Token transfer | ERC-20 transfer (same standard) | `useTransfer` (gasless) |
| PDA (program derived address) | Contract storage variables | Standard contract patterns |
| ATA (associated token account) | Not needed (ERC-20 `balanceOf`) | `useGetTokenBalance` |
| Anchor framework | Cairo + Scarb | For contract devs only |
| Transaction fees (~$0.001 SOL) | Gas fees (Chipi sponsors) | All txs gasless |
| Instruction | External function call | `useCallAnyContract` |
| Account (data storage) | Contract storage variables | Storage slots in Cairo |
| CPI (cross-program invocation) | Contract-to-contract calls | Native in Cairo |
| Rent / rent-exempt | No rent concept | One-time deploy cost |
| Anchor IDL | Cairo ABI (Sierra JSON) | SDK handles encoding |
| Versioned transactions / address lookup tables | Batched calls (native multicall) | `calls[]` array |
| Compute budget / priority fees | Not needed | Gasless through Chipi |
| Base58 addresses | 0x + 64 hex chars (felt252) | SDK handles format |

> **Why no ATAs:** On Solana, sending USDC to someone requires checking if they have an ATA, creating one if not, then transferring. On StarkNet via Chipi: `useTransfer({ recipientAddress, amount, tokenAddress: "USDC" })`. That's it.

## Step 1: Audit the Solana App

Analyze what the current app does. Ask the user about:
- **Programs** — which programs does the app interact with? Custom or standard (Token, Metaplex, etc.)?
- **PDAs** — list all PDAs, their seeds, and what data they store
- **CPIs** — does the app chain program calls? Which programs call which?
- **ATAs** — which tokens does the app handle? Any custom SPL tokens?
- **Wallet adapter** — which wallets are supported? (Phantom, Solflare, Backpack, etc.)
- **Anchor version** — is the app using Anchor? Which version? (0.28+, 0.29+, etc.)
- **@solana/web3.js version** — v1 (legacy) or v2 (pipe-based)?
- **Auth** — wallet-based signing or separate auth?

> **Why audit first:** Solana apps often have deep PDA dependency trees. Missing a single PDA mapping will break the migration.

## Step 2: Map Programs to Contracts

For each Solana program the app interacts with:

### Anchor IDL → Cairo ABI

```text
Anchor IDL:                          Cairo ABI:
├─ instructions[]                    ├─ external functions[]
│  ├─ name: "initialize"            │  ├─ name: "initialize"
│  ├─ accounts[]                    │  └─ inputs[] (calldata)
│  └─ args[]                        │
├─ accounts[]                        ├─ storage variables
│  └─ type definition               │  └─ defined in contract
└─ types[]                           └─ structs/enums
```

Key difference: Anchor IDL declares account constraints (ownership, mutability, signer) declaratively. In Cairo, you must implement equivalent runtime checks explicitly — use `get_caller_address()` for ownership/signer verification, and access control modifiers for mutability. Storage is internal to the contract, but you still need to guard who can write to it.

### Check if a StarkNet equivalent exists

Common protocols already on StarkNet:
- **USDC:** `0x033068f6539f8e6e6b131e6b2b814e6c34a5224bc66947c47dab9dfee93b35fb` (as of Feb 2026 — verify on [Starkscan](https://starkscan.co))
- **ETH:** `0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7`
- **STRK:** `0x04718f5a0fc34cc1af16a1cdee98ffb20c31f5cd61d6ab07201858f4287c938d`
- **VESU** (lending): Available via Chipi's `useStakeVesuUsdc`
- **Many DeFi protocols** have deployed on StarkNet — check ecosystem sites

### Can Chipi's built-in features replace it?

| If the Solana app does... | Chipi replaces it with... |
|---|---|
| SPL Token transfers | `useTransfer` (gasless, no ATA management) |
| Token approvals + swaps | Batched into `useCallAnyContract` calls (1 tx) |
| Staking / lending | `useStakeVesuUsdc` / `useWithdrawVesuUsdc` |
| Digital goods purchase | SKU marketplace (`usePurchaseSku`) |
| Frequent interactions | Session keys (`useChipiSession`) |

## Step 3: Replace Wallet Layer

**Remove:**
- @solana/wallet-adapter-react
- @solana/wallet-adapter-react-ui
- @solana/wallet-adapter-wallets (Phantom, Solflare, etc.)
- ConnectionProvider, WalletProvider, WalletModalProvider
- useWallet, useConnection hooks
- wallet-adapter CSS imports

**Replace with:**
- Chipi SDK (`useCreateWallet`, `useChipiWallet`)
- Users get passkey wallets — no browser extension needed
- One line to check wallet: `const { hasWallet, wallet } = useChipiWallet()`

> **Why this is better:** Your users go from "install Phantom → create account → write down 12 words → connect wallet → approve site" to "tap fingerprint". That's the entire onboarding.

**Full wallet-adapter removal:**

```tsx
// BEFORE (Solana wallet-adapter):
import { ConnectionProvider, WalletProvider } from "@solana/wallet-adapter-react";
import { WalletModalProvider, WalletMultiButton } from "@solana/wallet-adapter-react-ui";
import { PhantomWalletAdapter, SolflareWalletAdapter } from "@solana/wallet-adapter-wallets";
import "@solana/wallet-adapter-react-ui/styles.css";

const wallets = [new PhantomWalletAdapter(), new SolflareWalletAdapter()];

function App({ children }) {
  return (
    <ConnectionProvider endpoint={clusterApiUrl("mainnet-beta")}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          <WalletMultiButton />
          {children}
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}
```

```tsx
// AFTER (Chipi):
import { useChipiWallet } from "@chipi-stack/nextjs";

function App({ children }) {
  // ChipiProvider is set up in layout.tsx (see chipi-wallet-setup skill)
  return <>{children}</>;
}

function WalletButton() {
  const { wallet, hasWallet } = useChipiWallet();
  // No extension. No connection. No modal. Just a passkey.
  if (!hasWallet) return <CreateWalletButton />;
  return <WalletSummary wallet={wallet} />;
}
```

## Step 4: Replace Transaction Layer

**Remove:**
- @solana/web3.js Transaction / VersionedTransaction
- sendTransaction, confirmTransaction
- Compute budget programs
- Priority fee estimation
- Blockhash fetching (`getLatestBlockhash`)
- Transaction serialization

**Replace with:**
- Chipi hooks (`useTransfer`, `useCallAnyContract`)
- All transactions are gasless and sponsored
- Built-in transaction status tracking

> **Why gasless matters:** On Solana, users need SOL to do anything — even send USDC. On Chipi, users never see gas. They pay in the token they're actually using (or nothing at all).

**Code mapping:**

```tsx
// BEFORE (@solana/web3.js):
import { Transaction, SystemProgram } from "@solana/web3.js";

const tx = new Transaction().add(
  SystemProgram.transfer({
    fromPubkey: wallet.publicKey,
    toPubkey: recipientPubkey,
    lamports: 1_000_000_000, // 1 SOL
  })
);
const blockhash = await connection.getLatestBlockhash();
tx.recentBlockhash = blockhash.blockhash;
tx.feePayer = wallet.publicKey;
const signature = await sendTransaction(tx, connection);
await connection.confirmTransaction(signature);
```

```tsx
// AFTER (Chipi):
import { useTransfer } from "@chipi-stack/nextjs";

const { mutateAsync: transfer } = useTransfer();
await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: 10,  // Human-readable, SDK handles decimals
  tokenAddress: "USDC",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No blockhash. No fee payer. No confirmation polling. Gasless.
```

## Step 5: Replace Token Operations

**Remove:**
- @solana/spl-token (createTransferInstruction, getAssociatedTokenAddress, getOrCreateAssociatedTokenAccount)
- Manual ATA creation and lookup
- Token mint / decimals management
- SPL Token program interactions

**Replace with:**
- `useTransfer` for token transfers (handles decimals automatically)
- `useGetTokenBalance` for balance checks
- `useCallAnyContract` for custom token interactions

```tsx
// BEFORE (SPL Token transfer):
import { createTransferInstruction, getAssociatedTokenAddress } from "@solana/spl-token";

const fromAta = await getAssociatedTokenAddress(usdcMint, senderPublicKey);
const toAta = await getAssociatedTokenAddress(usdcMint, recipientPublicKey);
// Check if toAta exists, create if not...
const tx = new Transaction().add(
  createTransferInstruction(fromAta, toAta, senderPublicKey, 10_000_000)
);
```

```tsx
// AFTER (Chipi):
import { useTransfer } from "@chipi-stack/nextjs";

const { mutateAsync: transfer } = useTransfer();
await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: 10,
  tokenAddress: "USDC",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No ATA lookup. No ATA creation. No mint address. Just transfer.
```

**Decimal handling:**

| Token | Solana Decimals | StarkNet Decimals | Chipi SDK |
|-------|----------------|-------------------|-----------|
| USDC | 6 (same) | 6 | Pass human-readable (e.g., `10` for $10) |
| SOL/ETH | 9 (lamports) | 18 (wei) | Pass human-readable (e.g., `0.1`) |
| Custom | Varies | Varies | Use `otherToken.decimals` |

## Step 6: Replace Auth

**Remove:**
- Wallet-based auth (signMessage → verify signature with bs58)
- "Connect Wallet" as login
- Wallet state management
- bs58 / nacl signature verification

**Replace with:**
- Standard auth (Clerk/Firebase/Supabase) + Chipi wallet
- Passkey = auth + wallet in one step
- Users sign in with email/Google/passkey, wallet is created automatically

```tsx
// BEFORE (Solana wallet signing):
import bs58 from "bs58";

const message = new TextEncoder().encode(`Sign in: ${nonce}`);
const signature = await wallet.signMessage(message);
// Send to backend: { publicKey, signature, message }
// Backend verifies with nacl.sign.detached.verify()
```

```tsx
// AFTER (Chipi) — standard auth
import { useUser } from "@clerk/nextjs";
const { user, isSignedIn } = useUser();
// User signs in with email/Google/passkey — wallet is separate
// No signature verification needed
```

> **Why separate auth from wallet:** On Solana, your wallet IS your identity. On Chipi, your auth provider handles identity and Chipi handles the wallet. This means users can recover access through email, not seed phrases.

## Step 7: Handle Custom Programs

If you have Anchor or native Solana programs with custom logic:

**Option A: Find StarkNet equivalent**
Many DeFi protocols, NFT standards, and utilities have StarkNet versions. Check the ecosystem.

**Option B: Rewrite in Cairo and deploy**
- Cairo is StarkNet's native language (like Rust for Solana, but ZK-native)
- Use Scarb for project management (like Cargo), Starknet Foundry for testing
- Load the `chipi-custom-contracts` skill for integration guidance
- Once deployed, call via `useCallAnyContract`

```tsx
// BEFORE (Anchor program call):
const tx = await program.methods
  .myFunction(arg1, arg2)
  .accounts({ user: wallet.publicKey, dataAccount: pda, systemProgram: SystemProgram.programId })
  .rpc();

// AFTER (Chipi):
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [{
    contractAddress: "0x_STARKNET_CONTRACT",
    entrypoint: "my_function",
    calldata: [arg1, arg2],
  }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No IDL. No provider. No account constraints. Gasless.
```

**Option C: Use Chipi's built-in features**
If the program just does transfers, approvals, staking, or marketplace operations — Chipi has hooks for all of these. No custom contract needed.

## Step 8: Address Format

Solana uses Base58-encoded 32-byte public keys. StarkNet uses hex-encoded felt252 values.

| Property | Solana | StarkNet |
|----------|--------|----------|
| Format | Base58 (32 bytes) | 0x + 64 hex chars |
| Example | `7EcDhSYGxXyscszYEp35KHN8sADaQT4P...` | `0x049d3657...` |
| Validation | Base58 alphabet, 32-44 chars | `/^0x[0-9a-fA-F]{64}$/` |

**Common mistake:** Using a Solana Base58 address on StarkNet. Always convert user-facing addresses.

## Step 9: Bridge Assets

**LayerSwap is the primary direct Solana → StarkNet bridge.** Other options like StarkGate, rhino.fi, and Orbiter may also support cross-chain transfers — check their current route availability.

See `references/solana-token-bridge-guide.md` for the full bridge flow.

Quick path:
1. User goes to layerswap.io
2. Source: Solana, Destination: StarkNet
3. Enters their Chipi wallet address (from `wallet.publicKey`)
4. Sends tokens on Solana
5. Tokens arrive on StarkNet in 2-10 minutes

Alternative: Solana → CEX (Binance, Coinbase) → StarkNet withdrawal via LayerSwap.

## Step 10: Test Migration

1. **Verify all token operations work** — transfers, balances (no ATA issues)
2. **Verify wallet creation** — passkey works, no Phantom prompts
3. **Verify custom contract calls** — Anchor logic correctly mapped to Cairo
4. **Verify gasless UX** — users never see gas prompts or SOL requirements
5. **Verify address format** — no Base58 addresses leaking into StarkNet calls
6. **Verify decimal handling** — USDC amounts match (both 6 decimals, but verify)
7. **Compare: old UX vs new UX** — should be dramatically simpler

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| Base58 address rejected | Using Solana address format on StarkNet | Convert to 0x + 64 hex chars |
| "ATA not found" patterns in code | Leftover Solana ATA logic | Remove — StarkNet uses ERC-20 `balanceOf` |
| PDA derivation code fails | No PDA concept on StarkNet | Use contract storage instead |
| Anchor IDL import errors | Anchor is Solana-only | Map IDL to Cairo ABI, use `useCallAnyContract` |
| Wallet-adapter imports fail | Removed wallet-adapter but missed imports | Search for all `@solana/` imports and remove |
| Token decimal mismatch | SOL (9 decimals) vs ETH (18 decimals) | Use SDK's human-readable amount format |
| "Fee payer" errors | Leftover Solana gas logic | Remove — all Chipi txs are gasless |
| Transaction too large | Solana 1232-byte limit mindset | StarkNet has no hard limit, batch freely |
| Confirmation timeout | Using Solana confirmation patterns | Chipi hooks handle confirmation internally |
| "Program not found" | Solana program ID used as StarkNet address | Deploy Cairo contract, use new address |

## Hooks Used

All hooks from `@chipi-stack/nextjs` (or `@chipi-stack/chipi-react` / `@chipi-stack/chipi-expo`):
- `useCreateWallet` — replaces Phantom wallet creation
- `useChipiWallet` — replaces `useWallet` from wallet-adapter
- `useTransfer` — replaces SPL token transfers
- `useGetTokenBalance` — replaces ATA balance lookups
- `useCallAnyContract` — replaces Anchor program calls
- `useChipiSession` — replaces manual session management (no Solana equivalent)

## Reference Files

For detailed code-level mappings, see:
- `references/solana-account-model-mapping.md` — Account model, PDAs, rent, CPI deep dive
- `references/solana-code-migration-patterns.md` — Web3.js, Anchor, wallet-adapter, SPL patterns
- `references/solana-common-errors.md` — 15+ common migration errors with fixes
- `references/solana-token-bridge-guide.md` — LayerSwap bridge flow for Solana → StarkNet

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key migration-specific UI rules:
- Show a "migration checklist" component with checkmarks as each layer is replaced
- Use before/after code comparisons to validate the migration
- Display chain indicators ("Migrated from Solana" badge) during testing

## What's Next

- `chipi-session-keys` — Add frictionless transactions (one auth, many txs)
- `chipi-custom-contracts` — Call any StarkNet contract through Chipi
- `chipi-debug` — Diagnose errors, validate addresses, troubleshoot failed transactions
