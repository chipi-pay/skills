---
name: chipi-passkey-troubleshooting
description: Debug passkey and PIN authentication issues in Chipi integrations. Diagnose encryption version mismatches, PRF failures, migration errors, multi-device problems, and decryption failures. Use when user says "passkey not working", "decryption failed", "wrong key", "migration failed", "biometric error", or "PRF not supported".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Debug Passkey & PIN Authentication

Systematic debugging guide for passkey and PIN authentication issues in Chipi wallets.

## When in Doubt, Ask

Before debugging, ask the user for:
- **Error message** — exact text
- **Auth method** — PIN or passkey?
- **How was the wallet created?** — MCP components, SDK hooks, or custom?
- **Browser/device** — Chrome, Safari, mobile? (matters for PRF)
- **Did it ever work?** — new setup vs regression?

## Step 1: Identify the Problem

```text
Authentication error
├─ Decryption failed? → Step 2
│  ("wrong key", empty result, corrupted data)
├─ PRF not working? → Step 3
│  ("NotAllowedError", null PRF output, biometric cancelled)
├─ Migration failed? → Step 4
│  (PIN → passkey migration errors)
├─ Multi-device issue? → Step 5
│  (works on one device, fails on another)
├─ Encryption version mismatch? → Step 6
│  (MCP vs SDK incompatibility)
└─ PIN issues? → Step 7
   (wrong PIN, change PIN, lost PIN)
```

## Step 2: Decryption Failed

| Error | Cause | Fix |
|-------|-------|-----|
| "Decryption failed - wrong key or corrupted data" | Encryption key doesn't match what was used to encrypt | Check encryption version (Step 6). Verify correct auth method. |
| Empty string after decrypt | Key derived differently than at creation time | PRF salt or derivation method mismatch. See Step 6. |
| "Cannot read property 'toString' of undefined" | crypto-es version mismatch | Check crypto-es version: MCP uses v3.1.2, SDK uses v2.1.0. |

**Quick diagnostic:**
```typescript
// Check which auth method the wallet uses
const metadata = user.unsafeMetadata as PasskeyMetadata;
const authMethod = metadata?.passkeys?.authMethod; // "prf" | "pin" | undefined
const encryptionVersion = metadata?.passkeys?.encryptionVersion; // "sdk-v1" | "mcp-v1" | undefined
console.log({ authMethod, encryptionVersion });
```

## Step 3: PRF Not Working

| Error | Cause | Fix |
|-------|-------|-----|
| "NotAllowedError" | User cancelled biometric prompt | Prompt again. Check if device has biometric enrolled. |
| PRF output is null | Browser doesn't support PRF extension | Check browser: Chrome 116+, Edge 116+, Safari 17.4+. Firefox not supported. |
| `prf.enabled === false` | Credential was created without PRF | Re-register passkey with PRF enabled. |
| Works in Chrome, fails in Safari | Safari PRF support is newer (17.4+) | Check Safari version. Fall back to PIN on unsupported browsers. |

**Quick diagnostic:**
```typescript
import { isPRFSupported } from "@/lib/prf-encryption";
const supported = await isPRFSupported();
console.log("PRF supported:", supported);
```

## Step 4: Migration Failed (PIN → Passkey)

| Error | Stage | Fix |
|-------|-------|-----|
| "Decryption failed" during PIN verify | Step 1 (verify PIN) | Wrong PIN entered. Retry. |
| "NotAllowedError" during registration | Step 2 (register passkey) | User cancelled biometric. Old PIN still works. |
| "PRF not enabled on credential" | Step 3 (authenticate) | Passkey was created without PRF. Delete and re-register. |
| "Failed to update user" | Step 4 (re-encrypt) | Clerk API error. Old PIN still works. Retry. |

**Migration is safe**: If any step fails, the old PIN continues to work. No data is lost.

## Step 5: Multi-Device Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Works on iPhone, fails on laptop | Different passkey credentials per device | Register a new passkey on the laptop via Passkey Manager. |
| "Credential not found" on new device | Passkey is device-bound (not synced) | Register a new passkey on the new device. |
| "Counter mismatch" | Cloned credential detected | Security measure. Re-register passkey. |

**How multi-device works**: Each device has its own passkey credential. All use the same PRF salt, so they derive the same encryption key. The encrypted private key in Clerk metadata is shared across devices.

## Step 6: Encryption Version Mismatch

**This is the most common issue when mixing MCP components and SDK.**

Two encryption versions exist:

| Version | PRF Salt | Key Derivation | Used By |
|---------|----------|---------------|---------|
| `mcp-v1` | `chipi-wallet-encryption-v1` (UTF-8 bytes) | SHA256(base64(prfOutput)) | MCP components (legacy) |
| `sdk-v1` | `chipi-wallet-encryption-key-v1` (base64url) | arrayBufferToHex(prfOutput) | SDK @chipi-stack/chipi-passkey |

**A wallet encrypted with mcp-v1 CANNOT be decrypted with sdk-v1, and vice versa.**

**Diagnostic:**
```typescript
const metadata = user.unsafeMetadata as PasskeyMetadata;
const version = metadata?.passkeys?.encryptionVersion;
// "sdk-v1" → SDK encryption
// "mcp-v1" → MCP encryption
// undefined → legacy wallet, try both
```

**Fix for legacy wallets** (no `encryptionVersion` in metadata):
- MCP's updated `tryDecryptWithVersions()` tries both methods automatically
- After successful decryption, stamps the correct version in metadata
- Going forward, all new wallets use `sdk-v1`

**Fix for mixed implementations:**
- If using MCP components: update to latest MCP (PR #65+) which aligns with SDK
- If using SDK directly: no changes needed
- If using both: ensure MCP is updated, then legacy wallets auto-migrate on next sign

## Step 7: PIN Issues

| Problem | Fix |
|---------|-----|
| Forgot PIN | No recovery if wallet is PIN-only. Consider migrating to passkey first next time. |
| Change PIN | Decrypt with old PIN, re-encrypt with new PIN via `useUpdateWalletEncryption`. |
| PIN works on MCP but not SDK | Shouldn't happen — PIN encryption is the same. Check if PIN has leading zeros (they matter). |

## Metadata Reference

```typescript
interface PasskeyMetadata {
  chipiWallet?: {
    publicKey: string;
    encryptedPrivateKey: string;
    txHash?: string;
  };
  passkeys?: {
    enabled: boolean;
    credentials: PassKey[];
    authMethod?: "prf" | "pin";
    prfEnabled?: boolean;
    encryptionVersion?: "mcp-v1" | "sdk-v1"; // Added in MCP PR #65
    lastAuthAt?: number;
  };
}
```

Check metadata at: Clerk Dashboard → Users → select user → Unsafe Metadata.
