# Calldata Debugging Guide

How to encode, debug, and verify calldata for StarkNet contract calls via Chipi.

## u256 Encoding Walkthrough

StarkNet's `u256` type is stored as two `felt252` values: `low` (lower 128 bits) and `high` (upper 128 bits).

### Simple Case (Most Common)

For values under 2^128 (which covers virtually all practical token amounts):

```tsx
// The value fits entirely in "low", "high" is "0"
const calldata = [amount.toString(), "0"];
```

### Examples

| Human Amount | Token | Raw Amount | Calldata [low, high] |
|-------------|-------|-----------|---------------------|
| 10 USDC | USDC (6 dec) | 10,000,000 | `["10000000", "0"]` |
| 100 USDC | USDC (6 dec) | 100,000,000 | `["100000000", "0"]` |
| 0.5 ETH | ETH (18 dec) | 500,000,000,000,000,000 | `["500000000000000000", "0"]` |
| 1 ETH | ETH (18 dec) | 1,000,000,000,000,000,000 | `["1000000000000000000", "0"]` |
| 1000 STRK | STRK (18 dec) | 1,000,000,000,000,000,000,000 | `["1000000000000000000000", "0"]` |

### When high ≠ 0

Only needed for astronomical values (>340 undecillion). You'll almost never need this:

```tsx
// For truly massive values:
const MAX_U128 = BigInt(2) ** BigInt(128);
const value = BigInt(rawAmount);
const low = (value % MAX_U128).toString();
const high = (value / MAX_U128).toString();
const calldata = [low, high];
```

### Important: useTransfer vs useCallAnyContract

| Hook | Amount Format |
|------|--------------|
| `useTransfer` | Human-readable: `amount: 10` (SDK handles decimals) |
| `useCallAnyContract` | Raw calldata: `["10000000", "0"]` for 10 USDC |

This is the #1 source of encoding confusion. `useTransfer` handles conversion for you. `useCallAnyContract` passes raw calldata to the contract.

## Common Encoding Mistakes

### 1. Single value instead of [low, high]

```tsx
// WRONG:
calldata: [recipientAddress, "10000000"]
// Error: calldata length mismatch — contract expects u256 (2 felts)

// CORRECT:
calldata: [recipientAddress, "10000000", "0"]
// recipient + amount_low + amount_high
```

### 2. Wrong argument order

```tsx
// Contract ABI: transfer(recipient: ContractAddress, amount: u256)

// WRONG (amount before recipient):
calldata: ["10000000", "0", recipientAddress]

// CORRECT (match ABI order):
calldata: [recipientAddress, "10000000", "0"]
```

### 3. JavaScript number precision loss

```tsx
// WRONG (loses precision for large numbers):
calldata: [1000000000000000000]  // JS number can't represent this exactly

// CORRECT (use strings):
calldata: ["1000000000000000000"]
```

### 4. Human-readable amount in calldata

```tsx
// WRONG (passing 10 USDC as "10"):
calldata: [recipientAddress, "10", "0"]
// This sends 0.00001 USDC (10 raw units with 6 decimals)

// CORRECT (10 USDC = 10 * 10^6):
calldata: [recipientAddress, "10000000", "0"]
```

### 5. Missing array length for array parameters

```tsx
// Cairo function: fn process(values: Array<felt252>)

// WRONG:
calldata: ["100", "200", "300"]

// CORRECT (prepend length):
calldata: ["3", "100", "200", "300"]
```

## Block Explorer Verification

After a transaction fails, check the details:

### Starkscan
```
https://starkscan.co/tx/{txHash}
```
Look for:
- **Status:** REJECTED, REVERTED, or SUCCEEDED
- **Execution Resources:** Shows compute usage
- **Events:** Emitted events (if any)
- **Input Data:** The actual calldata sent

### Voyager
```
https://voyager.online/tx/{txHash}
```
Look for:
- **Execution Status:** Shows revert reason
- **Function Calls:** Trace of internal calls
- **Calldata:** Decoded if ABI is available

## Type Mapping: Solidity → Cairo → Calldata

| Solidity Type | Cairo Type | Calldata Encoding | Example |
|---------------|-----------|-------------------|---------|
| `uint256` | `u256` | 2 felts: [low, high] | `["1000000", "0"]` |
| `uint128` | `u128` | 1 felt | `["1000000"]` |
| `uint64` | `u64` | 1 felt | `["1000000"]` |
| `uint32` | `u32` | 1 felt | `["42"]` |
| `uint8` | `u8` | 1 felt | `["255"]` |
| `int256` | `i256` | 2 felts (signed) | Similar to u256 |
| `address` | `ContractAddress` | 1 felt (hex) | `["0x049d..."]` |
| `bool` | `bool` | 1 felt | `["0"]` or `["1"]` |
| `string` | `ByteArray` | Complex encoding | Use SDK helper |
| `bytes32` | `felt252` | 1 felt | `["0xabc..."]` |
| `bytes` | `Array<felt252>` | [length, ...values] | `["2", "0xab", "0xcd"]` |
| `address[]` | `Array<ContractAddress>` | [length, ...addrs] | `["2", "0x...", "0x..."]` |

## Debugging Checklist for Failed Contract Calls

1. **Check contract address** — Is it deployed? Check on Starkscan.
2. **Check entrypoint name** — Exact match? Cairo uses snake_case.
3. **Check calldata length** — Count: each u256 = 2 felts. Arrays need length prefix.
4. **Check calldata types** — All values should be strings. No JS numbers for large values.
5. **Check argument order** — Must match the ABI exactly.
6. **Check u256 encoding** — [low, high], not single value.
7. **Check approval** — Does the function spend tokens? Batch approve + call.
8. **Check permissions** — Does the contract restrict callers?
9. **Check the tx on explorer** — Starkscan/Voyager show the revert reason.
10. **Compare with a working call** — If another user's call works, diff the calldata.
