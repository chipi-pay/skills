# Transaction States

## Lifecycle

```
PENDING -> PROCESSING -> COMPLETED
                      -> FAILED
```

## States

| State | Description | Action |
|-------|-------------|--------|
| PENDING | Transaction submitted, awaiting processing | Wait |
| PROCESSING | Transaction being executed on StarkNet | Wait — do NOT retry |
| COMPLETED | Transaction confirmed on-chain | Show success |
| FAILED | Transaction reverted or rejected | Show error, allow retry |

## Checking Status

```typescript
import { useGetSkuPurchase } from "@chipi-stack/nextjs";

const { data } = useGetSkuPurchase({
  purchaseId: "purchase_123",
  bearerToken: process.env.NEXT_PUBLIC_CHIPI_API_KEY!,
});

switch (data.status) {
  case "PENDING": break;     // Show spinner
  case "PROCESSING": break;  // Show "processing on StarkNet"
  case "COMPLETED": break;   // Show success with txHash
  case "FAILED": break;      // Show error, offer retry
}
```

## Important Notes

- Do NOT retry while status is PROCESSING — can cause duplicate transactions
- COMPLETED transactions have a `txHash` verifiable on StarkNet explorer
- FAILED transactions can be safely retried
- Average confirmation time: ~15-30 seconds on StarkNet
