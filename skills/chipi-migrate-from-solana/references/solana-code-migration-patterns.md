# Solana Code Migration Patterns

Expanded code-level mappings from @solana/web3.js, Anchor, wallet-adapter, and SPL tokens to Chipi SDK equivalents.

## Web3.js v1 (Legacy) → Chipi

### Connection & Wallet

```tsx
// BEFORE (@solana/web3.js v1)
import { Connection, PublicKey, clusterApiUrl } from "@solana/web3.js";
import { useWallet } from "@solana/wallet-adapter-react";

const connection = new Connection(clusterApiUrl("mainnet-beta"));
const { publicKey, signTransaction, connected } = useWallet();
const balance = await connection.getBalance(publicKey);
const solBalance = balance / LAMPORTS_PER_SOL;
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
// No connection object. No cluster config. No wallet adapter.
```

### Send SOL → Send Token

```tsx
// BEFORE
import { Transaction, SystemProgram, LAMPORTS_PER_SOL } from "@solana/web3.js";

const tx = new Transaction().add(
  SystemProgram.transfer({
    fromPubkey: wallet.publicKey,
    toPubkey: new PublicKey(recipient),
    lamports: amount * LAMPORTS_PER_SOL,
  })
);
const { blockhash } = await connection.getLatestBlockhash();
tx.recentBlockhash = blockhash;
tx.feePayer = wallet.publicKey;
const sig = await sendTransaction(tx, connection);
await connection.confirmTransaction(sig, "finalized");
```

```tsx
// AFTER (Chipi)
import { useTransfer } from "@chipi-stack/nextjs";

const { mutateAsync: transfer } = useTransfer();
await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: amount,
  tokenAddress: "ETH", // or "USDC", "STRK"
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No blockhash. No fee payer. No confirmation polling.
```

## Web3.js v2 (Pipe-Based) → Chipi

Web3.js v2 uses a pipe-based API pattern. If the app uses this, the migration is the same — just different syntax to remove.

```tsx
// BEFORE (@solana/kit — formerly @solana/web3.js v2)
import {
  createTransactionMessage,
  appendTransactionMessageInstructions,
  setTransactionMessageFeePayerSigner,
  setTransactionMessageLifetimeUsingBlockhash,
} from "@solana/kit";
import { getTransferInstruction } from "@solana-program/token";

const message = createTransactionMessage({ version: 0 });
const withInstructions = appendTransactionMessageInstructions(
  [getTransferInstruction({ source: fromAta, destination: toAta, authority, amount })],
  message,
);
const withFeePayer = setTransactionMessageFeePayerSigner(walletSigner, withInstructions);
const withLifetime = setTransactionMessageLifetimeUsingBlockhash(blockhash, withFeePayer);
const sig = await sendAndConfirmTransaction(withLifetime);
```

```tsx
// AFTER (Chipi) — same as v1 migration
const { mutateAsync: transfer } = useTransfer();
await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: 10,
  tokenAddress: "USDC",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Anchor Deep Mapping

### Program → Contract

```text
Anchor (Solana):                    Cairo (StarkNet):
├─ lib.rs (program)                ├─ lib.cairo (contract)
├─ instructions/                   ├─ (functions in contract)
├─ state/ (account structs)        ├─ (storage structs)
├─ Anchor.toml                     ├─ Scarb.toml
├─ programs/                       ├─ src/
└─ target/idl/program.json         └─ target/dev/contract.sierra.json
```

### IDL → ABI

```json
// ANCHOR IDL (target/idl/my_program.json)
{
  "instructions": [
    {
      "name": "initialize",
      "accounts": [
        { "name": "user", "isMut": true, "isSigner": true },
        { "name": "userData", "isMut": true, "isSigner": false },
        { "name": "systemProgram", "isMut": false, "isSigner": false }
      ],
      "args": [
        { "name": "name", "type": "string" },
        { "name": "amount", "type": "u64" }
      ]
    }
  ]
}
```

```json
// CAIRO ABI (target/dev/contract.sierra.json — simplified)
[
  {
    "type": "function",
    "name": "initialize",
    "inputs": [
      { "name": "name", "type": "core::felt252" },
      { "name": "amount", "type": "core::integer::u64" }
    ],
    "outputs": [],
    "state_mutability": "external"
  }
]
```

Key differences:
- **No `accounts` array** — Cairo contracts access their own storage, no need to declare accounts
- **No `systemProgram`** — StarkNet doesn't have system programs
- **No `isMut` / `isSigner`** — Access control is handled by `get_caller_address()` in Cairo
- **Type mapping:** `u64` → `u64`, `string` → `felt252` or `ByteArray`, `Pubkey` → `ContractAddress`

### Anchor Methods → useCallAnyContract

```tsx
// BEFORE (Anchor client)
import { Program, AnchorProvider } from "@coral-xyz/anchor";
import { BN } from "@coral-xyz/anchor";
import { IDL, MyProgram } from "./idl/my_program";

const provider = new AnchorProvider(connection, wallet, {});
const program = new Program<MyProgram>(IDL, programId, provider);

// Call with automatic account resolution
const tx = await program.methods
  .initialize("Alice", new BN(1000))
  .accounts({
    user: wallet.publicKey,
    userData: userDataPda,
    systemProgram: SystemProgram.programId,
  })
  .rpc();
```

```tsx
// AFTER (Chipi)
import { useCallAnyContract } from "@chipi-stack/nextjs";

const { mutateAsync: callContract } = useCallAnyContract();
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [{
    contractAddress: "0x_CAIRO_CONTRACT",
    entrypoint: "initialize",
    calldata: [
      "0x416c696365",  // "Alice" as felt252 (hex-encoded short string)
      "1000", "0",     // u64 as felt252 (or u256 as [low, high])
    ],
  }],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No IDL file needed. No provider. No account resolution.
```

### Anchor Account Constraints → Cairo Access Control

```rust
// ANCHOR — account constraints
#[derive(Accounts)]
pub struct UpdateData<'info> {
    #[account(mut, has_one = authority)]
    pub data_account: Account<'info, UserData>,
    pub authority: Signer<'info>,
}
```

```cairo
// CAIRO — access control in function body
#[external(v0)]
fn update_data(ref self: ContractState, new_value: u256) {
    let caller = get_caller_address();
    assert(caller == self.authority.read(), 'Not authorized');
    self.data.write(new_value);
}
```

## Full Wallet-Adapter Removal

### Provider Stack

```tsx
// REMOVE ALL OF THIS:
import { ConnectionProvider, WalletProvider } from "@solana/wallet-adapter-react";
import { WalletModalProvider } from "@solana/wallet-adapter-react-ui";
import { PhantomWalletAdapter, SolflareWalletAdapter, BackpackWalletAdapter } from "@solana/wallet-adapter-wallets";
import "@solana/wallet-adapter-react-ui/styles.css";

const wallets = useMemo(() => [
  new PhantomWalletAdapter(),
  new SolflareWalletAdapter(),
  new BackpackWalletAdapter(),
], []);

function Providers({ children }) {
  return (
    <ConnectionProvider endpoint={clusterApiUrl("mainnet-beta")}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          {children}
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}
```

```tsx
// REPLACE WITH (in layout.tsx):
// ChipiProvider wraps the app — see chipi-wallet-setup skill
// No connection provider. No wallet provider. No modal provider.
// Just the Chipi provider + your auth provider (Clerk/Firebase/Supabase).
```

### Wallet Hooks

```tsx
// REMOVE:
import { useWallet, useConnection } from "@solana/wallet-adapter-react";
const { publicKey, connected, signTransaction, signMessage, disconnect } = useWallet();
const { connection } = useConnection();

// REPLACE WITH:
import { useChipiWallet } from "@chipi-stack/nextjs";
const { wallet, hasWallet } = useChipiWallet();
const address = wallet?.publicKey;
// No "connected" state — wallet either exists or doesn't
// No "signTransaction" — Chipi hooks handle signing via passkey
// No "disconnect" — user signs out via auth provider
```

### UI Components

```tsx
// REMOVE:
import { WalletMultiButton, WalletDisconnectButton } from "@solana/wallet-adapter-react-ui";
<WalletMultiButton />
<WalletDisconnectButton />

// REPLACE WITH:
// Use Chipi's create-wallet-dialog and wallet-summary components
// Call get_component_code("create-wallet-dialog") and get_component_code("wallet-summary")
```

## SPL Token Expanded

### createAssociatedTokenAccount → Not Needed

```tsx
// BEFORE (Solana) — must create ATA before receiving tokens
import { createAssociatedTokenAccountInstruction, getAssociatedTokenAddress } from "@solana/spl-token";

const ata = await getAssociatedTokenAddress(mint, recipientPubkey);
const ataInfo = await connection.getAccountInfo(ata);
if (!ataInfo) {
  const tx = new Transaction().add(
    createAssociatedTokenAccountInstruction(
      payer,           // fee payer
      ata,             // ATA address
      recipientPubkey, // owner
      mint             // token mint
    )
  );
  await sendTransaction(tx, connection);
}
```

```tsx
// AFTER (Chipi) — no ATA concept
// Just transfer. The recipient's balance updates automatically.
// ERC-20 standard: balanceOf(address) — no separate token accounts.
await transfer({
  recipientAddress: recipient,
  amount: 10,
  tokenAddress: "USDC",
  // ... other params
});
```

### getOrCreateAssociatedTokenAccount → Not Needed

```tsx
// BEFORE
import { getOrCreateAssociatedTokenAccount } from "@solana/spl-token";

const tokenAccount = await getOrCreateAssociatedTokenAccount(
  connection,
  payer,
  mint,
  owner
);
const balance = tokenAccount.amount;
```

```tsx
// AFTER (Chipi)
const { data: balance } = useGetTokenBalance({
  walletAddress: ownerAddress,
  tokenAddress: "USDC",
});
// One hook. No account creation. No payer needed.
```

## Versioned Transactions → StarkNet Multicall

```tsx
// BEFORE (Solana v0 transaction with lookup tables)
import { VersionedTransaction, TransactionMessage } from "@solana/web3.js";

const messageV0 = new TransactionMessage({
  payerKey: wallet.publicKey,
  recentBlockhash: blockhash,
  instructions: [instruction1, instruction2, instruction3],
}).compileToV0Message(lookupTableAccounts);

const tx = new VersionedTransaction(messageV0);
const signed = await wallet.signTransaction(tx);
await connection.sendRawTransaction(signed.serialize());
```

```tsx
// AFTER (Chipi) — native multicall
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [
    { contractAddress: addr1, entrypoint: "fn1", calldata: [...] },
    { contractAddress: addr2, entrypoint: "fn2", calldata: [...] },
    { contractAddress: addr3, entrypoint: "fn3", calldata: [...] },
  ],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No versioned transactions. No lookup tables. No message compilation.
// StarkNet supports multicall natively via account abstraction.
```

## Priority Fees → Gasless

```tsx
// BEFORE (Solana priority fees)
import { ComputeBudgetProgram } from "@solana/web3.js";

const tx = new Transaction()
  .add(ComputeBudgetProgram.setComputeUnitPrice({ microLamports: 50000 }))
  .add(ComputeBudgetProgram.setComputeUnitLimit({ units: 200000 }))
  .add(yourInstruction);
```

```tsx
// AFTER (Chipi)
// Delete all ComputeBudgetProgram code.
// Chipi handles gas sponsorship. Users pay nothing.
// No compute unit pricing. No priority fee estimation.
```

## NPM Packages to Remove

When migrating, remove these from `package.json`:

```json
{
  "dependencies": {
    // REMOVE ALL OF THESE:
    "@solana/web3.js": "^1.x",
    "@solana/spl-token": "^0.x",
    "@solana/wallet-adapter-react": "^0.x",
    "@solana/wallet-adapter-react-ui": "^0.x",
    "@solana/wallet-adapter-wallets": "^0.x",
    "@coral-xyz/anchor": "^0.x",
    "bs58": "^5.x",
    "@metaplex-foundation/js": "^0.x",

    // ADD:
    "@chipi-stack/nextjs": "latest"
    // (or @chipi-stack/chipi-react or @chipi-stack/chipi-expo)
  }
}
```

Run:
```bash
npm uninstall @solana/web3.js @solana/spl-token @solana/wallet-adapter-react @solana/wallet-adapter-react-ui @solana/wallet-adapter-wallets @coral-xyz/anchor bs58
npm install @chipi-stack/nextjs
```
