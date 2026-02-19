# EVM Migration — Common Errors and Fixes

18 errors frequently encountered when migrating from EVM (ethers.js, viem, wagmi) to StarkNet + Chipi.

## Wallet & Auth Errors

### 1. "Cannot find module 'ethers'" / "Cannot find module 'viem'"

| | |
|---|---|
| **Why on EVM** | You removed the package but still have imports in your codebase |
| **StarkNet equivalent** | N/A — different SDK |
| **Fix** | Search for all `import.*from ["']ethers`, `import.*from ["']viem`, and `import.*from ["']wagmi` and remove. Replace with Chipi hooks from `@chipi-stack/nextjs`. |

### 2. "window.ethereum is undefined" / "No injected provider found"

| | |
|---|---|
| **Why on EVM** | App tries to detect MetaMask or another browser extension |
| **StarkNet equivalent** | N/A — Chipi uses passkeys, not extensions |
| **Fix** | Remove all `window.ethereum` checks, `typeof window.ethereum !== "undefined"` guards, and MetaMask detection logic. Chipi wallets work without any browser extension. |

### 3. MetaMask "User rejected the request" (code 4001)

| | |
|---|---|
| **Why on EVM** | User clicked "Reject" on the MetaMask popup |
| **StarkNet equivalent** | User cancelled the passkey prompt (NotAllowedError) |
| **Fix** | Replace MetaMask error handling (`if (error.code === 4001)`) with passkey cancellation handling. Show: "Authentication cancelled. Please try again and confirm with your fingerprint." |

### 4. ConnectorNotFoundError / "Connector not found"

| | |
|---|---|
| **Why on EVM** | wagmi can't find the configured connector (MetaMask, WalletConnect, etc.) |
| **StarkNet equivalent** | N/A — no connectors in Chipi |
| **Fix** | Remove all wagmi connector config (`injected()`, `metaMask()`, `walletConnect()`). Chipi uses passkeys — no connectors needed. |

## Transaction Errors

### 5. "nonce too low" / "nonce has already been used"

| | |
|---|---|
| **Why on EVM** | Transaction nonce conflicts from rapid or parallel submissions |
| **StarkNet equivalent** | N/A — Chipi handles nonce management internally |
| **Fix** | Remove all manual nonce tracking (`getTransactionCount`, `nonce: n`). Chipi SDK manages nonce sequencing automatically. |

### 6. "insufficient funds for gas * price + value" / "insufficient funds for intrinsic transaction cost"

| | |
|---|---|
| **Why on EVM** | User doesn't have enough ETH to pay for gas |
| **StarkNet equivalent** | N/A — all Chipi transactions are gasless |
| **Fix** | Remove all gas balance checks, `estimateGas` calls, and gas price logic. Users never need gas tokens with Chipi. If the user has insufficient token balance for the operation itself, Chipi returns a clear error. |

### 7. "transaction was not mined within X blocks" / "replacement transaction underpriced"

| | |
|---|---|
| **Why on EVM** | Transaction stuck in mempool, gas price too low |
| **StarkNet equivalent** | N/A — Chipi handles submission and confirmation |
| **Fix** | Remove transaction replacement logic, gas price bumping, and mempool monitoring. Chipi hooks handle the full transaction lifecycle. |

### 8. "execution reverted" / CALL_EXCEPTION (ethers) / ContractFunctionExecutionError (viem)

| | |
|---|---|
| **Why on EVM** | Smart contract reverted the transaction (require/revert) |
| **StarkNet equivalent** | Cairo `assert` failures or `panic` — returned as error from Chipi SDK |
| **Fix** | Map EVM revert reasons to Cairo assert messages. Check calldata encoding (especially u256 [low, high] format). Verify contract address and entrypoint name are correct. |

## Token Errors

### 9. "ERC20: transfer amount exceeds allowance" / "ERC20InsufficientAllowance"

| | |
|---|---|
| **Why on EVM** | Forgot to call `approve()` before `transferFrom()` |
| **StarkNet equivalent** | Same pattern exists — but use Chipi's multicall to batch approve + action |
| **Fix** | Use `useCallAnyContract` with `calls[]` to batch `approve` + your action in a single transaction. No separate approval tx needed. See `evm-code-migration-patterns.md` for the multicall pattern. |

### 10. Token decimal mismatch (EVM 18 vs StarkNet 6, etc.)

| | |
|---|---|
| **Why on EVM** | Different tokens have different decimals (ETH=18, USDC=6) |
| **StarkNet equivalent** | Same decimal standards (ETH=18, USDC=6, STRK=18) |
| **Fix** | Use Chipi SDK's human-readable amount format. Pass `amount: 10` for $10 USDC. The SDK handles decimal conversion. Don't manually multiply by `10^n`. |

## Address & Format Errors

### 11. "invalid address" / "ENS name not configured" / "invalid ENS name"

| | |
|---|---|
| **Why on EVM** | Invalid address format or failed ENS resolution |
| **StarkNet equivalent** | ENS doesn't exist on StarkNet; StarkNet ID is separate |
| **Fix** | Remove all ENS resolution code (`resolveName`, `getEnsAddress`). Use raw StarkNet hex addresses. Validate with `/^0x[0-9a-fA-F]{64}$/`. If needed, integrate starknet.id separately. |

### 12. Address length confusion (40 chars vs 64 chars)

| | |
|---|---|
| **Why on EVM** | Developers paste EVM addresses (0x + 40 hex) into StarkNet calls |
| **StarkNet equivalent** | StarkNet uses 0x + 64 hex characters |
| **Fix** | Add validation: if address matches `/^0x[0-9a-fA-F]{40}$/`, show error "This looks like an Ethereum address. StarkNet addresses are 0x + 64 hex characters." Pad or reject as appropriate. |

### 13. EIP-55 checksum mismatch

| | |
|---|---|
| **Why on EVM** | Mixed-case checksum validation fails for addresses |
| **StarkNet equivalent** | StarkNet addresses are case-insensitive (no checksum) |
| **Fix** | Remove all checksum validation logic (`getAddress()` in ethers, `checksumAddress()` in viem). StarkNet hex addresses are case-insensitive — `0x033...` and `0x033...` are equivalent. |

## Provider & Config Errors

### 14. "Chain ID mismatch" / "wallet is connected to the wrong network"

| | |
|---|---|
| **Why on EVM** | User's wallet is on Ethereum but app expects Polygon (or vice versa) |
| **StarkNet equivalent** | N/A — Chipi only targets StarkNet, no chain switching |
| **Fix** | Remove all chain switching logic (`wallet_switchEthereumChain`, `useSwitchChain`, chain ID checks). There's only one network: StarkNet. No chain switching UI needed. |

### 15. Provider kit initialization errors (RainbowKit/ConnectKit/Web3Modal)

| | |
|---|---|
| **Why on EVM** | Missing WalletConnect projectId, wrong chain config, or version mismatch |
| **StarkNet equivalent** | N/A — Chipi uses ChipiProvider, no external wallet kit |
| **Fix** | Remove the entire provider stack (RainbowKitProvider, ConnectKitProvider, Web3Modal, WagmiProvider, QueryClientProvider). Replace with ChipiProvider. See `evm-code-migration-patterns.md` for full removal patterns. |

## Encoding Errors

### 16. u256 single value vs [low, high] encoding

| | |
|---|---|
| **Why on EVM** | EVM uint256 is a single 256-bit value |
| **StarkNet equivalent** | StarkNet u256 = two felt252 values: [low_128_bits, high_128_bits] |
| **Fix** | For amounts under 2^128 (almost all practical values), pass `[amount, "0"]`. Example: 1000000 USDC raw = `["1000000", "0"]`. For large values, split: `low = value & ((1n << 128n) - 1n)`, `high = value >> 128n`. |

### 17. ABI encoding errors / "could not decode result data"

| | |
|---|---|
| **Why on EVM** | Wrong ABI, mismatched function signature, or incorrect argument types |
| **StarkNet equivalent** | Wrong calldata encoding or mismatched entrypoint |
| **Fix** | Chipi's `useCallAnyContract` doesn't use ABI files. Pass entrypoint as a string and calldata as a string array. Verify entrypoint name matches the Cairo function name exactly (snake_case). Verify calldata types (felt252 strings, u256 as [low, high]). |

### 18. BigNumber / bigint conversion errors

| | |
|---|---|
| **Why on EVM** | Mixing ethers.js v5 BigNumber with v6 bigint, or wrong formatting |
| **StarkNet equivalent** | N/A — Chipi uses human-readable amounts |
| **Fix** | Remove all BigNumber imports (`ethers.BigNumber`, `@ethersproject/bignumber`). Remove `parseUnits`/`formatUnits` calls. Chipi SDK accepts human-readable numbers: `amount: 10` for $10 USDC. |

## Quick Diagnosis Checklist

If something isn't working after migration, check these in order:

1. **All EVM imports removed?** — Search for `ethers`, `viem`, `wagmi`, `@rainbow-me`, `connectkit`, `@web3modal`
2. **Provider stack removed?** — No WagmiProvider, RainbowKitProvider, ConnectKitProvider, QueryClientProvider (if only for wagmi)
3. **Using 64-char hex addresses (not 40)?** — StarkNet addresses are `0x` + 64 hex chars
4. **No `window.ethereum` checks?** — Remove all injected provider detection
5. **No gas estimation logic?** — Remove estimateGas, gasLimit, gasPrice, maxFeePerGas
6. **No nonce management?** — Remove getTransactionCount, manual nonce tracking
7. **No chain switching?** — Remove switchChain, chainId checks, multi-chain config
8. **Using human-readable amounts?** — `amount: 10` not `amount: 10000000n`
9. **ChipiProvider wrapping the app?** — Check layout.tsx
10. **Environment variables set?** — `NEXT_PUBLIC_CHIPI_API_KEY` exists
