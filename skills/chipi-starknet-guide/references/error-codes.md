# Chipi Error Codes & Solutions

## Authentication Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "User not authenticated" | Auth provider not configured | Set up Clerk/Firebase/Supabase |
| "Invalid bearer token" | Wrong or expired API key | Use pk_prod_ for frontend, sk_prod_ for backend |
| "Unauthorized" | Missing Authorization header | Add Bearer token header |

## Wallet Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Wallet not found" | No wallet for this user ID | Call useCreateWallet |
| "Wallet already exists" | Duplicate creation | Use useGetWallet to fetch existing |
| "Invalid wallet type" | Unknown walletType | Use "CHIPI" (default) or "READY" |
| "Invalid PIN" | Wrong encryption key | Re-enter correct PIN |

## Transaction Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Insufficient balance" | Not enough tokens | Check with useGetTokenBalance |
| "Invalid address" | Malformed address | Must be 0x + 64 hex characters |
| "Transaction failed" | On-chain revert | Check calldata, balance, contract |
| "Invalid amount" | Amount <= 0 | Use positive numbers as strings |

## Session Key Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Session key not valid" | Expired or wrong wallet type | Requires CHIPI wallet |
| "No active session" | Not created/registered | Run createSession + registerSession |
| "Session expired" | Past 6-hour expiry | Create new session |
| "No remaining calls" | Limit reached | Re-register session |

## SKU Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "SKU not found" | Invalid SKU ID | Check with useGetSkuList |
| "Invalid reference" | Doesn't match pattern | Validate against referencePattern |
| "SKU not available" | Wrong market | Mexico only |
