# Brasa Skill

## Overview

Brasa enables liquid staking on Fogo - stake FOGO and receive stFOGO tokens that accrue staking rewards while remaining liquid for DeFi use.

## Key Concepts

- **Deposit FOGO** → receive stFOGO
- **stFOGO appreciates** against FOGO as staking rewards accrue
- **Use stFOGO** in DeFi: trading, lending, LP
- **Redeem stFOGO** → get back FOGO (or swap on Valiant)

## Token Addresses

| Token | Mint Address | Decimals |
|-------|--------------|----------|
| stFOGO | `Brasa3xzkSC9XqMBEcN9v53x4oMkpb1nQwfaGMyJE88b` | 9 |
| FOGO | `So11111111111111111111111111111111111111112` | 9 |

## stFOGO Exchange Rate

stFOGO appreciates against FOGO over time as staking rewards accumulate:

```typescript
// Current ratio (example)
// 1 stFOGO ≈ 0.73 FOGO (stFOGO is worth more)
// This means: 1 FOGO → ~1.37 stFOGO when staking

// Check current rate via FOGO-stFOGO pool
import { fetchVortex } from "@valiant-trade/vortex-client";
import { sqrtPriceToPrice } from "@valiant-trade/vortex-core";

const FOGO_STFOGO_POOL = "Be2eoA9g1Yp8WKqMM14tXjSHuYCudaPpaudLTmC4gizp";
const pool = await fetchVortex(rpc, address(FOGO_STFOGO_POOL));
const rate = sqrtPriceToPrice(pool.data.sqrtPrice, 9, 9);
console.log("stFOGO/FOGO rate:", rate); // ~0.73
```

## How to Stake (Web Interface)

**No public SDK yet** - interact via web app:

1. Go to https://brasa.finance
2. Connect your wallet
3. Enter FOGO amount to stake
4. Click "Stake"
5. Receive stFOGO tokens

## How to Unstake

**Option 1: Brasa Web App**
1. Go to https://brasa.finance
2. Select "Unstake" tab
3. Enter stFOGO amount
4. Receive FOGO (may have cooldown period)

**Option 2: Swap on Valiant**
```typescript
// Swap stFOGO → FOGO via Valiant pool
// Faster but may have slippage
const FOGO_STFOGO_POOL = "Be2eoA9g1Yp8WKqMM14tXjSHuYCudaPpaudLTmC4gizp";

// Use swap pattern from valiant skill
// aForB = true (stFOGO → FOGO)
```

## DeFi Strategies with stFOGO

### 1. LP in stFOGO-FOGO Pool

Tight-range LP for fee capture on a stable pair:

```typescript
// Very tight range since it's a stable pair
const currentRate = 0.73; // stFOGO/FOGO
const lowerPrice = currentRate * 0.995;
const upperPrice = currentRate * 1.005;

// Open position with stFOGO
await openPosition(rpc, FOGO_STFOGO_POOL, lowerPrice, upperPrice, { tokenB: stfogoAmount }, wallet);
```

### 2. Use as Collateral on Pyron

Deposit stFOGO as collateral for borrowing (if supported).

### 3. Arbitrage

When stFOGO pool rate differs from Brasa staking rate:
- If pool rate < Brasa rate: Buy stFOGO from pool, unstake on Brasa
- If pool rate > Brasa rate: Stake on Brasa, sell stFOGO on pool

## Checking stFOGO Balance

```typescript
const STFOGO_MINT = "Brasa3xzkSC9XqMBEcN9v53x4oMkpb1nQwfaGMyJE88b";

const tokenAccounts = await rpc.getTokenAccountsByOwner(
  wallet.address,
  { mint: address(STFOGO_MINT) },
  { encoding: "jsonParsed" }
).send();

const stfogoBalance = tokenAccounts.value.length > 0
  ? BigInt(tokenAccounts.value[0].account.data.parsed.info.tokenAmount.amount)
  : 0n;

console.log("stFOGO Balance:", Number(stfogoBalance) / 1e9);
```

## Yield Calculation

```typescript
// Approximate APY from staking rewards
// stFOGO appreciates ~X% per epoch

// Check historical rate change
const currentRate = 0.73;
const rateLastWeek = 0.735;
const weeklyAppreciation = (rateLastWeek - currentRate) / rateLastWeek;
const estimatedAPY = weeklyAppreciation * 52 * 100;
console.log("Estimated APY:", estimatedAPY.toFixed(2) + "%");
```

## References

- Brasa App: https://brasa.finance
- Brasa Docs: https://docs.brasa.finance/
- stFOGO Token: `Brasa3xzkSC9XqMBEcN9v53x4oMkpb1nQwfaGMyJE88b`
- FOGO-stFOGO Pool: `Be2eoA9g1Yp8WKqMM14tXjSHuYCudaPpaudLTmC4gizp`
- Related: [valiant](../valiant/SKILL.md), [fogo-core](../fogo-core/SKILL.md)
