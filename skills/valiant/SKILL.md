# Valiant Skill

## Overview

Valiant is the primary DEX on Fogo. Vortex pools are concentrated liquidity AMMs. This skill enables creating/managing LP positions, swapping tokens, and collecting fees.

## Prerequisites

```bash
npm install @valiant-trade/vortex @valiant-trade/vortex-client @valiant-trade/vortex-core @solana/kit
```

**CRITICAL:**
- Uses `@solana/kit` v2.x (NOT `@solana/web3.js` v1.x)
- Always call `setVortexConfig('fogoMainnet')` before any SDK operations

## Configuration

```typescript
import { setVortexConfig } from "@valiant-trade/vortex";
import { createSolanaRpc, address } from "@solana/kit";

// Initialize SDK - REQUIRED before any operations
await setVortexConfig("fogoMainnet"); // or "fogoTestnet"

const rpc = createSolanaRpc("https://mainnet.fogo.io/");
```

## Token Addresses

| Token | Symbol | Mint Address | Decimals |
|-------|--------|--------------|----------|
| FOGO | FOGO | `So11111111111111111111111111111111111111112` | 9 |
| USDC | USDC | `uSd2czE61Evaf76RNbq4KPpXnkiL3irdzgLFUMe3NoG` | 6 |
| stFOGO | stFOGO | `Brasa3xzkSC9XqMBEcN9v53x4oMkpb1nQwfaGMyJE88b` | 9 |
| iFOGO | iFOGO | `iFoGoY5nMWpuMJogR7xjUAWDJtygHDF17zREeP4MKuD` | 9 |
| wSOL | wSOL | `HLc5qava5deWKeoVkN9nsu9RrqqD8PoHaGsvQzPFNbxR` | 9 |
| FISH | FISH | `F1SHsk3rbKUJp28MQyyqtmfoJqnrkVUuZYK5ymGW4ZAr` | 9 |
| FISH Classic | FISH | `F1SHuJ3sFF2wJoYbUJxK4iZ6CYg6MakFj8q6QHACFd4s` | 9 |
| CHASE | CHASE | `GPK7grvKT8kQPMYsgAN8N537XUNdJLs3WsnXkUNfpump` | 6 |

## Pool Addresses

| Pool | Address | Token A | Token B | Fee | Tick Spacing |
|------|---------|---------|---------|-----|--------------|
| FOGO-USDC | `J7mxBLSz51Tcbog3XsiJTAXS64N46KqbpRGQmd3dQMKp` | FOGO | USDC | 0.30% | 64 |
| FOGO-stFOGO | `Be2eoA9g1Yp8WKqMM14tXjSHuYCudaPpaudLTmC4gizp` | FOGO | stFOGO | 0.30% | 64 |
| FOGO-iFOGO | `HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E` | FOGO | iFOGO | 0.05% | 8 |
| USDC-FISH | `DjM47hJzwQmsXwRRhsEWsjRVt4vXfEmZTDAP1zhM6XKF` | USDC | FISH | 1.00% | 128 |
| USDC-wSOL | `2exTq4dyaUa1mwXytfMSZ9r5BAcZ98L2zawBQDMfaU9o` | USDC | wSOL | 0.30% | 64 |
| FOGO-CHASE | `2zKEnSqCVwPUgR6UkNDr6U5PYGpfwXFrV1pT9LRxQCtk` | FOGO | CHASE | 1.00% | 128 |
| FOGO-FISH | `HEv4767Y7NwPm367bHuunuhcjbsgVpsybT7etovqc9KR` | FOGO | FISH | 1.00% | 128 |

### Pool Details

**FOGO-USDC** - Main Trading Pool
- Address: `J7mxBLSz51Tcbog3XsiJTAXS64N46KqbpRGQmd3dQMKp`
- Use: Primary FOGO/USD price discovery and trading
- Fee: 0.30% (3000)
- Decimals: 9/6

**FOGO-stFOGO** - Liquid Staking Stable Pool
- Address: `Be2eoA9g1Yp8WKqMM14tXjSHuYCudaPpaudLTmC4gizp`
- Use: stFOGO redemption, arbitrage, tight-range LP
- Fee: 0.30% (3000)
- Note: stFOGO appreciates vs FOGO (~0.73 ratio)

**FOGO-iFOGO** - Pegged Asset Pool
- Address: `HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E`
- Use: iFOGO/FOGO swaps (pegged ~1:1)
- Fee: 0.05% (500) - Low fee for stable pair
- Tick Spacing: 8 (very tight ranges possible)

## Common Patterns

### Pattern 1: Fetch Pool Data

```typescript
import { fetchVortex } from "@valiant-trade/vortex-client";
import { sqrtPriceToPrice } from "@valiant-trade/vortex-core";

const FOGO_USDC_POOL = "J7mxBLSz51Tcbog3XsiJTAXS64N46KqbpRGQmd3dQMKp";

const pool = await fetchVortex(rpc, address(FOGO_USDC_POOL));

// Convert sqrt price to human-readable price
const price = sqrtPriceToPrice(
  pool.data.sqrtPrice,
  9,  // Token A decimals (FOGO)
  6   // Token B decimals (USDC)
);

console.log("FOGO Price:", price.toFixed(6));
console.log("Liquidity:", pool.data.liquidity);
console.log("Current Tick:", pool.data.tickCurrentIndex);
```

### Pattern 2: Open LP Position

```typescript
import { openPositionInstructions } from "@valiant-trade/vortex";

const sessionSigner = { address: wallet.address, signer: wallet };

const vortexBaseInfo = {
  tokenMintA: pool.data.tokenMintA,
  tokenMintB: pool.data.tokenMintB,
  tokenVaultA: pool.data.tokenVaultA,
  tokenVaultB: pool.data.tokenVaultB,
  sqrtPrice: pool.data.sqrtPrice,
  tickSpacing: pool.data.tickSpacing,
};

const result = await openPositionInstructions(
  rpc,
  address(poolAddress),
  vortexBaseInfo,
  null,  // mintA - fetched internally
  null,  // mintB - fetched internally
  { tokenB: usdcAmount },  // or { tokenA: fogoAmount }
  lowerPrice,   // e.g., currentPrice * 0.95
  upperPrice,   // e.g., currentPrice * 1.05
  sessionSigner,
  false,  // signerIsSession
  {},     // cachedTokenMintToProgramAddress
  null,   // cachedMintToDecimals
  [],     // existingTickArrayAddresses
  wallet, // tickArrayFunder
  100,    // slippageBps (1%)
  wallet  // funder
);

console.log("Position Mint:", result.positionMint);
console.log("Quote:", result.quote);

// Execute transaction
const sig = await executeInstructions(rpc, result.instructions, wallet);
```

### Pattern 3: Close Position (Withdraw + Collect Fees)

```typescript
import { closePositionInstructions } from "@valiant-trade/vortex";

const result = await closePositionInstructions(
  rpc,
  address(positionMint),
  sessionSigner,
  100,    // slippageBps
  wallet, // funder
  false   // doClose - set FALSE due to SDK v1.0.3 bug
);

// Returns withdrawn tokens and collected fees
console.log("Token A:", result.quote.tokenEstA);
console.log("Token B:", result.quote.tokenEstB);

const sig = await executeInstructions(rpc, result.instructions, wallet);
```

### Pattern 4: Increase Liquidity (Compound)

```typescript
import { increaseLiquidityInstructions } from "@valiant-trade/vortex";
import { fetchPosition } from "@valiant-trade/vortex-client";

const position = await fetchPosition(rpc, address(positionAddress));

const positionFacade = {
  liquidity: position.data.liquidity,
  tickLowerIndex: position.data.tickLowerIndex,
  tickUpperIndex: position.data.tickUpperIndex,
  feeGrowthCheckpointA: position.data.feeGrowthCheckpointA,
  feeOwedA: position.data.feeOwedA,
  feeGrowthCheckpointB: position.data.feeGrowthCheckpointB,
  feeOwedB: position.data.feeOwedB,
  rewardInfos: position.data.rewardInfos.map(r => ({
    growthInsideCheckpoint: r.growthInsideCheckpoint,
    amountOwed: r.amountOwed,
  })),
};

const result = await increaseLiquidityInstructions(
  rpc,
  pool.data,
  address(poolAddress),
  address(positionAddress),
  address(positionMint),
  positionFacade,
  { tokenB: usdcAmount },
  sessionSigner,
  false,
  {},
  100, // slippageBps
  wallet
);
```

### Pattern 5: Harvest Fees (Without Closing)

```typescript
import { harvestPositionInstructions } from "@valiant-trade/vortex";

const result = await harvestPositionInstructions(
  rpc,
  address(positionMint),
  sessionSigner,
  wallet // funder
);

const sig = await executeInstructions(rpc, result.instructions, wallet);
```

### Pattern 6: Swap Tokens

```typescript
import {
  swapInstructionsWithAmounts,
  prepareTickArrayAddressesForSwap,
} from "@valiant-trade/vortex";

// Prepare tick arrays for swap
const tickArrayAddresses = await prepareTickArrayAddressesForSwap(
  pool.data.tickSpacing,
  pool.data.tickCurrentIndex,
  address(poolAddress)
);

// Token programs (SPL Token for both)
const tokenProgramAddresses = [
  address("TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"),
  address("TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"),
];

const result = await swapInstructionsWithAmounts(
  rpc,
  {
    inputAmount: inputAmount,
    outputAmount: minOutputAmount,
    aForB: false,  // false = B→A (USDC→FOGO), true = A→B (FOGO→USDC)
    isExactIn: true,
  },
  pool.data,
  address(poolAddress),
  tickArrayAddresses,
  tokenProgramAddresses,
  sessionSigner,
  false,
  wallet,
  false
);

const sig = await executeInstructions(rpc, result.instructions, wallet);
```

### Pattern 7: Get User Positions

```typescript
import { fetchPositionsForOwner } from "@valiant-trade/vortex";

const positions = await fetchPositionsForOwner(rpc, wallet.address);

for (const pos of positions) {
  if (pos.isPositionBundle) {
    for (const bundledPos of pos.positions) {
      console.log("Position:", bundledPos.data.positionMint);
      console.log("Pool:", bundledPos.data.vortex);
      console.log("Liquidity:", bundledPos.data.liquidity);
    }
  } else {
    console.log("Position:", pos.data.positionMint);
    console.log("Pool:", pos.data.vortex);
    console.log("Liquidity:", pos.data.liquidity);
  }
}
```

### Pattern 8: Check If Position In Range

```typescript
import { isPositionInRange } from "@valiant-trade/vortex-core";

const inRange = isPositionInRange(
  pool.data.sqrtPrice,
  position.data.tickLowerIndex,
  position.data.tickUpperIndex
);

console.log("Earning fees:", inRange);
```

## LP Strategies

### Sell Ladder (FOGO → USDC on price rise)

```typescript
// Place positions ABOVE current price
// When price rises INTO these ranges, FOGO converts to USDC
const sellLadders = [
  { lower: price * 1.005, upper: price * 1.015, pct: 20 },
  { lower: price * 1.015, upper: price * 1.03,  pct: 20 },
  { lower: price * 1.03,  upper: price * 1.05,  pct: 20 },
  { lower: price * 1.05,  upper: price * 1.08,  pct: 20 },
  { lower: price * 1.08,  upper: price * 1.12,  pct: 20 },
];

for (const ladder of sellLadders) {
  const fogoAmount = BigInt(Math.floor(totalFogo * ladder.pct / 100));
  await openPosition(rpc, pool, ladder.lower, ladder.upper, { tokenA: fogoAmount }, wallet);
}
```

### Buy Ladder (Accumulate FOGO on price drop)

```typescript
// Place positions BELOW current price with USDC
// When price drops INTO these ranges, USDC converts to FOGO
const buyLadders = [
  { lower: price * 0.95,  upper: price * 0.985, pct: 25 },
  { lower: price * 0.90,  upper: price * 0.95,  pct: 25 },
  { lower: price * 0.85,  upper: price * 0.90,  pct: 25 },
  { lower: price * 0.80,  upper: price * 0.85,  pct: 25 },
];
```

### Tight Range for Stable Pairs

```typescript
// For FOGO-stFOGO or FOGO-iFOGO stable pools
// Use very tight ranges for maximum fee capture
const stablePrice = 0.99; // ~1:1 peg
const lowerPrice = stablePrice * 0.995; // -0.5%
const upperPrice = stablePrice * 1.005; // +0.5%
```

## Error Handling

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Zero quote returned` | Position out of range or SDK limitation | Use in-range positions |
| `insufficient funds` | Not enough tokens | Check balance, reduce amount |
| `custom program error: 0x1` | Token transfer failed | Check token balances |
| `Tick not initializable` | Tick not aligned to spacing | Use tick multiples of tickSpacing |

### SDK v1.0.3 Known Issues

1. **closePositionInstructions with doClose=true fails** - Use `doClose: false` to withdraw only
2. **Single-sided out-of-range positions return zero quote** - Create in-range positions instead

## Constants Template

```typescript
// constants.ts
export const TOKENS = {
  FOGO: "So11111111111111111111111111111111111111112",
  USDC: "uSd2czE61Evaf76RNbq4KPpXnkiL3irdzgLFUMe3NoG",
  stFOGO: "Brasa3xzkSC9XqMBEcN9v53x4oMkpb1nQwfaGMyJE88b",
  iFOGO: "iFoGoY5nMWpuMJogR7xjUAWDJtygHDF17zREeP4MKuD",
} as const;

export const DECIMALS = {
  FOGO: 9,
  USDC: 6,
  stFOGO: 9,
  iFOGO: 9,
} as const;

export const POOLS = {
  FOGO_USDC: "J7mxBLSz51Tcbog3XsiJTAXS64N46KqbpRGQmd3dQMKp",
  FOGO_STFOGO: "Be2eoA9g1Yp8WKqMM14tXjSHuYCudaPpaudLTmC4gizp",
  FOGO_IFOGO: "HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E",
  USDC_FISH: "DjM47hJzwQmsXwRRhsEWsjRVt4vXfEmZTDAP1zhM6XKF",
  FOGO_FISH: "HEv4767Y7NwPm367bHuunuhcjbsgVpsybT7etovqc9KR",
} as const;
```

## References

- Valiant App: https://valiant.trade
- Valiant Pools: https://valiant.trade/pools
- Valiant Docs: https://docs.valiant.trade/
- Vortex NPM: https://www.npmjs.com/package/@valiant-trade/vortex
- Vortex Core NPM: https://www.npmjs.com/package/@valiant-trade/vortex-core
- Related: [fogo-core](../fogo-core/SKILL.md), [brasa](../brasa/SKILL.md)
