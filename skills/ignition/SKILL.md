# Ignition Skill

## Overview

Ignition is a liquid staking protocol on Fogo - stake FOGO and receive iFOGO tokens that remain liquid for DeFi use while earning staking rewards.

## Key Concepts

- **Deposit FOGO** → receive iFOGO
- **iFOGO is pegged ~1:1** to FOGO (unlike stFOGO which appreciates)
- **Use iFOGO** in DeFi: trading, lending, LP
- **Redeem iFOGO** → get back FOGO (or swap on Valiant)

## Token Addresses

| Token | Mint Address | Decimals |
|-------|--------------|----------|
| iFOGO | `iFoGoY5nMWpuMJogR7xjUAWDJtygHDF17zREeP4MKuD` | 9 |
| FOGO | `So11111111111111111111111111111111111111112` | 9 |

## Program Address

| Program | Address |
|---------|---------|
| Ignition Stake Pool | `SP1s4uFeTAX9jsXXmwyDs1gxYYf7cdDZ8qHUHVxE1yr` |

## iFOGO vs stFOGO

| Token | Protocol | Peg | Mechanism |
|-------|----------|-----|-----------|
| iFOGO | Ignition | ~1:1 with FOGO | Rewards distributed separately |
| stFOGO | Brasa | Appreciates vs FOGO | Rewards accrue in token value |

## How to Stake (Web Interface)

1. Go to https://ignitionfi.xyz
2. Connect your wallet
3. Enter FOGO amount to stake
4. Click "Stake"
5. Receive iFOGO tokens

## How to Unstake

**Option 1: Ignition Web App**
1. Go to https://ignitionfi.xyz
2. Select unstake option
3. Enter iFOGO amount
4. Receive FOGO

**Option 2: Swap on Valiant**
```typescript
// Swap iFOGO → FOGO via Valiant pool
// Faster, may have minor slippage
const FOGO_IFOGO_POOL = "HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E";

// Use swap pattern from valiant skill
// aForB = true (iFOGO → FOGO)
```

## DeFi Strategies with iFOGO

### 1. LP in FOGO-iFOGO Pool

Very tight range LP for fee capture on a stable pair:

```typescript
// Extremely tight range since it's a 1:1 pegged pair
const currentRate = 1.0; // iFOGO/FOGO ~1:1
const lowerPrice = 0.995;
const upperPrice = 1.005;

// Low fee pool (0.05%) with tight tick spacing (8)
const FOGO_IFOGO_POOL = "HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E";

// Open position with iFOGO
await openPosition(rpc, FOGO_IFOGO_POOL, lowerPrice, upperPrice, { tokenB: ifogoAmount }, wallet);
```

### 2. Arbitrage

When iFOGO pool rate deviates from 1:1 peg:
- If pool rate < 1: Buy iFOGO from pool, redeem on Ignition
- If pool rate > 1: Stake on Ignition, sell iFOGO on pool

### 3. Use as Collateral

Deposit iFOGO as collateral on lending protocols (if supported).

## Checking iFOGO Balance

```typescript
const IFOGO_MINT = "iFoGoY5nMWpuMJogR7xjUAWDJtygHDF17zREeP4MKuD";

const tokenAccounts = await rpc.getTokenAccountsByOwner(
  wallet.address,
  { mint: address(IFOGO_MINT) },
  { encoding: "jsonParsed" }
).send();

const ifogoBalance = tokenAccounts.value.length > 0
  ? BigInt(tokenAccounts.value[0].account.data.parsed.info.tokenAmount.amount)
  : 0n;

console.log("iFOGO Balance:", Number(ifogoBalance) / 1e9);
```

## Pool Details

**FOGO-iFOGO Pool**
- Address: `HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E`
- Fee: 0.05% (500) - Low fee for stable pair
- Tick Spacing: 8 (very tight ranges possible)
- Use: iFOGO/FOGO swaps, tight-range LP

## References

- Ignition App: https://ignitionfi.xyz
- Ignition Docs: https://docs.ignitionfi.xyz/
- iFOGO Token: `iFoGoY5nMWpuMJogR7xjUAWDJtygHDF17zREeP4MKuD`
- FOGO-iFOGO Pool: `HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E`
- Related: [valiant](../valiant/SKILL.md), [brasa](../brasa/SKILL.md), [fogo-core](../fogo-core/SKILL.md)
