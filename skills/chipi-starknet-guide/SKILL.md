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

## In Plain English
StarkNet is a network that runs on top of Ethereum but is faster and cheaper. Think of it like an express lane on a highway — transactions get bundled together and verified with math (ZK proofs) instead of being checked one by one. Chipi handles all the complicated parts — your users just see a simple app with no gas fees, no seed phrases, and instant wallets via fingerprint.

## Wallet Types

| Type | Contract | Session Keys | Default |
|------|----------|-------------|---------|
| **CHIPI** | Chipi's own account contract | **Yes** | **Yes** |
| READY | Argent X compatible | No | No |

Always default to CHIPI. Only use READY if user explicitly needs Argent X compatibility.

> **Why CHIPI is the default:** The CHIPI wallet type is Chipi's own account contract. It supports session keys (auth once, transact many times), has the best security, and is the most feature-rich. READY wallets are Argent X compatible but lack session key support.

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

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key StarkNet UI rules:
- **Addresses**: `font-mono text-sm` truncated to `0x1234...abcd` (6+4 chars). Copy button with `Copy`→`Check` icon swap + `toast.success("Copied")`
- **Token amounts**: `font-mono tabular-nums` — USDC uses 2 decimals (`$10.00`), ETH/STRK uses 4 decimals (`0.0042 ETH`). Always show token symbol suffix
- **Transaction status**: use `StepProgress` with steps `["Submitted", "Processing", "Confirmed"]`. Submitted = `--accent`, Confirmed = `--success`
- **Block explorer links**: use `ExternalLink` icon (`h-4 w-4`) next to transaction hashes. `target="_blank" rel="noopener noreferrer"`
- **Error codes**: map hex error codes to human-readable messages. Show "Contract reverted: insufficient balance" not `0x4e6f742...`. Use `text-destructive` color
- **Chain indicator**: small badge showing "StarkNet" with chain icon near addresses and transactions
- **Gas display**: show "Gasless" badge in `text-success` when sponsored, or gas estimate in `font-mono tabular-nums` when not
- **Responsive**: addresses use `text-xs` on mobile to prevent overflow, `text-sm` at `md:`

## When in Doubt, Ask
If the user asks about a StarkNet concept not covered here, be honest about what you don't know. It's better to say "I'm not sure about that specific detail — let me check" than to guess incorrectly about blockchain mechanics.
