# Address Validation Guide

Validate and debug address formats for StarkNet, EVM, and Solana.

## StarkNet Address Format

```
Format:  0x + 64 hexadecimal characters
Length:  66 characters total
Regex:   /^0x[0-9a-fA-F]{64}$/
Case:    Case-insensitive (but normalize to lowercase)
Example: 0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7
```

### Validation

```tsx
function isValidStarkNetAddress(address: string): boolean {
  return /^0x[0-9a-fA-F]{64}$/.test(address);
}
```

### Common StarkNet Address Issues

| Issue | Example | Fix |
|-------|---------|-----|
| Leading zeros stripped | `0x49d365...` (63 hex chars) | Pad: `0x0` + remaining to reach 64 |
| Missing 0x | `049d36570d4e...` | Prepend `0x` |
| Trailing space | `0x049d...dc7 ` | `.trim()` |
| Uppercase | `0x049D3657...` | Valid but normalize with `.toLowerCase()` |
| Too short | `0x049d3657` | Not a valid address — get the full address |

## EVM Address Format (Common Mistake)

```
Format:  0x + 40 hexadecimal characters
Length:  42 characters total
Regex:   /^0x[0-9a-fA-F]{40}$/
Example: 0x742d35Cc6634C0532925a3b844Bc9e7595f2bD00
```

### Detecting EVM vs StarkNet

```tsx
function detectAddressChain(address: string): "starknet" | "evm" | "solana" | "unknown" {
  if (/^0x[0-9a-fA-F]{64}$/.test(address)) return "starknet";
  if (/^0x[0-9a-fA-F]{40}$/.test(address)) return "evm";
  if (/^[1-9A-HJ-NP-Za-km-z]{32,44}$/.test(address)) return "solana";
  return "unknown";
}
```

**If a user provides an EVM address:**
- "This looks like an Ethereum address (40 hex chars). StarkNet addresses are 64 hex chars. Please provide your StarkNet wallet address."

## Solana Address Format (Common Mistake)

```
Format:  Base58-encoded, 32-44 characters
Charset: 1-9, A-H, J-N, P-Z, a-k, m-z (no 0, O, I, l)
Regex:   /^[1-9A-HJ-NP-Za-km-z]{32,44}$/
Example: 7EcDhSYGxXyscszYEp35KHN8sADaQT4P8gGCsMVzWmGq
```

**If a user provides a Solana address:**
- "This looks like a Solana address (Base58 format). StarkNet uses hex addresses starting with 0x. Please provide your StarkNet wallet address."

## Token Contract Addresses (StarkNet Mainnet)

| Token | Address |
|-------|---------|
| USDC | `0x053c91253bc9682c04929ca02ed00b3e423f6710d2ee7e0d5ebb06f3ecf368a8` |
| ETH | `0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7` |
| STRK | `0x04718f5a0fc34cc1af16a1cdee98ffb20c31f5cd61d6ab07201858f4287c938d` |
| USDT | `0x068f5c6a61780768455de69077e07e89787839bf8166decfbf92b645209c0fb8` |
| DAI | `0x00da114221cb83fa859dbdb4c44beeaa0bb37c7537ad5ae66fe5e0efd20e6eb3` |
| WBTC | `0x03fe2b97c1fd336e750087d68b9b867997fd64a2661ff3ca5a7c771641e8e7ac` |

### Shorthand Tokens in Chipi SDK

When using `useTransfer` or `useGetTokenBalance`, you can pass:
- `"USDC"` — maps to the USDC contract address above
- `"ETH"` — maps to the ETH contract address above
- `"STRK"` — maps to the STRK contract address above
- Or pass the full hex address for any other ERC-20 token

## Full Validation Utility

```tsx
interface AddressValidation {
  isValid: boolean;
  chain: "starknet" | "evm" | "solana" | "unknown";
  error?: string;
  suggestion?: string;
}

function validateAddress(address: string): AddressValidation {
  const trimmed = address.trim();

  if (/^0x[0-9a-fA-F]{64}$/.test(trimmed)) {
    return { isValid: true, chain: "starknet" };
  }

  if (/^0x[0-9a-fA-F]{40}$/.test(trimmed)) {
    return {
      isValid: false,
      chain: "evm",
      error: "This is an EVM address (40 hex chars)",
      suggestion: "StarkNet addresses are 64 hex chars. Please provide your StarkNet address.",
    };
  }

  if (/^[1-9A-HJ-NP-Za-km-z]{32,44}$/.test(trimmed)) {
    return {
      isValid: false,
      chain: "solana",
      error: "This is a Solana address (Base58)",
      suggestion: "StarkNet addresses start with 0x and are 64 hex chars.",
    };
  }

  if (/^0x[0-9a-fA-F]+$/.test(trimmed)) {
    const hexLen = trimmed.length - 2;
    return {
      isValid: false,
      chain: "unknown",
      error: `Address has ${hexLen} hex chars (expected 64)`,
      suggestion: hexLen < 64
        ? `Pad with ${64 - hexLen} leading zeros after 0x`
        : "Address is too long. Check for extra characters.",
    };
  }

  return {
    isValid: false,
    chain: "unknown",
    error: "Unrecognized address format",
    suggestion: "StarkNet addresses are 0x followed by 64 hex characters.",
  };
}
```
