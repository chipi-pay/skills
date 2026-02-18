# Chipi SDK — Wallet Hooks Reference

## useCreateWallet

Deploy a new CHIPI or READY wallet on StarkNet (gasless via Avnus).

### Parameters

| Param | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| encryptKey | string | Yes | — | PIN or passkey-derived key for wallet encryption |
| externalUserId | string | Yes | — | Your app's user ID (e.g., Clerk user ID) |
| chain | "STARKNET" | Yes | — | Blockchain network |
| walletType | "CHIPI" \| "READY" | No | "CHIPI" | CHIPI = session keys, READY = Argent X |
| usePasskey | boolean | No | false | Set true for passkey/biometric (RECOMMENDED) |
| bearerToken | string | Yes | — | Chipi API key (pk_prod_...) |

### Returns

| Field | Type | Description |
|-------|------|-------------|
| mutate / mutateAsync | function | Trigger wallet creation |
| data | object | { txHash, publicKey, credentialId? } |
| isPending | boolean | Loading state |
| isError | boolean | Error state |
| error | Error | Error details |

## useChipiWallet

All-in-one wallet hook: fetch, create, balance.

### Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| externalUserId | string | Yes | Your app's user ID |
| getBearerToken | () => Promise<string> | Yes | Function returning API key |
| defaultToken | "USDC" \| "ETH" \| "STRK" | No | Token for balance. Default: "USDC" |

### Returns

| Field | Type | Description |
|-------|------|-------------|
| wallet | object \| null | Wallet data |
| hasWallet | boolean | Whether user has a wallet |
| balance | string | Raw token balance |
| formattedBalance | string | Formatted (e.g., "$10.50") |
| createWallet | function | Create wallet function |
| isCreating | boolean | Creation in progress |
| isLoadingWallet | boolean | Loading state |
| refetchAll | function | Refresh all data |

## useGetWallet

Get wallet by external user ID.

### Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| externalUserId | string | Yes | Your app's user ID |
| bearerToken | string | Yes | Chipi API key |

## useMigrateWalletToPasskey

Convert PIN-encrypted wallet to passkey/biometric. After migration, old PIN stops working.

### Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| wallet | object | Yes | Existing wallet |
| currentEncryptKey | string | Yes | Current PIN |
| bearerToken | string | Yes | Chipi API key |

## useTransfer

Transfer USDC/ETH/STRK/custom tokens (gasless).

### Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| encryptKey | string | Yes | PIN or passkey |
| wallet | object | Yes | Sender wallet |
| token | "USDC" \| "ETH" \| "STRK" | Yes* | Token (*or use otherToken) |
| recipient | string | Yes | Recipient address (0x + 64 hex) |
| amount | string | Yes | Amount (human-readable) |
| bearerToken | string | Yes | Chipi API key |
| otherToken | object | No | Custom ERC-20: { name, address, symbol, decimals } |

## useGetTokenBalance

Fetch on-chain token balance.

### Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| chainToken | "USDC" \| "ETH" \| "STRK" | Yes | Token |
| chain | "STARKNET" | Yes | Chain |
| walletPublicKey | string | No | Wallet address (or use externalUserId) |
| externalUserId | string | No | User ID (or use walletPublicKey) |
| getBearerToken | () => Promise<string> | No | Required if using externalUserId |

## Wallet Types

| Type | Contract | Session Keys | Default |
|------|----------|-------------|---------|
| **CHIPI** | Chipi's own | **Yes** | **Yes** |
| READY | Argent X | No | No |

## Authentication

| Method | Security | UX | Default |
|--------|----------|-----|---------|
| **Passkey** | Strong — device-bound | Biometric tap | **Yes** |
| PIN | Weak — 4 digits | Type digits | No — warn user |
