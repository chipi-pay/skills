---
name: chipi-debug
description: Systematic debugging guide for Chipi integrations. Diagnose wallet errors, failed transactions, session key issues, address format mistakes, calldata encoding problems, and webhook failures. Use when user says "debug", "error", "not working", "transaction failed", "troubleshoot", or "why is my transaction failing".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Debug Chipi Integrations

Systematic debugging guide for every type of error in the Chipi ecosystem. Use this when something isn't working — wallet creation fails, transactions revert, session keys expire, addresses are rejected, or webhooks don't fire.

## Used in

All Chipi integrations — wallet setup, payments, session keys, staking, SKU marketplace, custom contracts, and migrations from EVM/Solana.

## When in Doubt, Ask

Before debugging, ask the user for:
- **Full error message** — exact text, not paraphrased
- **Platform** — Next.js, React, Expo, Python, backend?
- **SDK version** — which `@chipi-stack/*` package and version?
- **What they were trying to do** — which hook, what parameters?
- **Browser/device** — Chrome, Safari, mobile? (matters for passkey/PRF issues)

## Step 1: Identify Error Category

Use this decision tree to route to the right section:

```
Error received
├─ Wallet-related? → Step 2
│  (create wallet, wallet not found, auth method, passkey, PIN)
├─ Transaction-related? → Step 3
│  (transfer failed, insufficient balance, timeout)
├─ Session key-related? → Step 4
│  (session expired, wrong wallet type, registration failed)
├─ Contract call-related? → Step 5
│  (contract not found, entrypoint error, calldata invalid)
├─ Address-related? → Step 6
│  (invalid address, wrong format, wrong chain)
├─ Calldata encoding? → Step 7
│  (u256 encoding, argument order, type mismatch)
├─ Webhook-related? → Step 8
│  (HMAC mismatch, no webhook received, parsing error)
└─ Migration-specific? → Step 9
   (EVM/Solana patterns that don't apply on StarkNet)
```

## Step 2: Wallet Debugging

| Error | Cause | Diagnostic Steps | Fix |
|-------|-------|-----------------|-----|
| "Wallet not found" | No wallet exists for this `externalUserId` | Check `hasWallet` from `useChipiWallet`. Verify `externalUserId` matches auth provider's user ID. | Create wallet first with `useCreateWallet`. |
| "User not authenticated" | Auth session expired or missing | Check auth provider (Clerk/Firebase/Supabase) status. Is user signed in? | Ensure auth provider wraps the app. Re-authenticate the user. |
| "NotAllowedError" (passkey) | User cancelled the passkey prompt or device doesn't support WebAuthn | Check if `isPRFSupported()` returns true. Try on Chrome (best support). | If PRF not supported, fall back to PIN. If user cancelled, prompt again. |
| "Invalid PIN" | Wrong encryption key provided | User entered the wrong 4-digit PIN | Prompt user to re-enter PIN. No retry limit in SDK (implement your own). |
| PRF returns null | Device supports WebAuthn but not the PRF extension | Call `isPRFSupported()` to check. Safari on iOS <17 lacks PRF. | Fall back to PIN auth. Consider showing "Upgrade to passkey" later. |
| "Wallet already exists" | Attempting to create a second wallet for the same user | Check `hasWallet` before calling `useCreateWallet` | Skip wallet creation. Use `useChipiWallet` to access existing wallet. |

## Step 3: Transaction Debugging

| Error | Cause | Diagnostic Steps | Fix |
|-------|-------|-----------------|-----|
| "Insufficient balance" | Not enough tokens for the transfer | Call `useGetTokenBalance` and compare to transfer amount. Check correct token. | Fund the wallet. Ensure amount doesn't exceed balance. |
| "Invalid address" | Malformed StarkNet address in `recipientAddress` | Validate with `/^0x[0-9a-fA-F]{64}$/`. Check for trailing spaces. | Fix the address format. See Step 6. |
| "Invalid amount" | Amount is 0, negative, or malformed | Check that amount > 0 and is a valid number | Pass a positive number. SDK handles decimals. |
| Transaction timeout | Transaction submitted but not confirmed in time | Check StarkNet network status. It may be temporary congestion. | Retry the transaction. If persistent, check if the wallet has been deployed. |
| "EXECUTION_FAILED" | On-chain execution reverted | Check the transaction on Starkscan/Voyager for revert reason. | Fix the calldata or check contract requirements. |

## Step 4: Session Key Debugging

| Error | Cause | Diagnostic Steps | Fix |
|-------|-------|-----------------|-----|
| "Session keys require CHIPI wallet type" | Wallet is type "READY" (Argent X) | Check `wallet.walletType` | Session keys only work with CHIPI wallets. Create a new CHIPI wallet. |
| Session expired | Session key exceeded its expiry time | Check `hasActiveSession` and session expiry timestamp | Create a new session with `createSession` + `registerSession` |
| "No remaining calls" | Session key exhausted its call allowance | Check `remainingCalls` from `useChipiSession` | Create and register a new session key |
| Registration failed | On-chain session key registration transaction reverted | Check transaction status. Wallet may not be deployed yet. | Ensure wallet is deployed (has made at least one transaction). Retry registration. |
| "Method not allowed" | Session key doesn't have permission for this contract/entrypoint | Check `allowedMethods` used during session creation | Create a new session key with the correct `allowedMethods` scope |

## Step 5: Contract Call Debugging

| Error | Cause | Diagnostic Steps | Fix |
|-------|-------|-----------------|-----|
| "Contract not found" | Invalid contract address or contract not deployed | Check address on Starkscan. Verify hex format. | Use the correct deployed contract address (0x + 64 hex chars). |
| "Entrypoint not found" | Function name doesn't exist on the contract | Check contract ABI on Starkscan/Voyager. Case-sensitive. | Use exact entrypoint name from ABI (snake_case in Cairo). |
| "Invalid calldata" | Wrong number of arguments or wrong types | Compare calldata array length to function signature. Check u256 encoding. | See Step 7 for calldata encoding rules. |
| u256 encoding error | Passing single value instead of [low, high] pair | u256 requires two felt252 values: `[low_128_bits, high_128_bits]` | Split: `[amountLow, "0"]` for values under 2^128. See Step 7. |
| Approval needed | Contract tries to spend tokens without approval | Check if entrypoint requires prior ERC-20 approval | Batch approve + call in one transaction using `calls[]` array |

## Step 6: Address Validation

### StarkNet Address Format
```
Valid:   0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7
Regex:   /^0x[0-9a-fA-F]{64}$/
Length:  66 characters total (0x prefix + 64 hex chars)
```

### Common Format Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| EVM address (too short) | `0x742d35Cc6634C0532925a3b844Bc9e7595f2bD` (40 chars) | StarkNet needs 64 hex chars. This is an Ethereum address. |
| Solana address (Base58) | `7EcDhSYGxXyscszYEp35KHN8sAD...` | Convert to StarkNet hex format. Solana addresses can't be used on StarkNet. |
| Missing 0x prefix | `049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7` | Add `0x` prefix. |
| Leading zeros stripped | `0x49d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7` (63 chars) | Pad with leading zeros to reach 64 hex chars after `0x`. |
| Trailing whitespace | `0x049d...dc7 ` | Trim the address string. |
| Checksum mixed case | `0x049D36570D4E46F48E99674BD3FCC84644DDD6B96F7C741B1562B82F9E004DC7` | StarkNet addresses are case-insensitive. This is valid but normalize to lowercase. |

### Validation Code

```tsx
function isValidStarkNetAddress(address: string): boolean {
  return /^0x[0-9a-fA-F]{64}$/.test(address);
}

function detectAddressType(address: string): string {
  if (/^0x[0-9a-fA-F]{40}$/.test(address)) return "EVM (Ethereum/Base/Polygon)";
  if (/^0x[0-9a-fA-F]{64}$/.test(address)) return "StarkNet";
  if (/^[1-9A-HJ-NP-Za-km-z]{32,44}$/.test(address)) return "Solana (Base58)";
  return "Unknown format";
}
```

## Step 7: Calldata Debugging

### u256 Encoding

StarkNet's u256 is represented as two felt252 values: `[low_128_bits, high_128_bits]`.

```tsx
// For values under 2^128 (most practical values):
const calldata = [amount.toString(), "0"];  // [low, high=0]

// Example: 10 USDC (6 decimals) = 10_000_000
const calldata = ["10000000", "0"];

// Example: 1 ETH (18 decimals) = 1_000_000_000_000_000_000
const calldata = ["1000000000000000000", "0"];
```

### Common Encoding Mistakes

| Mistake | What Happens | Fix |
|---------|-------------|-----|
| Passing u256 as single value | "Invalid calldata" error | Split into `[low, high]` pair |
| Wrong argument order | Unexpected behavior or revert | Match exact order from contract ABI |
| Using number instead of string | Precision loss for large values | Always pass calldata as strings |
| Forgetting to encode address | Revert or wrong recipient | Pass address as hex string with 0x prefix |
| Human-readable amount in calldata | Wrong amount (way too small) | For `useCallAnyContract`, pass raw amounts. For `useTransfer`, pass human-readable. |

### Block Explorer Verification

After a failed transaction, check the details on:
- **Starkscan:** `https://starkscan.co/tx/{txHash}`
- **Voyager:** `https://voyager.online/tx/{txHash}`

Look for:
- Revert reason in the execution trace
- Actual calldata sent vs expected
- Contract state at time of execution

### Type Mapping: Solidity → Cairo → Calldata

| Solidity Type | Cairo Type | Calldata Format |
|---------------|-----------|-----------------|
| `uint256` | `u256` | `[low_string, high_string]` |
| `address` | `ContractAddress` | `"0x..."` (64 hex chars) |
| `uint128` | `u128` | `"value_string"` |
| `uint64` | `u64` | `"value_string"` |
| `bool` | `bool` | `"0"` or `"1"` |
| `string` | `felt252` / `ByteArray` | Hex-encoded short string or array |
| `bytes` | `Array<felt252>` | `[length, ...elements]` |

## Step 8: Webhook Debugging

| Error | Cause | Diagnostic Steps | Fix |
|-------|-------|-----------------|-----|
| HMAC signature mismatch | Wrong key or body parsing | Verify using `sk_prod_` key (not `pk_prod_`). Ensure raw body text is used for HMAC, not parsed JSON. | Use `request.text()` not `request.json()` for signature verification. |
| No webhook received | Wrong endpoint URL or firewall | Check webhook URL in Chipi dashboard. Test with a tool like ngrok for local dev. | Verify endpoint is publicly accessible. Check for CORS/firewall issues. |
| Body parsing error | Parsing body twice (once for HMAC, once for data) | Read body as text first for HMAC, then parse as JSON | `const body = await request.text(); verify(body); const data = JSON.parse(body);` |
| Wrong key type | Using public key (`pk_prod_`) instead of secret key (`sk_prod_`) | Check the key prefix in your .env | Use `CHIPI_SECRET_KEY` (sk_prod_*) for webhook verification, never the public key |
| 401/403 response | Signature verification rejecting valid webhooks | Log both expected and received signatures for comparison | Ensure no middleware modifies the request body before verification |

### Webhook HMAC Verification Pattern

```tsx
import crypto from "crypto";

export async function POST(request: Request) {
  const body = await request.text(); // IMPORTANT: text(), not json()
  const signature = request.headers.get("chipi-signature");

  const expected = crypto
    .createHmac("sha256", process.env.CHIPI_SECRET_KEY!) // sk_prod_*
    .update(body)
    .digest("hex");

  // Use constant-time comparison to prevent timing attacks
  const sigBuf = Buffer.from(signature ?? "", "utf8");
  const expBuf = Buffer.from(expected, "utf8");
  if (sigBuf.length !== expBuf.length || !crypto.timingSafeEqual(sigBuf, expBuf)) {
    return Response.json({ error: "Invalid signature" }, { status: 401 });
  }

  const payload = JSON.parse(body);
  // Process payment...
}
```

## Step 9: Migration-Specific Errors

### EVM Patterns That Don't Apply

| Pattern | Why It Fails | StarkNet Fix |
|---------|-------------|-------------|
| `window.ethereum` | No MetaMask on StarkNet | Use `useChipiWallet` — no browser extension needed |
| `ethers.parseUnits(amount, 6)` | ethers.js not needed | Pass human-readable amount to `useTransfer` |
| `provider.getNetwork()` | No network switching | Always StarkNet — single chain |
| `signer.sendTransaction()` | No signer pattern | Use Chipi hooks — `useTransfer`, `useCallAnyContract` |
| `contract.on("event")` | No real-time event listening | Use `useGetTransactionList` to poll for updates |

### Solana Patterns That Don't Apply

| Pattern | Why It Fails | StarkNet Fix |
|---------|-------------|-------------|
| `findProgramAddressSync()` | No PDAs on StarkNet | Use contract storage directly |
| `getAssociatedTokenAddress()` | No ATAs on StarkNet | Use ERC-20 `balanceOf` via `useGetTokenBalance` |
| `getLatestBlockhash()` | No blockhash in StarkNet txs | Chipi handles transaction lifecycle |
| `ComputeBudgetProgram` | No compute budget on StarkNet | Delete — Chipi handles gas |
| `wallet.signMessage()` | No wallet-based auth | Use standard auth (Clerk/Firebase/Supabase) |

## Diagnostic Checklist

Run through this checklist before debugging specific errors:

1. **SDK installed?** — `@chipi-stack/nextjs` (or react/expo) in package.json
2. **Provider wrapping app?** — ChipiProvider in layout.tsx/App.tsx
3. **Auth provider configured?** — Clerk/Firebase/Supabase wrapping the app
4. **Environment variables set?** — `NEXT_PUBLIC_CHIPI_API_KEY` exists and starts with `pk_prod_`
5. **User authenticated?** — Auth provider shows signed-in state
6. **Wallet exists?** — `hasWallet` from `useChipiWallet` returns true
7. **Correct wallet type?** — CHIPI for session keys, READY for Argent X
8. **Addresses valid?** — All addresses match `/^0x[0-9a-fA-F]{64}$/`
9. **Amounts valid?** — Positive numbers, correct decimal format for the hook
10. **Network correct?** — StarkNet mainnet for production, testnet for development

## Hooks Used

All Chipi hooks can produce errors. The most common error-producing hooks:
- `useCreateWallet` — auth, passkey, PRF issues
- `useTransfer` — balance, address, amount issues
- `useCallAnyContract` — calldata, contract, entrypoint issues
- `useChipiSession` / session hooks — wallet type, expiry, scope issues
- `usePurchaseSku` — reference validation, balance issues

## UI Guidance

> **Load `chipi-frontend-design` before generating any error UI.**

Error display patterns:
- Show the full error message in a dismissible toast or alert
- For wallet errors, show a "Create Wallet" or "Sign In" CTA
- For transaction errors, show a "Retry" button
- For address errors, show the expected format with an example
- Never show raw error objects to users — format them into human-readable messages

## Reference Files

For detailed error databases and debugging guides, see:
- `references/error-database.md` — 40+ categorized errors with fixes
- `references/address-validation.md` — Format specs, regex patterns, validation code
- `references/calldata-debugging.md` — u256 encoding, type mapping, common mistakes

## What's Next

Based on the error type, the user may need:
- `chipi-wallet-setup` — for wallet creation issues
- `chipi-session-keys` — for session key issues
- `chipi-custom-contracts` — for contract call issues
- `chipi-migrate-from-evm` — for EVM migration errors
- `chipi-migrate-from-solana` — for Solana migration errors
