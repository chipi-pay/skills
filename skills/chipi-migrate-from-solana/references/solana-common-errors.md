# Solana Migration — Common Errors and Fixes

15+ errors frequently encountered when migrating from Solana to StarkNet + Chipi.

## Wallet & Auth Errors

### 1. "Cannot find module '@solana/wallet-adapter-react'"

| | |
|---|---|
| **Why on Solana** | You removed the package but still have imports |
| **StarkNet equivalent** | N/A — different wallet system |
| **Fix** | Search codebase for all `@solana/` imports and remove. Replace `useWallet` with `useChipiWallet` from `@chipi-stack/nextjs`. |

### 2. "window.solana is undefined" / "Phantom not found"

| | |
|---|---|
| **Why on Solana** | App tries to detect Phantom browser extension |
| **StarkNet equivalent** | N/A — Chipi uses passkeys, not extensions |
| **Fix** | Remove all `window.solana` / `window.phantom` checks. Chipi wallets work without any browser extension. |

### 3. Wallet-adapter "WalletNotConnectedError"

| | |
|---|---|
| **Why on Solana** | User hasn't connected their wallet via the adapter |
| **StarkNet equivalent** | Check `hasWallet` from `useChipiWallet` |
| **Fix** | Replace wallet connection check: `if (!connected)` → `if (!hasWallet)`. Create wallet with `useCreateWallet` if none exists. |

### 4. Passkey denied / "NotAllowedError: The operation was not allowed"

| | |
|---|---|
| **Why on Solana** | Phantom rejection = user clicked "Cancel" |
| **StarkNet equivalent** | User cancelled the passkey prompt (fingerprint/face) |
| **Fix** | Show a user-friendly message: "Authentication cancelled. Please try again and confirm with your fingerprint." Don't retry automatically. |

## Transaction Errors

### 5. "BlockhashNotFound" / "Transaction was not confirmed in X seconds"

| | |
|---|---|
| **Why on Solana** | Blockhash expired before transaction was confirmed |
| **StarkNet equivalent** | N/A — Chipi handles transaction lifecycle |
| **Fix** | Remove all `getLatestBlockhash`, `recentBlockhash`, and confirmation polling code. Chipi hooks handle confirmation internally. |

### 6. "Transaction too large" mindset

| | |
|---|---|
| **Why on Solana** | Solana has a 1232-byte transaction size limit |
| **StarkNet equivalent** | No hard transaction size limit |
| **Fix** | You can batch more calls in StarkNet's `calls[]` array than Solana's instruction list. Don't artificially split transactions. |

### 7. "Insufficient funds for fee" / fee payer errors

| | |
|---|---|
| **Why on Solana** | User doesn't have enough SOL for transaction fees |
| **StarkNet equivalent** | N/A — all Chipi transactions are gasless |
| **Fix** | Remove all fee payer logic, SOL balance checks for gas, and `feePayer` assignments. Users never need gas tokens with Chipi. |

### 8. "Transaction simulation failed"

| | |
|---|---|
| **Why on Solana** | Transaction would fail, detected during simulation |
| **StarkNet equivalent** | Chipi SDK returns error with details |
| **Fix** | Remove `simulateTransaction` calls. Chipi hooks return errors directly. Check the error message for specific issues (wrong calldata, insufficient balance, etc.). |

## Token Errors

### 9. "Account does not exist" (ATA-related)

| | |
|---|---|
| **Why on Solana** | Associated Token Account doesn't exist for the recipient |
| **StarkNet equivalent** | N/A — ERC-20 tokens don't use separate accounts |
| **Fix** | Remove all ATA logic (`getAssociatedTokenAddress`, `createAssociatedTokenAccountInstruction`, `getOrCreateAssociatedTokenAccount`). Just use `useTransfer` with the recipient address. |

### 10. Token decimal mismatch (SOL 9 decimals vs ETH 18 decimals)

| | |
|---|---|
| **Why on Solana** | SOL uses 9 decimals (lamports), USDC uses 6 |
| **StarkNet equivalent** | ETH uses 18 decimals (wei), USDC uses 6, STRK uses 18 |
| **Fix** | Use Chipi SDK's human-readable amount format. Pass `amount: 10` for $10 USDC. The SDK handles decimal conversion. Don't manually multiply by `10^n`. |

### 11. "Invalid mint" / SPL Token errors

| | |
|---|---|
| **Why on Solana** | Wrong token mint address or invalid SPL token program |
| **StarkNet equivalent** | Use `tokenAddress: "USDC"` or the full StarkNet contract address |
| **Fix** | Replace Solana mint addresses with StarkNet token addresses. USDC on StarkNet: `0x053c91253bc9682c04929ca02ed00b3e423f6710d2ee7e0d5ebb06f3ecf368a8`. |

## Address & Format Errors

### 12. Base58 address used on StarkNet

| | |
|---|---|
| **Why on Solana** | Solana addresses are Base58-encoded (e.g., `7EcDhSYGx...`) |
| **StarkNet equivalent** | StarkNet addresses are hex (0x + 64 chars) |
| **Fix** | Replace all Base58 addresses with StarkNet hex addresses. Validate with regex: `/^0x[0-9a-fA-F]{64}$/`. If a user pastes a Solana address, show an error: "This looks like a Solana address. StarkNet addresses start with 0x." |

### 13. Address length confusion (40 chars vs 64 chars)

| | |
|---|---|
| **Why on Solana** | Developers may confuse EVM (40 hex) and StarkNet (64 hex) formats |
| **StarkNet equivalent** | Always 0x + 64 hex characters |
| **Fix** | StarkNet addresses are longer than EVM. If you see `0x` + 40 chars, that's an EVM address. StarkNet needs `0x` + 64 chars (with leading zeros). |

## Program/Contract Errors

### 14. "PDA derivation" patterns in migrated code

| | |
|---|---|
| **Why on Solana** | Code still contains `findProgramAddressSync` calls |
| **StarkNet equivalent** | N/A — no PDA concept |
| **Fix** | Replace PDA derivation with direct contract storage reads. See `references/solana-account-model-mapping.md` for pattern-by-pattern mapping. |

### 15. Anchor IDL import errors

| | |
|---|---|
| **Why on Solana** | Code imports IDL files for Anchor program interaction |
| **StarkNet equivalent** | Cairo ABI (Sierra JSON), but Chipi SDK doesn't need it |
| **Fix** | Remove IDL imports. When using `useCallAnyContract`, you pass entrypoint name and calldata directly — no IDL/ABI file needed on the frontend. |

### 16. "Program not found" / wrong program ID

| | |
|---|---|
| **Why on Solana** | Using a Solana program ID as a StarkNet contract address |
| **StarkNet equivalent** | Use the deployed Cairo contract address |
| **Fix** | Solana program IDs (Base58) are not StarkNet contract addresses (hex). Deploy your Cairo contract and use its StarkNet address. |

## Serialization Errors

### 17. Borsh serialization errors

| | |
|---|---|
| **Why on Solana** | Borsh is Solana's binary serialization format |
| **StarkNet equivalent** | felt252 arrays (calldata) — SDK handles encoding |
| **Fix** | Remove all Borsh imports (`borsh`, `@coral-xyz/borsh`). Chipi SDK handles calldata encoding. Pass values as strings in the `calldata` array. |

### 18. u256 encoding mistakes

| | |
|---|---|
| **Why on Solana** | Solana uses u64 natively; StarkNet u256 needs [low, high] split |
| **StarkNet equivalent** | u256 = two felt252 values: [low_128_bits, high_128_bits] |
| **Fix** | For amounts under 2^128 (almost all practical values), pass `[amount, "0"]`. Example: 1000000 USDC raw = `["1000000", "0"]`. |

## Quick Diagnosis Checklist

If something isn't working after migration, check these in order:

1. **All `@solana/` imports removed?** — Search for `@solana/` and `@coral-xyz/`
2. **Wallet-adapter providers removed?** — No ConnectionProvider, WalletProvider, WalletModalProvider
3. **Using hex addresses (not Base58)?** — All addresses should start with `0x`
4. **No ATA logic remaining?** — No getAssociatedTokenAddress, createAssociatedTokenAccountInstruction
5. **No PDA derivation remaining?** — No findProgramAddressSync
6. **No fee payer / gas logic?** — No feePayer, getLatestBlockhash, ComputeBudgetProgram
7. **No Borsh serialization?** — No borsh.serialize/deserialize
8. **Using human-readable amounts?** — `amount: 10` not `amount: 10000000`
9. **ChipiProvider wrapping the app?** — Check layout.tsx
10. **Environment variables set?** — NEXT_PUBLIC_CHIPI_API_KEY exists
