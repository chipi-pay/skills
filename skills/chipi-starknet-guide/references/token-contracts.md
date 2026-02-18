# StarkNet Token Contracts

## Supported Tokens

| Token | Decimals | Description |
|-------|----------|-------------|
| USDC | 6 | USD Coin — primary payment token |
| ETH | 18 | Ether — native gas token (gasless via Chipi) |
| STRK | 18 | StarkNet token |

## Decimal Conversion

### USDC (6 decimals)
| Human | Raw |
|-------|-----|
| 1.00 | 1000000 |
| 10.50 | 10500000 |
| 100.00 | 100000000 |

### ETH / STRK (18 decimals)
| Human | Raw |
|-------|-----|
| 1.0 | 1000000000000000000 |
| 0.1 | 100000000000000000 |
| 0.01 | 10000000000000000 |

## Custom ERC-20 Tokens

```typescript
const otherToken = {
  name: "My Token",
  address: "0x...",
  symbol: "MTK",
  decimals: 18,
};

await transfer({
  encryptKey, wallet, otherToken,
  recipient: "0x...", amount: "100", bearerToken,
});
```

## u256 Encoding

StarkNet amounts are u256 = [low, high]:
- For amounts < 2^128: low = amount, high = "0"
- Example: 10 USDC = ["10000000", "0"]
