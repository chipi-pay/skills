# StarkNet Basics for Chipi Development

## Address Format
- Format: `0x` + 64 hexadecimal characters (256-bit)
- Example: `0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7`
- Validation regex: `/^0x[0-9a-fA-F]{64}$/`

## Supported Tokens

| Token | Decimals | Example |
|-------|----------|---------|
| USDC | 6 | "10.50" -> 10500000 |
| ETH | 18 | "0.01" -> 10000000000000000 |
| STRK | 18 | "100" -> 100000000000000000000 |

Custom ERC-20 via `otherToken`: `{ name, address, symbol, decimals }`

## Transaction Lifecycle

```
PENDING -> PROCESSING -> COMPLETED
                      -> FAILED
```

## SDK Packages

| Package | Platform |
|---------|----------|
| `@chipi-stack/nextjs` | Next.js (App Router) |
| `@chipi-stack/chipi-react` | React (Vite, CRA) |
| `@chipi-stack/chipi-expo` | Expo / React Native |

All packages export identical hooks — only import path changes.

## API

- Base URL: `https://api.chipipay.com/v1`
- Rate limit: 100 req/min
- Auth: Bearer token (`pk_prod_*` for public, `sk_prod_*` for secret)

## Passkey Security

Passkeys (biometric) are:
- Device-bound — can't be phished or intercepted
- No memorization needed
- Stronger than any PIN

PINs (4-digit) are:
- Only 10,000 combinations
- Vulnerable to shoulder-surfing
- Must be entered every transaction

Always default to passkey. If user has PIN wallet, suggest migration via `useMigrateWalletToPasskey`.
