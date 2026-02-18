# Calldata Formatting for StarkNet Contracts

## Basic Types

| Type | Encoding | Example |
|------|----------|---------|
| felt252 | Hex or decimal string | "0x1234" or "4660" |
| u256 | Two felt252 [low, high] | ["1000000", "0"] |
| address | 0x + 64 hex chars | "0x049d36..." |
| bool | "1" or "0" | "1" |

## u256 Encoding

StarkNet u256 = two 128-bit parts:
- `low`: Lower 128 bits
- `high`: Upper 128 bits

For amounts < 2^128 (all practical amounts):
```typescript
const low = amount.toString();
const high = "0";
```

## Token Amount Encoding

| Token | Decimals | "1.0" encoded | Calldata |
|-------|----------|---------------|----------|
| USDC | 6 | 1000000 | ["1000000", "0"] |
| ETH | 18 | 1000000000000000000 | ["1000000000000000000", "0"] |
| STRK | 18 | 1000000000000000000 | ["1000000000000000000", "0"] |

## Common Call Patterns

### ERC-20 Transfer
```typescript
{ contractAddress: token, entrypoint: "transfer", calldata: [recipient, amountLow, amountHigh] }
```

### ERC-20 Approve
```typescript
{ contractAddress: token, entrypoint: "approve", calldata: [spender, amountLow, amountHigh] }
```

### Approve + Call (Multi-call)
```typescript
[
  { contractAddress: token, entrypoint: "approve", calldata: [protocol, amount, "0"] },
  { contractAddress: protocol, entrypoint: "deposit", calldata: [amount, "0"] },
]
```
