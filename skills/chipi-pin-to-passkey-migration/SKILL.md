---
name: chipi-pin-to-passkey-migration
description: Migrate a PIN-encrypted Chipi wallet to passkey (PRF) authentication. Handles PIN verification, passkey registration, PRF key derivation, private key re-encryption, and metadata update. Use when user says "migrate to passkey", "upgrade from PIN", "switch to biometric", "replace PIN with passkey", or "passkey migration".
license: MIT
metadata:
  author: Chipi Pay
  version: 1.0.0
  mcp-server: chipi-registry
---

# Chipi PIN-to-Passkey Migration

Upgrade a PIN-encrypted wallet to passkey (PRF) authentication. After migration, the old PIN stops working — only passkey/biometric works.

## Prerequisites

- Existing Chipi wallet created with PIN authentication
- Device with WebAuthn PRF support (Chrome 116+, Edge 116+, Safari 17.4+)
- Clerk authentication configured (for metadata storage)
- Passkey API routes in place (`/api/passkeys/register`, `/api/passkeys/authenticate`)

## Step 1: Get the Migration Component

If MCP is connected, call: `get_component_code("migrate-to-passkey-dialog")`

This provides:
- `MigrateToPasskeyDialog` — Full migration flow UI
- `prf-encryption.ts` — Encryption/decryption utilities including `reEncryptPrivateKey`
- `passkeys.ts` — Passkey types and helpers
- Passkey API routes (register + authenticate)

Install dependencies:
```bash
npm install @simplewebauthn/browser @simplewebauthn/types crypto-es
npx shadcn@latest add button dialog input-otp --y
```

**VERIFY:** Migration component and dependencies are installed.

## Step 2: Ensure Passkey API Routes Exist

Check that these routes exist in the project:
- `app/api/passkeys/register/route.ts` — POST (generate options), PUT (verify registration)
- `app/api/passkeys/authenticate/route.ts` — POST (generate options), PUT (verify auth)

These routes use `@simplewebauthn/server` for WebAuthn verification and store passkey credentials in Clerk user metadata.

If missing, the `get_component_code("migrate-to-passkey-dialog")` call includes them.

Server-side dependency:
```bash
npm install @simplewebauthn/server
```

**VERIFY:** Both passkey API routes exist and return proper responses.

## Step 3: Add Migration Trigger

Show the migration button only for users with `authMethod === "pin"`:

```tsx
import { useUser } from "@clerk/nextjs";
import { MigrateToPasskeyDialog } from "@/components/migrate-to-passkey-dialog";
import { getStoredAuthMethod } from "@/lib/prf-encryption";
import type { PasskeyMetadata } from "@/lib/passkeys";

function WalletSettings() {
  const { user } = useUser();
  const [showMigrate, setShowMigrate] = useState(false);

  const metadata = user?.unsafeMetadata as PasskeyMetadata | undefined;
  const authMethod = metadata?.passkeys?.authMethod || getStoredAuthMethod() || "pin";
  const hasWallet = !!metadata?.chipiWallet;

  // Only show for PIN users with an existing wallet
  if (!hasWallet || authMethod !== "pin") return null;

  return (
    <>
      <Button onClick={() => setShowMigrate(true)}>
        Upgrade to Passkey
      </Button>
      <MigrateToPasskeyDialog
        open={showMigrate}
        onClose={() => setShowMigrate(false)}
        onSuccess={() => {
          setShowMigrate(false);
          // Refresh wallet state
        }}
      />
    </>
  );
}
```

**VERIFY:** Migration button appears only for PIN-authenticated wallet users.

## Step 4: Test the Full Flow

1. Create a wallet with PIN (use `create-wallet-dialog` with PIN option)
2. Open migration dialog
3. Enter the correct PIN (try wrong PIN first to verify error handling)
4. Follow biometric prompt to register passkey
5. Follow biometric prompt again to authenticate (derives PRF key)
6. Verify success screen appears
7. Try signing a transaction — should use passkey (biometric prompt, no PIN input)
8. Verify old PIN no longer works for signing

**VERIFY:** Full migration flow works end-to-end.

## Step 5: Custom Storage Adapters

The default component stores encrypted private keys in Clerk `unsafeMetadata`. If your project uses a different storage backend:

### Database Storage
Replace the `user.update()` call in the migration component with your DB update:

```tsx
// Instead of Clerk metadata update:
await updateWalletEncryption(userId, {
  encryptedPrivateKey: newEncryptedPrivateKey,
  authMethod: "prf",
});
```

### Supabase/Firebase
Same pattern — replace the metadata update with your auth provider's user data update.

The core encryption logic (`reEncryptPrivateKey`) is storage-agnostic — it takes the old encrypted key and returns a new one. Only the storage step needs adaptation.

## Security Notes

- **Atomic safety:** If any step fails after PIN verification, the old encrypted key remains untouched in storage. The user can still use their PIN.
- **No plaintext persistence:** The decrypted private key exists only in memory during `reEncryptPrivateKey()` and is immediately re-encrypted.
- **PRF determinism:** The same passkey + salt always produces the same encryption key, so the wallet can be decrypted on subsequent authentications.
- **One-way migration:** After metadata update, PIN-based decryption will fail because the encrypted payload has changed. This is intentional.

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Wrong PIN" on first attempt | User entered incorrect PIN | Try again — decryption is tested before any passkey steps |
| "PRF extension not available" | Device/browser doesn't support PRF | User cannot migrate; they need Chrome 116+, Edge 116+, or Safari 17.4+ |
| "Registration was cancelled" | User dismissed biometric prompt | Retry — PIN is still valid, nothing has changed |
| "Failed to derive encryption key" | PRF output was null | Retry authentication step; may be a browser issue |
| Passkey works but old flows still show PIN | `authMethod` not updated in metadata | Check that `storeAuthMethod("prf")` was called and metadata update succeeded |
| Migration succeeds but TransactionSigner shows PIN | localStorage has stale `chipi_auth_method` | Clear localStorage or call `storeAuthMethod("prf")` explicitly |

## UI Guidance

> **Load `chipi-frontend-design` before generating any UI for this feature.**

Key migration-specific rules:
- **Step progress**: use `StepProgress` component with 5 steps: `["Verify PIN", "Register", "Authenticate", "Encrypt", "Done"]`. Active step uses `--accent` token, completed steps filled solid
- **Step labels**: show current step label below the progress bar in `text-sm font-medium text-muted-foreground` — users must always know which step they are on
- **Safety messaging**: show `ShieldCheck` icon (`h-5 w-5 text-success`) + "Your PIN still works until migration completes" in `text-muted-foreground`. This is critical for user confidence — display it on every step except Done
- **PIN input**: use `InputOTP` with 4 masked slots (`mask` prop). Each slot: `w-14 h-14 text-xl border-2`. Disable during processing with `disabled={isProcessing}`. Auto-focus first slot on mount
- **PIN error state**: on wrong PIN, shake the input group (`animate-shake` for 300ms), clear all slots, re-focus first slot. Show "Incorrect PIN — please try again" in `text-sm text-destructive`
- **Biometric prompt**: show `Fingerprint` icon (`h-12 w-12 text-accent animate-pulse`) centered with "Follow your device's biometric prompt" in `text-sm text-muted-foreground`
- **Processing guard**: show "Do not close this window..." in `text-xs text-muted-foreground animate-pulse` during all processing steps. Hide close button with `showCloseButton={false}`. Disable dialog escape/backdrop click
- **Encryption step**: show `Lock` icon + "Re-encrypting your private key..." — never expose raw key material in the UI. Progress is indeterminate (spinner, not percentage)
- **Success**: SVG checkmark-draw celebration (`animate-checkmark` + `animate-scale-bounce`) in emerald circle (`bg-emerald-500/20 border-2 border-emerald-500`). Message: "Your wallet is now protected by your passkey". Show a "Done" button that closes the dialog
- **Error recovery**: distinguish PIN errors (show PIN input again) from passkey errors (show Cancel + Retry buttons). Actionable messages: "Passkey registration was cancelled — tap Retry to try again"
- **Focus management**: trap focus inside the dialog for the entire flow. On success dismiss, return focus to the trigger button that opened the dialog
- **Responsive**: dialog uses `sm:max-w-md`. PIN input slots scale down to `w-12 h-12` on very small screens if needed. Touch targets for Cancel/Retry buttons: minimum 44x44px
