# Fogo Agent Kit

Claude Code skills for building AI agents on the Fogo blockchain ecosystem.

## Overview

This kit provides modular skills for interacting with Fogo's DeFi protocols:

| Skill | Description |
|-------|-------------|
| [fogo-core](./skills/fogo-core/SKILL.md) | Core blockchain interactions - wallet, RPC, transactions |
| [valiant](./skills/valiant/SKILL.md) | Valiant DEX concentrated liquidity - LP, swap, fees |
| [pyron](./skills/pyron/SKILL.md) | Pyron lending protocol - deposit, borrow, repay |
| [brasa](./skills/brasa/SKILL.md) | Brasa liquid staking - FOGO to stFOGO |
| [ignition](./skills/ignition/SKILL.md) | Ignition liquid staking - FOGO to iFOGO |
| [fogolend](./skills/fogolend/SKILL.md) | Fogolend lending protocol - lend, borrow, earn yield |
| [data-providers](./skills/data-providers/SKILL.md) | Price feeds from Codex, Birdeye, DexScreener, etc. |

## Quick Start

```bash
# Install dependencies
npm install @valiant-trade/vortex @valiant-trade/vortex-client @valiant-trade/vortex-core @solana/kit dotenv

# Set environment
cp .env.example .env
# Edit .env with your RPC URL and wallet path
```

## Token Addresses

| Token | Mint | Decimals |
|-------|------|----------|
| FOGO | `So11111111111111111111111111111111111111112` | 9 |
| USDC | `uSd2czE61Evaf76RNbq4KPpXnkiL3irdzgLFUMe3NoG` | 6 |
| stFOGO | `Brasa3xzkSC9XqMBEcN9v53x4oMkpb1nQwfaGMyJE88b` | 9 |
| iFOGO | `iFoGoY5nMWpuMJogR7xjUAWDJtygHDF17zREeP4MKuD` | 9 |

## Pool Addresses

| Pool | Address | Fee |
|------|---------|-----|
| FOGO-USDC | `J7mxBLSz51Tcbog3XsiJTAXS64N46KqbpRGQmd3dQMKp` | 0.30% |
| FOGO-stFOGO | `Be2eoA9g1Yp8WKqMM14tXjSHuYCudaPpaudLTmC4gizp` | 0.30% |
| FOGO-iFOGO | `HULdR8aMSxJAiNJmrTBcfKN4Zq6FgG33AHbQ3nDD8P5E` | 0.05% |

## Ecosystem Links

- [Fogo](https://fogo.io) | [Docs](https://docs.fogo.io/)
- [Valiant](https://valiant.trade) | [Docs](https://docs.valiant.trade/)
- [Pyron](https://pyron.finance) | [Docs](https://docs.pyron.fi/)
- [Brasa](https://brasa.finance) | [Docs](https://docs.brasa.finance/)
- [Ignition](https://ignitionfi.xyz) | [Docs](https://docs.ignitionfi.xyz/)
- [Fogolend](https://fogolend.com) | [Docs](https://docs.fogolend.com/)
- [Explorer](https://explorer.fogo.io)

## License

MIT License - see [LICENSE](./LICENSE) for details.
