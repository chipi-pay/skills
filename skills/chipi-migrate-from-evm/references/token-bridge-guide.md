# Bridging Assets to StarkNet

How to move tokens from EVM chains (Ethereum, Base, Polygon, Arbitrum, Optimism) or Solana to StarkNet.

## Why Bridge?

When migrating an app from EVM or Solana to StarkNet, your users' existing tokens are on the old chain. They need to move their assets to StarkNet to use your new app. This guide covers the main bridging options.

## Option 1: StarkGate (Official Ethereum ↔ StarkNet Bridge)

**Best for:** Ethereum mainnet → StarkNet

- **URL:** https://starkgate.starknet.io
- **Supported tokens:** ETH, USDC, USDT, DAI, WBTC, and more
- **Direction:** Ethereum L1 → StarkNet L2 (and back)
- **Time:** L1 → L2: ~15-30 minutes | L2 → L1: ~12 hours (finality period)
- **Fees:** Ethereum gas fees for the bridge transaction

### How it works
1. User connects MetaMask (Ethereum) and a StarkNet wallet
2. Selects token and amount to bridge
3. Approves the bridge transaction on Ethereum
4. Tokens appear on StarkNet after confirmation

### For your app
- Link to StarkGate from an "Add Funds" page
- After bridging, users can use their StarkNet tokens in your Chipi-powered app
- The StarkNet USDC address is: `0x053c91253bc9682c04929ca02ed00b3e423f6710d2ee7e0d5ebb06f3ecf368a8`

## Option 2: LayerSwap

**Best for:** L2 → StarkNet (Base, Arbitrum, Optimism, Polygon) and CEX → StarkNet

- **URL:** https://layerswap.io
- **Supported chains:** Ethereum, Base, Arbitrum, Optimism, Polygon, Solana, and 20+ more
- **Supported tokens:** ETH, USDC, USDT, and chain-specific tokens
- **Time:** 2-10 minutes (fast!)
- **Fees:** Small bridging fee (0.1-0.5%)

### Why LayerSwap for migration
- Supports Solana → StarkNet (StarkGate doesn't)
- Supports L2 → StarkNet directly (no need to go through Ethereum mainnet)
- Much faster than StarkGate for most routes
- Supports CEX withdrawals directly to StarkNet (Binance, Coinbase, etc.)

### How it works
1. User selects source chain/CEX and destination (StarkNet)
2. Enters StarkNet wallet address (from Chipi)
3. Sends tokens on source chain
4. Tokens arrive on StarkNet in minutes

## Option 3: Orbiter Finance

**Best for:** Fast L2-to-L2 bridging

- **URL:** https://orbiter.finance
- **Supported chains:** Ethereum, StarkNet, Arbitrum, Optimism, Base, zkSync, Polygon, and more
- **Time:** 1-5 minutes
- **Fees:** Variable, competitive with LayerSwap

### How it works
- Uses a maker network for fast cross-chain transfers
- Good for small to medium amounts
- Simple UI with wallet connection

## Recommendation for Your App

### For EVM → StarkNet migration:
1. **Default recommendation:** LayerSwap — supports the most chains, fastest, best UX
2. **Ethereum mainnet specifically:** StarkGate — official bridge, most trusted
3. **Quick L2 transfers:** Orbiter — fastest for small amounts

### For Solana → StarkNet migration:
1. **Only option:** LayerSwap — the only major bridge supporting Solana → StarkNet directly
2. Alternative: Solana → CEX → StarkNet withdrawal via LayerSwap

### Integration pattern:
```tsx
// Add a "Fund Your Wallet" button that links to the recommended bridge
function FundWallet({ walletAddress }: { walletAddress: string }) {
  return (
    <div>
      <p>Your StarkNet address: {walletAddress}</p>
      <p>Bridge tokens from another chain:</p>
      <a
        href={`https://layerswap.io/?destNetwork=STARKNET_MAINNET&destAddress=${walletAddress}`}
        target="_blank"
        rel="noopener noreferrer"
      >
        Bridge via LayerSwap
      </a>
    </div>
  );
}
```

## Token Address Reference

After bridging, tokens arrive at these StarkNet addresses:

| Token | StarkNet Address |
|-------|-----------------|
| USDC | `0x053c91253bc9682c04929ca02ed00b3e423f6710d2ee7e0d5ebb06f3ecf368a8` |
| ETH | `0x049d36570d4e46f48e99674bd3fcc84644ddd6b96f7c741b1562b82f9e004dc7` |
| USDT | `0x068f5c6a61780768455de69077e07e89787839bf8166decfbf92b645209c0fb8` |
| DAI | `0x00da114221cb83fa859dbdb4c44beeaa0bb37c7537ad5ae66fe5e0efd20e6eb3` |
| WBTC | `0x03fe2b97c1fd336e750087d68b9b867997fd64a2661ff3ca5a7c771641e8e7ac` |
| STRK | `0x04718f5a0fc34cc1af16a1cdee98ffb20c31f5cd61d6ab07201858f4287c938d` |

## Important Notes

- **Bridge time varies:** L1 → L2 is slower (15-30 min) than L2 → L2 (1-10 min)
- **Withdrawal to L1 is slow:** StarkNet → Ethereum takes ~12 hours due to ZK proof finality
- **Double-check addresses:** StarkNet addresses are 0x + 64 hex chars (longer than EVM's 40 chars)
- **Test with small amounts first** before bridging large sums
- **Gas on source chain:** Users still need gas on the source chain to initiate the bridge — but once on StarkNet, everything through Chipi is gasless
