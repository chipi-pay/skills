---
name: chipi-starknet-guide
description: StarkNet blockchain reference for Chipi development. Covers address formats, supported tokens (USDC/ETH/STRK), transaction lifecycle, error codes, rate limits, and security best practices. Use when user asks about "Starknet address", "USDC decimals", "transaction status", "Chipi errors", or "Chipi troubleshooting".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi StarkNet Guide

Complete StarkNet reference for Chipi development.

## Wallet Types

| Type | Contract | Session Keys | Default |
|------|----------|-------------|---------|
| **CHIPI** | Chipi's own account contract | **Yes** | **Yes** |
| READY | Argent X compatible | No | No |

Always default to CHIPI. Only use READY if user explicitly needs Argent X compatibility.

## Address Format

- Format: `0x` + 64 hexadecimal characters (256-bit)
- Example: `0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7`
- Validation: `/^0x[0-9a-fA-F]{64}$/`

## Supported Tokens

| Token | Decimals | "1.0" raw value |
|-------|----------|-----------------|
| USDC | 6 | 1000000 |
| ETH | 18 | 1000000000000000000 |
| STRK | 18 | 1000000000000000000 |

Custom ERC-20 via `otherToken`: `{ name, address, symbol, decimals }`

## Transaction Lifecycle

```
PENDING -> PROCESSING -> COMPLETED
                      -> FAILED
```

Average confirmation: ~15-30 seconds on StarkNet.

## Authentication

| Method | Security | Recommended |
|--------|----------|-------------|
| **Passkey** (biometric) | Strong — device-bound | **Yes** |
| PIN (4-digit) | Weak — limited entropy | No — warn user |

Migration: Use the `migrate-to-passkey-dialog` component to convert PIN to passkey. After migration, PIN stops working.

## API

- **Base URL:** `https://api.chipipay.com/v1`
- **Rate Limit:** 100 requests/minute
- **Auth:** `Authorization: Bearer pk_prod_YOUR_KEY`
- `pk_prod_*` = public (frontend), `sk_prod_*` = secret (backend only)

## SDK Packages

| Package | Platform | Install |
|---------|----------|---------|
| `@chipi-stack/nextjs` | Next.js | `npm install @chipi-stack/nextjs` |
| `@chipi-stack/chipi-react` | React | `npm install @chipi-stack/chipi-react` |
| `@chipi-stack/chipi-expo` | Expo | `npx expo install @chipi-stack/chipi-expo` |

All packages export identical hooks.

## Auth Providers

Clerk (recommended), Firebase, Supabase, Better Auth

## Common Errors

| Error | Solution |
|-------|----------|
| "User not authenticated" | Configure auth provider, ensure user is signed in |
| "Wallet not found" | Create wallet with `useCreateWallet` |
| "Insufficient balance" | Check with `useGetTokenBalance` |
| "Invalid address" | Must be 0x + 64 hex chars |
| "Session key not valid" | Session keys require CHIPI wallet |
| "Rate limit exceeded" | Implement request throttling |
| "Invalid PIN" | Wrong encryption key |
| "SKU not available" | Mexico only |

## 20 SDK Hooks

**Wallet:** useCreateWallet, useChipiWallet, useGetWallet

**Tokens:** useTransfer, useApprove, useGetTokenBalance

**Session Keys (CHIPI only):** useChipiSession, useCreateSessionKey, useAddSessionKeyToContract, useExecuteWithSession, useGetSessionData, useRevokeSessionKey

**DeFi:** useStakeVesuUsdc, useWithdrawVesuUsdc

**SKU Marketplace:** useGetSkuList, useGetSku, usePurchaseSku, useGetSkuPurchase

**Utilities:** useCallAnyContract, useGetTransactionList
