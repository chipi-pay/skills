# Chipi Error Database

40+ categorized errors with diagnostic steps and fixes.

## Wallet Errors

| # | Error Message | Category | Cause | Diagnostic Steps | Fix |
|---|---|---|---|---|---|
| 1 | "Wallet not found" | Wallet | No wallet for this externalUserId | Check `hasWallet` from `useChipiWallet` | Create wallet with `useCreateWallet` |
| 2 | "User not authenticated" | Wallet | Auth session missing/expired | Check auth provider (Clerk/Firebase/Supabase) | Sign in the user first |
| 3 | "Wallet already exists" | Wallet | Second wallet creation attempt | Check `hasWallet` before calling create | Use existing wallet via `useChipiWallet` |
| 4 | "NotAllowedError" | Wallet | User cancelled passkey prompt | User interaction required | Show friendly retry message |
| 5 | "InvalidStateError" | Wallet | WebAuthn ceremony in progress | Multiple passkey prompts fired | Wait for current ceremony, debounce calls |
| 6 | "Invalid PIN" | Wallet | Wrong 4-digit encryption key | User entered wrong PIN | Prompt to re-enter. Consider passkey migration. |
| 7 | "PRF not supported" | Wallet | Device lacks WebAuthn PRF | Call `isPRFSupported()` | Fall back to PIN auth |
| 8 | "SecurityError: origin mismatch" | Wallet | Passkey created on different domain | Check domain in passkey config | Ensure same origin for create and authenticate |
| 9 | "AbortError" | Wallet | Passkey operation timed out | User didn't respond to biometric prompt | Retry with user instruction |
| 10 | "externalUserId is required" | Wallet | Missing user ID parameter | Check auth provider user ID | Pass `externalUserId` from auth session |

## Transaction Errors

| # | Error Message | Category | Cause | Diagnostic Steps | Fix |
|---|---|---|---|---|---|
| 11 | "Insufficient balance" | Transaction | Not enough tokens | Check `useGetTokenBalance` | Fund wallet or reduce amount |
| 12 | "Invalid address" | Transaction | Malformed recipient address | Validate: `/^0x[0-9a-fA-F]{64}$/` | Fix address format |
| 13 | "Invalid amount" | Transaction | Amount ≤ 0 or NaN | Check amount value and type | Pass positive number |
| 14 | "EXECUTION_FAILED" | Transaction | On-chain revert | Check tx on Starkscan | Fix calldata or contract params |
| 15 | "Transaction timeout" | Transaction | Network congestion | Check StarkNet status | Retry after delay |
| 16 | "REJECTED" | Transaction | Transaction rejected by sequencer | Check for duplicate nonce | Retry — SDK handles nonce |
| 17 | "Rate limit exceeded" | Transaction | >100 requests/minute | Check request frequency | Implement throttling/debounce |
| 18 | "Invalid bearer token" | Transaction | Wrong or expired API key | Check NEXT_PUBLIC_CHIPI_API_KEY | Verify key starts with `pk_prod_` |
| 19 | "Network error" | Transaction | API unreachable | Check internet, check API status | Retry with exponential backoff |
| 20 | "Invalid token address" | Transaction | Token not recognized | Check supported tokens list | Use "USDC", "ETH", "STRK", or valid hex address |

## Session Key Errors

| # | Error Message | Category | Cause | Diagnostic Steps | Fix |
|---|---|---|---|---|---|
| 21 | "Session keys require CHIPI wallet" | Session | Wrong wallet type | Check `wallet.walletType` | Create CHIPI wallet (not READY) |
| 22 | "Session expired" | Session | Past expiry timestamp | Check `hasActiveSession` | Create + register new session |
| 23 | "No remaining calls" | Session | Call limit exhausted | Check `remainingCalls` | Create + register new session |
| 24 | "Session not registered" | Session | Registration tx not confirmed | Check registration tx status | Wait or re-register |
| 25 | "Method not allowed" | Session | Entrypoint not in allowedMethods | Compare call to session scope | Create session with correct allowedMethods |
| 26 | "Session key revoked" | Session | Session was explicitly revoked | Check session state | Create + register new session |
| 27 | "Wallet not deployed" | Session | Wallet has never transacted | First transaction deploys wallet | Make any transaction first, then register session |

## Contract Call Errors

| # | Error Message | Category | Cause | Diagnostic Steps | Fix |
|---|---|---|---|---|---|
| 28 | "Contract not found" | Contract | Invalid/undeployed address | Check address on Starkscan | Verify contract address |
| 29 | "Entrypoint not found" | Contract | Wrong function name | Check ABI on Starkscan | Use exact name (snake_case) |
| 30 | "Invalid calldata" | Contract | Wrong arg count or types | Compare to function signature | Fix calldata array |
| 31 | "u256 overflow" | Contract | Value exceeds u256 max | Check value bounds | Reduce value or check encoding |
| 32 | "Approval needed" | Contract | Token spending without approve | Check if function spends tokens | Batch approve + call in `calls[]` |
| 33 | "Revert: not authorized" | Contract | Caller lacks permission | Check contract access control | Use correct wallet/role |
| 34 | "Revert: paused" | Contract | Contract is paused | Check contract state | Wait for unpause or contact owner |

## Address Errors

| # | Error Message | Category | Cause | Diagnostic Steps | Fix |
|---|---|---|---|---|---|
| 35 | "Invalid StarkNet address" | Address | Not 0x + 64 hex | Count chars, check format | Pad/fix to 66 chars total |
| 36 | "EVM address detected" | Address | 0x + 40 hex used | Address is Ethereum format | Use StarkNet address (64 hex) |
| 37 | "Base58 address detected" | Address | Solana address used | No 0x prefix, Base58 chars | Convert to StarkNet hex address |
| 38 | "Address too short" | Address | Leading zeros stripped | Count hex chars after 0x | Pad with leading zeros |

## Webhook Errors

| # | Error Message | Category | Cause | Diagnostic Steps | Fix |
|---|---|---|---|---|---|
| 39 | "Invalid signature" (HMAC) | Webhook | Wrong key or body format | Check key type (sk_prod_), body parsing | Use `request.text()` for HMAC, `sk_prod_` key |
| 40 | "No webhook received" | Webhook | Wrong URL or firewall | Check dashboard URL, test with ngrok | Verify public accessibility |
| 41 | "Body parsing error" | Webhook | Double body consumption | Reading body twice | Read as text, verify, then JSON.parse |
| 42 | "Unauthorized (401)" | Webhook | Signature check failing | Compare expected vs received signature | Fix HMAC computation |

## Environment & Config Errors

| # | Error Message | Category | Cause | Diagnostic Steps | Fix |
|---|---|---|---|---|---|
| 43 | "ChipiProvider not found" | Config | Provider missing from tree | Check layout.tsx | Wrap app in ChipiProvider |
| 44 | "API key not set" | Config | Missing env var | Check .env.local | Add NEXT_PUBLIC_CHIPI_API_KEY |
| 45 | "Invalid API key format" | Config | Key doesn't match expected pattern | Check key prefix | Must start with `pk_prod_` (public) or `sk_prod_` (secret) |
| 46 | "Module not found: @chipi-stack/*" | Config | SDK not installed | Check package.json | Run `npm install @chipi-stack/nextjs` |
