---
name: chipi-session-keys
description: Implement gasless session keys for frictionless Chipi wallet transactions without requiring passkey/PIN for every transaction. REQUIRES CHIPI wallet type (not READY/Argent). Use when user says "session keys", "gasless transactions", "frictionless UX", "delegated transactions", or "no auth every time".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi Session Keys

Enable frictionless transactions — authenticate once, execute many transactions without re-entering passkey/PIN.

## Critical Prerequisite

**Session keys ONLY work with `walletType: "CHIPI"`.** They do NOT work with READY (Argent X) wallets.

```tsx
if (wallet.walletType !== "CHIPI") {
  throw new Error("Session keys require CHIPI wallet type.");
}
```

## Step 1: Verify Wallet Setup

Ensure the user has a CHIPI wallet. If not, run `chipi-wallet-setup` first with `walletType: "CHIPI"`.

**VERIFY:** Wallet exists with `walletType: "CHIPI"`.

## Step 2: Use useChipiSession (Recommended)

```tsx
import { useChipiSession } from "@chipi-stack/nextjs"; // or chipi-react/chipi-expo

const {
  session, sessionState, hasActiveSession, remainingCalls,
  createSession, registerSession, executeWithSession, revokeSession,
} = useChipiSession();
```

## Step 3: Session Lifecycle

### 3a. Create Session (Local)
```tsx
const session = await createSession();
// Generates keypair locally — no on-chain tx yet
// sessionState -> "created"
```

### 3b. Register Session (On-Chain)
```tsx
await registerSession({
  encryptKey: passkeyCredential, // Owner must authenticate this one time
  wallet: userWallet,
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});
// sessionState -> "active"
```

### 3c. Execute with Session (Frictionless)
```tsx
await executeWithSession({
  calls: [{
    contractAddress: "0x...",
    entrypoint: "transfer",
    calldata: [recipientAddress, amountLow, amountHigh],
  }],
});
// No auth prompt! Session key signs the tx.
// remainingCalls decrements
```

### 3d. Monitor Session
```tsx
console.log(sessionState);     // "active"
console.log(hasActiveSession); // true
console.log(remainingCalls);   // 42
```

### 3e. Revoke on Logout
```tsx
await revokeSession();
// sessionState -> "revoked"
```

## Step 4: Integration with Auth Logout

```tsx
const handleLogout = async () => {
  await revokeSession(); // Revoke session key first
  await signOut();        // Then sign out
};
```

## Step 5: Test

1. Create session — verify sessionState changes
2. Register session — verify on-chain registration
3. Execute with session — verify NO auth prompt
4. Check remainingCalls — verify decrement
5. Revoke session — verify it's disabled

**VERIFY:** Full session lifecycle works.

## Advanced: Individual Hooks

For fine-grained control:

1. `useCreateSessionKey` — Generate keypair
2. `useAddSessionKeyToContract` — On-chain registration
3. `useExecuteWithSession` — Execute without auth
4. `useGetSessionData` — Query status
5. `useRevokeSessionKey` — Disable session

## Security Best Practices

- Never persist session private keys in backend — browser/device memory only
- Always revoke sessions on user logout
- Default session expiry: 6 hours
- Monitor `remainingCalls` and prompt re-registration when low
- One active session per wallet

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Session keys not working" | Verify wallet is CHIPI type (not READY) |
| "Session expired" | Re-create and re-register session |
| "No remaining calls" | Session exhausted — re-register |
| "Registration failed" | Check wallet balance for on-chain tx |

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key session key-specific rules:
- **Session status badges**: Active = `bg-success/10 text-success border-success/20` (`--success` token). Expired = `bg-amber-500/10 text-amber-600 border-amber-500/20`. Revoked = `bg-muted text-muted-foreground`
- **Remaining calls**: `font-mono tabular-nums` — show "12 / 100 calls remaining" format
- **Expiry countdown**: `font-mono tabular-nums` with `Clock` icon (`h-4 w-4`) — "5h 23m remaining"
- **One-time auth prompt**: glassmorphic card explaining "This authorizes up to N transactions for M hours without re-authentication". Show `ShieldCheck` icon
- **Revocation**: `variant="destructive"` button with confirmation dialog. Show `AlertTriangle` icon + "This cannot be undone"
- **Session creation**: use `StepProgress` with steps `["Generate", "Register", "Active"]`. Show `Loader2 animate-spin` during registration
- **Session list**: card per session with status badge, remaining calls, expiry. Use `hover:-translate-y-0.5` on cards
- **Responsive**: session cards stack vertically on mobile, 2-column grid at `md:`
