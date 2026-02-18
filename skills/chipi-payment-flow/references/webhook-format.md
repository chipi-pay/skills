# Chipi Webhook Format

## Headers

| Header | Description |
|--------|-------------|
| `chipi-signature` | HMAC-SHA256 signature of the request body |
| `Content-Type` | `application/json` |

## Verification

```typescript
import crypto from "crypto";

function verifyWebhook(body: string, signature: string): boolean {
  const expected = crypto
    .createHmac("sha256", process.env.CHIPI_SECRET_KEY!) // sk_prod_ key
    .update(body)
    .digest("hex");
  return signature === expected;
}
```

## Payload

```json
{
  "event": "payment.completed",
  "txHash": "0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7",
  "amount": "10.00",
  "token": "USDC",
  "sender": "0x...",
  "recipient": "0x...",
  "timestamp": "2025-01-15T10:30:00Z",
  "status": "COMPLETED"
}
```

## Event Types

| Event | Description |
|-------|-------------|
| `payment.completed` | Payment confirmed on StarkNet |
| `payment.failed` | Payment reverted or rejected |

## Security

- Always verify the `chipi-signature` header
- Use `sk_prod_` (secret key) for verification — NEVER expose this key
- Reject requests with invalid or missing signatures
- Process webhooks idempotently (same txHash may arrive multiple times)
