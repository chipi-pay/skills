# VESU Protocol Reference

## Overview

VESU is a lending protocol on StarkNet. Users deposit USDC to earn yield from lending.

## How It Works

1. User deposits USDC into VESU pool via Chipi
2. VESU lends deposited USDC to borrowers
3. Interest accrues automatically
4. User withdraws deposit + yield (subject to pool liquidity)

## Chipi Hooks

### useStakeVesuUsdc

Deposits USDC into VESU. Automatically handles:
1. ERC-20 approval (grants VESU permission to use USDC)
2. Deposit transaction

Parameters: `encryptKey`, `wallet`, `amount`, `bearerToken`

### useWithdrawVesuUsdc

Withdraws USDC from VESU. Same parameters as stake.

## Important Notes

- All transactions are gasless through Chipi
- USDC has 6 decimals
- Yield rate is variable — depends on pool utilization
- Withdrawals may be delayed if pool liquidity is low
- No minimum stake amount (but must be > 0)
- Non-custodial — Chipi never holds your funds
