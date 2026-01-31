# Data Providers Skill

## Overview

Fetch price feeds and market data for Fogo tokens from multiple sources: Codex, Birdeye, DexScreener, CoinGecko, and CoinMarketCap.

## Codex (Fogo Integrated)

Real-time enriched blockchain data API with native Fogo support.

### Installation

```bash
npm install @codex-data/sdk
```

### Usage

```typescript
import { Codex } from "@codex-data/sdk";

const sdk = new Codex(process.env.CODEX_API_KEY);

// Get Fogo network ID
const networks = await sdk.send(`
  query GetNetworks {
    getNetworks { id name }
  }
`);
const fogoNetworkId = networks.getNetworks.find(n => n.name === "Fogo")?.id;

// Get token data
const token = await sdk.queries.token({
  input: {
    address: "So11111111111111111111111111111111111111112", // FOGO
    networkId: fogoNetworkId
  }
});

// Subscribe to price updates (WebSocket)
sdk.subscriptions.onPriceUpdated(
  { address: "TOKEN_ADDRESS", networkId: fogoNetworkId },
  (result) => {
    console.log("Price:", result.data.onPriceUpdated);
  }
);
```

### Features
- Real-time USD pricing
- OHLCV charts
- Token holder data
- Volume/liquidity aggregates
- WebSocket subscriptions

**Docs:** https://docs.codex.io/

---

## Birdeye (Fogo Integrated)

On-chain crypto trading data aggregator.

### API Setup

```typescript
const BIRDEYE_API = "https://public-api.birdeye.so";
const headers = {
  "X-API-KEY": process.env.BIRDEYE_API_KEY,
  "x-chain": "fogo"
};
```

### Endpoints

```typescript
// Token price
async function getPrice(tokenAddress: string) {
  const res = await fetch(
    `${BIRDEYE_API}/defi/price?address=${tokenAddress}`,
    { headers }
  );
  return res.json();
}

// Price history
async function getPriceHistory(tokenAddress: string, type = "15m") {
  const timeFrom = Math.floor(Date.now() / 1000) - 86400; // 24h ago
  const timeTo = Math.floor(Date.now() / 1000);
  const res = await fetch(
    `${BIRDEYE_API}/defi/history_price?address=${tokenAddress}&type=${type}&time_from=${timeFrom}&time_to=${timeTo}`,
    { headers }
  );
  return res.json();
}

// Multiple token prices
async function getMultiPrice(addresses: string[]) {
  const res = await fetch(
    `${BIRDEYE_API}/defi/multi_price?list_address=${addresses.join(",")}`,
    { headers }
  );
  return res.json();
}

// Token overview (volume, liquidity, etc.)
async function getTokenOverview(tokenAddress: string) {
  const res = await fetch(
    `${BIRDEYE_API}/defi/token_overview?address=${tokenAddress}`,
    { headers }
  );
  return res.json();
}

// OHLCV data
async function getOHLCV(tokenAddress: string, type = "15m") {
  const res = await fetch(
    `${BIRDEYE_API}/defi/ohlcv?address=${tokenAddress}&type=${type}`,
    { headers }
  );
  return res.json();
}
```

**Docs:** https://docs.birdeye.so/

---

## DexScreener (Fogo Added)

Real-time DEX analytics.

### API (No Auth Required)

```typescript
const DEXSCREENER_API = "https://api.dexscreener.com";

// Get pair by address
async function getPair(pairAddress: string) {
  const res = await fetch(
    `${DEXSCREENER_API}/latest/dex/pairs/fogo/${pairAddress}`
  );
  return res.json();
}

// Search pairs
async function searchPairs(query: string) {
  const res = await fetch(
    `${DEXSCREENER_API}/latest/dex/search?q=${query}`
  );
  return res.json();
}

// Get token pairs
async function getTokenPairs(tokenAddress: string) {
  const res = await fetch(
    `${DEXSCREENER_API}/latest/dex/tokens/${tokenAddress}`
  );
  return res.json();
}

// Get pools for token on Fogo
async function getTokenPools(tokenAddress: string) {
  const res = await fetch(
    `${DEXSCREENER_API}/token-pools/v1/fogo/${tokenAddress}`
  );
  return res.json();
}
```

### Response Structure

```typescript
interface DexScreenerPair {
  chainId: string;      // "fogo"
  dexId: string;        // "valiant"
  pairAddress: string;
  baseToken: { address: string; name: string; symbol: string };
  quoteToken: { address: string; name: string; symbol: string };
  priceNative: string;
  priceUsd: string;
  liquidity: { usd: number; base: number; quote: number };
  volume: { h24: number; h6: number; h1: number; m5: number };
  priceChange: { h24: number; h6: number; h1: number; m5: number };
}
```

**Rate Limit:** 300 requests/minute

**Fogo on DexScreener:** https://dexscreener.com/fogo

---

## CoinGecko (FOGO Listed)

### API

```typescript
const COINGECKO_API = "https://api.coingecko.com/api/v3";

// Simple price
async function getFogoPrice() {
  const res = await fetch(
    `${COINGECKO_API}/simple/price?ids=fogo&vs_currencies=usd`
  );
  return res.json();
}

// Detailed data
async function getFogoData() {
  const res = await fetch(`${COINGECKO_API}/coins/fogo`);
  return res.json();
}

// Market chart (7 days)
async function getFogoChart(days = 7) {
  const res = await fetch(
    `${COINGECKO_API}/coins/fogo/market_chart?vs_currency=usd&days=${days}`
  );
  return res.json();
}

// OHLC
async function getFogoOHLC(days = 7) {
  const res = await fetch(
    `${COINGECKO_API}/coins/fogo/ohlc?vs_currency=usd&days=${days}`
  );
  return res.json();
}
```

**Rate Limits:**
- Demo (free): 30 calls/min, 10K calls/month
- Paid: 500+ calls/min

**FOGO on CoinGecko:** https://www.coingecko.com/en/coins/fogo

---

## CoinMarketCap (FOGO Listed)

### API

```typescript
const CMC_API = "https://pro-api.coinmarketcap.com/v1";
const headers = {
  "X-CMC_PRO_API_KEY": process.env.CMC_API_KEY
};

// Latest quotes
async function getFogoQuotes() {
  const res = await fetch(
    `${CMC_API}/cryptocurrency/quotes/latest?symbol=FOGO`,
    { headers }
  );
  return res.json();
}

// Metadata
async function getFogoMetadata() {
  const res = await fetch(
    `${CMC_API}/cryptocurrency/info?symbol=FOGO`,
    { headers }
  );
  return res.json();
}
```

**FOGO on CMC:** https://coinmarketcap.com/currencies/fogo/

---

## Unified Price Fetcher

```typescript
interface PriceData {
  price: number;
  volume24h?: number;
  priceChange24h?: number;
  liquidity?: number;
  source: "codex" | "birdeye" | "dexscreener" | "coingecko" | "coinmarketcap" | "onchain";
  timestamp: number;
}

// Fetch from multiple sources for reliability
async function getTokenPrice(tokenAddress: string): Promise<PriceData[]> {
  const sources = await Promise.allSettled([
    fetchFromBirdeye(tokenAddress),
    fetchFromDexScreener(tokenAddress),
    fetchFromOnChain(tokenAddress), // Direct from Valiant pool
  ]);

  return sources
    .filter((s): s is PromiseFulfilledResult<PriceData> => s.status === "fulfilled")
    .map(s => s.value);
}

// Get FOGO price directly from Valiant pool (most accurate)
async function fetchFromOnChain(tokenAddress: string): Promise<PriceData> {
  const pool = await fetchVortex(rpc, address("J7mxBLSz51Tcbog3XsiJTAXS64N46KqbpRGQmd3dQMKp"));
  const price = sqrtPriceToPrice(pool.data.sqrtPrice, 9, 6);

  return {
    price,
    source: "onchain",
    timestamp: Date.now(),
  };
}
```

## References

- Codex: https://docs.codex.io/
- Birdeye: https://docs.birdeye.so/
- DexScreener: https://docs.dexscreener.com/api/reference
- CoinGecko: https://docs.coingecko.com/
- CoinMarketCap: https://coinmarketcap.com/api/documentation/v1/
- Related: [valiant](../valiant/SKILL.md)
