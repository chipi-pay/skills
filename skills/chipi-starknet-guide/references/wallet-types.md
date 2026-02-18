# Chipi Wallet Types

## CHIPI (Default)

Chipi's own account contract on StarkNet.

- Session key support (one auth, many transactions)
- Gasless transactions via Chipi
- Passkey or PIN authentication
- **Default — use this unless user needs Argent X**

## READY (Argent X)

Argent X compatible account contract.

- Compatible with Argent X wallet app
- Gasless transactions via Chipi
- Passkey or PIN authentication
- **NO session key support**

## Comparison

| Feature | CHIPI | READY |
|---------|-------|-------|
| Default | Yes | No |
| Session Keys | Yes | No |
| Gasless | Yes | Yes |
| Passkey Auth | Yes | Yes |
| PIN Auth | Yes | Yes |
| Argent X Compatible | No | Yes |

## Session Key Implications

If a user creates a READY wallet and later wants session keys:
- Session keys will NOT work with READY wallets
- User must create a NEW CHIPI wallet
- Cannot convert READY to CHIPI (different account contracts)

Always recommend CHIPI by default to keep session keys as an option.

## Code

```typescript
// CHIPI (default)
await createWallet({ walletType: "CHIPI", usePasskey: true, ... });

// READY (only for Argent X)
await createWallet({ walletType: "READY", usePasskey: true, ... });
```
