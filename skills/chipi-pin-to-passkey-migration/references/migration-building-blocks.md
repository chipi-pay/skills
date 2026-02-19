# PIN-to-Passkey Migration — Building Blocks

## Core Functions

### reEncryptPrivateKey(encryptedPrivateKey, oldKey, newKey) → string

Re-encrypts a private key from one encryption key to another. Used during migration to switch from PIN-based encryption to passkey-derived encryption.

```typescript
import { reEncryptPrivateKey } from "@/lib/prf-encryption";

const newEncryptedKey = reEncryptPrivateKey(
  encryptedPrivateKey,  // Current AES-encrypted private key (stored in Clerk)
  oldPin,               // Current PIN (4-digit string)
  newPasskeyKey         // PRF-derived key (32-char base64 string)
);
// Returns: new AES-encrypted private key string
```

Internally: `decrypt(encryptedPrivateKey, oldKey)` → plaintext → `encrypt(plaintext, newKey)`. The plaintext private key exists only in memory during this call.

### decryptPrivateKey(encryptedPrivateKey, key) → string

Decrypts an AES-encrypted private key. Throws if the key is wrong.

```typescript
import { decryptPrivateKey } from "@/lib/prf-encryption";

try {
  const privateKey = decryptPrivateKey(encryptedPrivateKey, pin);
  // PIN is correct
} catch {
  // Wrong PIN — decryption failed
}
```

Used in migration to verify the user's PIN before proceeding.

### deriveKeyFromPRF(prfOutput) → string

Derives an encryption key from PRF output. Takes the first 32 characters of the base64-encoded PRF output.

```typescript
import { deriveKeyFromPRF } from "@/lib/prf-encryption";

const encryptionKey = deriveKeyFromPRF(prfOutput);
// Returns: 32-char base64 string used as AES encryption key
```

### extractPRFOutput(extensionResults) → string | null

Extracts PRF output from WebAuthn authentication extension results.

```typescript
import { extractPRFOutput } from "@/lib/prf-encryption";

const prfOutput = extractPRFOutput(authResponse.clientExtensionResults);
if (!prfOutput) throw new Error("PRF failed");
```

### getPRFAuthenticationExtension() → Record<string, unknown>

Returns the WebAuthn extension config for PRF during authentication.

```typescript
import { getPRFAuthenticationExtension } from "@/lib/prf-encryption";

const optionsWithPRF = {
  ...options,
  extensions: {
    ...options.extensions,
    ...getPRFAuthenticationExtension(),
  },
};
```

### storeAuthMethod(method) → void

Stores the auth method in localStorage. Used after migration to tell `TransactionSigner` to show passkey UI.

```typescript
import { storeAuthMethod } from "@/lib/prf-encryption";

storeAuthMethod("prf"); // or "pin"
```

## Clerk Metadata Shape

### PasskeyMetadata Interface

```typescript
interface PasskeyMetadata {
  chipiWallet?: {
    publicKey: string;
    encryptedPrivateKey: string;
    txHash?: string;
  };
  passkeys?: PasskeySettings;
}

interface PasskeySettings {
  enabled: boolean;
  credentials: PassKey[];
  currentChallenge?: string;
  challengeExpiresAt?: number;
  lastAuthAt?: number;
  authMethod?: "prf" | "pin";
  prfEnabled?: boolean;
}
```

After migration, update both:
```typescript
await user.update({
  unsafeMetadata: {
    ...user.unsafeMetadata,
    chipiWallet: {
      ...existing.chipiWallet,
      encryptedPrivateKey: newEncryptedKey, // Re-encrypted with passkey-derived key
    },
    passkeys: {
      ...existing.passkeys,
      authMethod: "prf",      // Changed from "pin"
      prfEnabled: true,        // Enable PRF flag
    },
  },
});
```

## Passkey API Routes

### POST /api/passkeys/register
Generate registration options. Send `{ enablePRF: true }` in body.

**Response:** `PublicKeyCredentialCreationOptionsJSON` + `{ prfRequested: true }`

### PUT /api/passkeys/register
Verify registration and store credential. Send:
```json
{
  "response": "<WebAuthn registration response>",
  "name": "Migration Passkey",
  "prfEnabled": true,
  "authMethod": "prf"
}
```

### POST /api/passkeys/authenticate
Generate authentication options. No body needed.

**Response:** `PublicKeyCredentialRequestOptionsJSON`

### PUT /api/passkeys/authenticate
Verify authentication. Send:
```json
{
  "response": "<WebAuthn authentication response>"
}
```

## Safety Guarantees

1. **PIN verified first** — `decryptPrivateKey()` proves wallet ownership before any passkey operations
2. **Passkey registered AND authenticated** — Both steps must succeed before re-encryption (need PRF key from authentication)
3. **Atomic metadata update** — Single `user.update()` call writes both the new encrypted key and auth method
4. **Failure recovery** — If any step fails after PIN verification, old encrypted key is still in Clerk and PIN still works
5. **No plaintext storage** — Private key exists only in memory during `reEncryptPrivateKey()`, never written to disk/storage
