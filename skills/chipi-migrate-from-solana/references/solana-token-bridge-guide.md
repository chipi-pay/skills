# Bridging Assets from Solana to StarkNet

How to move tokens from Solana to StarkNet for use in your Chipi-powered app.

## Primary Route: LayerSwap

**LayerSwap is a well-established bridge supporting direct Solana → StarkNet transfers.** Other bridges like StarkGate, rhino.fi, and Orbiter may also offer Solana routes — check their current availability.

- **URL:** https://layerswap.io
- **Supported tokens:** SOL, USDC (SPL), USDT (SPL)
- **Time:** 2-10 minutes
- **Fees:** 0.1-0.5% bridging fee
- **Minimum:** Varies by token (check LayerSwap UI)

## Step-by-Step Flow

### 1. Get the user's StarkNet address
```tsx
import { useChipiWallet } from "@chipi-stack/nextjs";

const { wallet } = useChipiWallet();
const starknetAddress = wallet?.publicKey;
// This is the destination address for the bridge
```

### 2. Direct to LayerSwap
```tsx
function BridgeFromSolana({ walletAddress }: { walletAddress: string }) {
  const bridgeUrl = `https://layerswap.io/?sourceExchangeName=SOLANA_MAINNET&destNetwork=STARKNET_MAINNET&destAddress=${walletAddress}&asset=USDC`;

  return (
    <a href={bridgeUrl} target="_blank" rel="noopener noreferrer">
      Bridge USDC from Solana
    </a>
  );
}
```

### 3. User completes bridge on LayerSwap
1. Connects Solana wallet (Phantom, Solflare, etc.)
2. Selects token and amount
3. Confirms the transaction on Solana
4. Waits 2-10 minutes
5. Tokens arrive on StarkNet at the Chipi wallet address

### 4. Verify arrival
```tsx
const { data: balance } = useGetTokenBalance({
  walletAddress: starknetAddress,
  tokenAddress: "USDC",
});
// Balance updates after bridge completes
```

## Alternative: Solana → CEX → StarkNet

If LayerSwap doesn't support the specific token:

1. **Send tokens** from Solana wallet to a CEX (Binance, Coinbase, OKX)
2. **Withdraw from CEX** to StarkNet address via LayerSwap or direct StarkNet withdrawal
3. Some CEXs support direct StarkNet withdrawal (Binance, OKX)

## Wormhole Considerations

Wormhole supports Solana ↔ StarkNet bridging, but:
- Wrapped tokens (wUSDC) are different from native StarkNet USDC
- Additional steps needed to swap wrapped → native tokens
- LayerSwap is simpler for most use cases
- Only consider Wormhole for tokens not supported by LayerSwap

## Token Address Mapping

| Token | Solana (SPL) | StarkNet |
|-------|-------------|----------|
| USDC | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` | `0x053c91253bc9682c04929ca02ed00b3e423f6710d2ee7e0d5ebb06f3ecf368a8` |
| ETH | Wrapped: `7vfCXTUXx5WJV5JADk17DUJ4ksgau7utNKj4b963voxs` | `0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7` |
| USDT | `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB` | `0x068f5c6a61780768455de69077e07e89787839bf8166decfbf92b645209c0fb8` |
| STRK | N/A (StarkNet native) | `0x04718f5a0fc34cc1af16a1cdee98ffb20c31f5cd61d6ab07201858f4287c938d` |
| SOL | Native | N/A (no native SOL on StarkNet — bridge to ETH or USDC) |

## Important Notes

- **SOL doesn't exist on StarkNet** — bridge to USDC or ETH instead
- **Bridge time varies:** 2-10 minutes is typical, but congestion can delay
- **Test with small amounts first** before bridging large sums
- **Double-check addresses:** StarkNet addresses are 0x + 64 hex chars (not Base58)
- **Gas on Solana side:** Users need SOL on Solana to pay for the bridge transaction. Once on StarkNet via Chipi, everything is gasless.
- **Decimal differences:** SOL has 9 decimals, ETH has 18, USDC has 6 on both chains
