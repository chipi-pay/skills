# Solana to StarkNet + Chipi — Program-Level Mapping

Detailed mapping of @solana/web3.js and Anchor patterns to their Chipi SDK equivalents.

## Connection & Wallet

### @solana/web3.js

```tsx
// BEFORE
import { Connection, PublicKey, clusterApiUrl } from "@solana/web3.js";
import { useWallet } from "@solana/wallet-adapter-react";

const connection = new Connection(clusterApiUrl("mainnet-beta"));
const { publicKey, signTransaction } = useWallet();
const balance = await connection.getBalance(publicKey);
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

### Phantom / Wallet Adapter

```tsx
// BEFORE
import { WalletMultiButton } from "@solana/wallet-adapter-react-ui";
// In JSX:
<WalletMultiButton />
// User installs Phantom, creates account, connects to site
```

```tsx
// AFTER (Chipi)
import { useCreateWallet } from "@chipi-stack/nextjs";

const { mutateAsync: createWallet } = useCreateWallet();
// User taps fingerprint — wallet created. No extension needed.
await createWallet({
  walletType: "CHIPI",
  usePasskey: true,
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
```

## Token Operations

### SPL Token Transfer

```tsx
// BEFORE (Solana)
import { createTransferInstruction, getAssociatedTokenAddress } from "@solana/spl-token";
import { Transaction } from "@solana/web3.js";

const fromAta = await getAssociatedTokenAddress(usdcMint, senderPublicKey);
const toAta = await getAssociatedTokenAddress(usdcMint, recipientPublicKey);

const tx = new Transaction().add(
  createTransferInstruction(fromAta, toAta, senderPublicKey, 10_000_000) // 10 USDC
);
const signature = await sendTransaction(tx, connection);
await connection.confirmTransaction(signature);
```

```tsx
// AFTER (Chipi)
import { useTransfer } from "@chipi-stack/nextjs";

const { mutateAsync: transfer } = useTransfer();
await transfer({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  recipientAddress: recipient,
  amount: 10,  // Human-readable
  tokenAddress: "USDC",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No ATA management. No manual instruction building. Gasless.
```

### SPL Token Balance

```tsx
// BEFORE (Solana)
import { getAccount, getAssociatedTokenAddress } from "@solana/spl-token";

const ata = await getAssociatedTokenAddress(usdcMint, publicKey);
const account = await getAccount(connection, ata);
const balance = Number(account.amount) / 1_000_000; // USDC has 6 decimals
```

```tsx
// AFTER (Chipi)
import { useGetTokenBalance } from "@chipi-stack/nextjs";

const { data: balance } = useGetTokenBalance({
  walletAddress: address,
  tokenAddress: "USDC",
});
// Already formatted. No ATA lookup. No manual decimal division.
```

## Program Interactions

### Anchor Program Call

```tsx
// BEFORE (Anchor)
import { Program, AnchorProvider } from "@coral-xyz/anchor";
import { IDL } from "./idl/my_program";

const provider = new AnchorProvider(connection, wallet, {});
const program = new Program(IDL, programId, provider);

const tx = await program.methods
  .myFunction(arg1, arg2)
  .accounts({
    user: wallet.publicKey,
    dataAccount: dataPda,
    systemProgram: SystemProgram.programId,
  })
  .rpc();
```

```tsx
// AFTER (Chipi) — call the equivalent Cairo contract
import { useCallAnyContract } from "@chipi-stack/nextjs";

const { mutateAsync: callContract } = useCallAnyContract();
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [
    {
      contractAddress: "0x_STARKNET_CONTRACT",
      entrypoint: "my_function",
      calldata: [arg1, arg2],  // felt252 array
    },
  ],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// No IDL file. No provider setup. No account constraints. Gasless.
```

### PDA (Program Derived Address)

```tsx
// BEFORE (Solana)
const [pda, bump] = PublicKey.findProgramAddressSync(
  [Buffer.from("user_data"), wallet.publicKey.toBuffer()],
  programId
);
// PDA is used as a data account
```

```tsx
// AFTER (StarkNet) — no PDA concept
// StarkNet contracts store data in contract storage directly
// Access pattern: contract_address + storage_variable_name
// No need to derive addresses — just read from the contract
```

## Transaction Patterns

### Multiple Instructions (Batching)

```tsx
// BEFORE (Solana) — multiple instructions in one transaction
const tx = new Transaction()
  .add(approveInstruction)
  .add(swapInstruction)
  .add(transferInstruction);
const sig = await sendTransaction(tx, connection);
```

```tsx
// AFTER (Chipi) — multiple calls in one transaction (native)
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [
    { contractAddress: token, entrypoint: "approve", calldata: [...] },
    { contractAddress: dex, entrypoint: "swap", calldata: [...] },
    { contractAddress: token, entrypoint: "transfer", calldata: [...] },
  ],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// Same pattern — array of calls. Both chains support batching natively.
```

### Transaction Confirmation

```tsx
// BEFORE (Solana)
const signature = await sendTransaction(tx, connection);
const confirmation = await connection.confirmTransaction(signature, "finalized");
if (confirmation.value.err) {
  throw new Error("Transaction failed");
}
```

```tsx
// AFTER (Chipi) — built into the hook
const result = await transfer({ ... });
// Result includes transaction hash and status
// Chipi hooks handle confirmation internally
```

## Auth Patterns

### Wallet-Based Auth (Solana)

```tsx
// BEFORE
import bs58 from "bs58";

const message = new TextEncoder().encode(`Sign in: ${nonce}`);
const signature = await wallet.signMessage(message);
// Send signature + publicKey to backend for verification
```

```tsx
// AFTER (Chipi) — standard auth
// Use Clerk, Firebase, or Supabase
// Wallet is created after auth, not used for auth
import { useUser } from "@clerk/nextjs";
const { user, isSignedIn } = useUser();
```

## Key Differences

| Concept | Solana | StarkNet + Chipi |
|---------|--------|------------------|
| Address format | Base58 (32 bytes) | 0x + 64 hex chars (felt252) |
| Token accounts | ATAs (associated token accounts) | ERC-20 standard (no ATAs) |
| Gas | ~$0.001 SOL per tx | Gasless (Chipi sponsors) |
| Programs vs Contracts | Programs are stateless | Contracts have storage |
| Data storage | PDAs / Account data | Contract storage variables |
| Transaction size | 1232 bytes max | No hard limit |
| Block time | ~400ms | ~15-30 seconds |
| Finality | ~30 seconds (finalized) | ~15-30 seconds |
| Account model | Account-based (PDAs) | Contract storage |
| Rent | Rent-exempt balance needed | No rent concept |
| Wallet standard | Wallet-adapter | Passkey (no extension) |
