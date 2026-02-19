# Solana Account Model → StarkNet Contract Model

Deep mapping of Solana's account-based architecture to StarkNet's contract storage model.

## The Core Difference

```
SOLANA                                    STARKNET
┌─────────────┐                          ┌─────────────────┐
│   Program    │ ← stateless code        │    Contract      │
│  (deployed)  │                          │  ┌────────────┐ │
└──────┬───────┘                          │  │  Storage    │ │
       │ operates on                      │  │  - owner    │ │
       ▼                                  │  │  - balance  │ │
┌─────────────┐                          │  │  - data     │ │
│   Account    │ ← data storage          │  └────────────┘ │
│  (PDA/owned) │                          │  ┌────────────┐ │
│  - owner     │                          │  │  Functions  │ │
│  - data[]    │                          │  │  - read()   │ │
│  - lamports  │                          │  │  - write()  │ │
└──────────────┘                          │  └────────────┘ │
                                          └─────────────────┘
```

**Solana:** Programs are stateless. Data lives in accounts. Programs specify which accounts they can read/write. Accounts are passed into instructions.

**StarkNet:** Contracts contain both code and storage. No separate data accounts. Storage is accessed directly within the contract.

## PDA (Program Derived Address) → Contract Storage

PDAs are Solana's mechanism for letting programs own data. They don't exist on StarkNet.

### PDA Derivation Code

```tsx
// SOLANA — derive a PDA for user data
const [userDataPda, bump] = PublicKey.findProgramAddressSync(
  [
    Buffer.from("user_data"),
    wallet.publicKey.toBuffer(),
  ],
  programId
);

// Read data from the PDA
const accountInfo = await connection.getAccountInfo(userDataPda);
const userData = borsh.deserialize(UserDataSchema, accountInfo.data);
```

### StarkNet Equivalent — Contract Storage

```cairo
// CAIRO CONTRACT — data stored directly in contract
#[storage]
struct Storage {
    user_data: Map<ContractAddress, UserData>,
}

#[external(v0)]
fn get_user_data(self: @ContractState, user: ContractAddress) -> UserData {
    self.user_data.read(user)
}
```

```tsx
// FRONTEND — read from contract via Chipi or starknet.js
// No PDA derivation needed — just call the contract
const result = await callContract({
  calls: [{
    contractAddress: "0x_CONTRACT",
    entrypoint: "get_user_data",
    calldata: [userAddress],
  }],
});
```

### Common PDA Patterns and Their StarkNet Equivalents

| Solana PDA Pattern | Seeds | StarkNet Equivalent |
|---|---|---|
| User data account | `["user_data", user_pubkey]` | `Map<ContractAddress, UserData>` in storage |
| Token vault | `["vault", mint, pool_id]` | Storage variable or separate vault contract |
| Config/settings | `["config"]` | Storage variables (owner-only write) |
| Metadata | `["metadata", mint]` | `Map<u256, Metadata>` in storage |
| Escrow | `["escrow", order_id]` | `Map<u256, EscrowData>` in storage |
| Tick data (DeFi) | `["tick", pool, tick_index]` | Nested mapping in storage |

### Key Insight

Every PDA pattern maps to a storage variable or mapping in Cairo. The "derivation" step disappears entirely:

```
Solana:    seeds → PDA address → fetch account → deserialize
StarkNet:  contract.storage_variable.read(key) → done
```

## Rent Exemption → Not Needed

### Solana Rent Math

```tsx
// SOLANA — calculate rent-exempt minimum
const rentExempt = await connection.getMinimumBalanceForRentExemption(
  accountDataSize  // Size in bytes
);
// Must fund the account with at least this many lamports
// Currently ~0.002 SOL for most accounts (~6960 bytes @ 19.055 lamports/byte/year × 2 years)
```

### StarkNet — No Equivalent

```tsx
// STARKNET — no rent concept
// Deploy a contract → it stays deployed forever
// Store data → it stays stored forever
// No minimum balance requirements for data storage
// No periodic rent payments
// No "garbage collection" of underfunded accounts
```

**Migration action:** Remove all `getMinimumBalanceForRentExemption` calls and rent-related logic. Simply delete it.

## CPI (Cross-Program Invocation) → Contract-to-Contract Calls

### Solana CPI

```rust
// SOLANA (Anchor) — invoke another program
use anchor_lang::prelude::*;

pub fn swap(ctx: Context<Swap>, amount: u64) -> Result<()> {
    // CPI to Token Program
    let cpi_accounts = Transfer {
        from: ctx.accounts.user_token.to_account_info(),
        to: ctx.accounts.pool_token.to_account_info(),
        authority: ctx.accounts.user.to_account_info(),
    };
    let cpi_program = ctx.accounts.token_program.to_account_info();
    let cpi_ctx = CpiContext::new(cpi_program, cpi_accounts);
    token::transfer(cpi_ctx, amount)?;
    Ok(())
}
```

### StarkNet Contract-to-Contract Calls

```cairo
// CAIRO — call another contract directly
use starknet::ContractAddress;

#[starknet::interface]
trait IERC20<TContractState> {
    fn transfer_from(ref self: TContractState, sender: ContractAddress, recipient: ContractAddress, amount: u256) -> bool;
}

#[external(v0)]
fn swap(ref self: ContractState, amount: u256) {
    let token = IERC20Dispatcher { contract_address: self.token_address.read() };
    token.transfer_from(get_caller_address(), get_contract_address(), amount);
    // No CPI context. No account constraints. Just call the function.
}
```

### From the Frontend (Chipi)

```tsx
// Batch approve + swap in ONE transaction (native multicall)
await callContract({
  encryptKey: passkeyCredential,
  wallet: userWallet,
  calls: [
    {
      contractAddress: tokenAddress,
      entrypoint: "approve",
      calldata: [swapContractAddress, amountLow, amountHigh],
    },
    {
      contractAddress: swapContractAddress,
      entrypoint: "swap",
      calldata: [amountLow, amountHigh],
    },
  ],
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// On Solana this would be multiple instructions in a Transaction
// On StarkNet via Chipi it's the same pattern — array of calls — but gasless
```

## Sealevel Parallel Execution → ZK Proofs

### Solana — Sealevel Runtime

Solana's Sealevel runtime executes non-overlapping transactions in parallel. This is why transactions must declare accounts upfront — the runtime needs to detect conflicts.

```
Transaction A: [Account 1, Account 2] → can run in parallel with
Transaction B: [Account 3, Account 4] → no overlap

Transaction C: [Account 1, Account 3] → must wait for A or B
```

### StarkNet — ZK Sequencer

StarkNet's sequencer orders transactions, executes them, and generates a ZK proof of validity. No parallel execution model for developers to think about.

```
Transactions → Sequencer → Execution → ZK Proof → L1 Verification
```

**Migration action:** Remove any Solana-specific parallelism optimizations (account ordering, compute budget requests). They don't apply on StarkNet.

## Data Serialization

| Aspect | Solana | StarkNet |
|--------|--------|----------|
| Format | Borsh (Binary Object Representation Serializer for Hashing) | felt252 arrays |
| Integers | u8, u16, u32, u64, u128, i64 | felt252, u8, u16, u32, u64, u128, u256 |
| Strings | Vec<u8> or String (Borsh) | felt252 (short string) or ByteArray |
| Addresses | Pubkey (32 bytes, Base58) | ContractAddress (felt252, hex) |
| Arrays | Vec<T> with length prefix | Array<T> with length prefix in calldata |
| Encoding | Borsh serialize/deserialize | SDK handles felt252 encoding |

**Migration action:** Replace all Borsh serialization with Chipi SDK's calldata encoding (the SDK handles this automatically for standard types).
