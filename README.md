# Chipi Skills

Agent skills for building with [Chipi](https://www.chipipay.com/) — self-custody StarkNet wallets, USDC payments, and service rails.

## What is Chipi?

Chipi lets you add non-custodial StarkNet wallets to your app in minutes. Passkey-secured, gasless, with session keys for frictionless UX.

**Works with:** Next.js, React, Expo (React Native), or any REST API client (Python, Go, etc.)

## Install

### All skills

```bash
npx skills add chipi-pay/skills
```

### Individual skill

```bash
npx skills add chipi-pay/skills --skill chipi-wallet-setup
```

### Claude Code plugin

```bash
/plugin marketplace add chipi-pay/skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| **chipi-wallet-setup** | End-to-end wallet integration — providers, auth, wallet creation with passkey, balance, USDC transfers |
| **chipi-payment-flow** | Accept USDC payments — Chipi wallet + external wallet checkout with webhook verification |
| **chipi-sku-marketplace** | Service marketplace — airtime, gift cards, bill pay, gaming credits (Mexico only) |
| **chipi-defi-staking** | VESU USDC staking — deposit and withdraw to earn yield on StarkNet |
| **chipi-session-keys** | Session keys — one auth, many transactions. Requires CHIPI wallet type |
| **chipi-custom-contracts** | Call any StarkNet contract through Chipi's gasless infra |
| **chipi-starknet-guide** | StarkNet reference — addresses, tokens, errors, troubleshooting |

## MCP Server

For real-time component access and SDK reference, connect the Chipi MCP server:

**Claude Code:**
```bash
claude mcp add --transport http chipiMcp https://mcp.chipipay.com/mcp
```

**Cursor:**
```json
{
  "mcpServers": {
    "chipiMcp": {
      "command": "npx mcp-remote https://mcp.chipipay.com/mcp"
    }
  }
}
```

The MCP server provides 9 tools, 9 resources, and 4 prompts for comprehensive Chipi development support.

## Key Defaults

- **Wallet type:** CHIPI (Chipi's own account contract — supports session keys)
- **Authentication:** Passkey/biometric (stronger than PIN, better UX)
- **Chain:** StarkNet
- **Tokens:** USDC (6 decimals), ETH (18), STRK (18)

## Links

- [Chipi Dashboard](https://dashboard.chipipay.com) — Get API keys
- [Chipi Docs](https://docs.chipipay.com) — Full documentation
- [MCP Server](https://mcp.chipipay.com) — Developer tools
- [Agent Skills Standard](https://agentskills.io) — Skills format specification

## License

MIT
