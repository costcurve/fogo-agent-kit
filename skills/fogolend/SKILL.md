# Fogolend Skill

## Overview

Fogolend is a decentralized lending protocol on Fogo, enabling users to lend, borrow, and earn yield on crypto assets with competitive rates and instant transactions.

## Key Concepts

- **Lend assets** → earn interest from borrowers
- **Deposit collateral** → borrow other assets
- **Maintain health factor** → avoid liquidation
- **Instant transactions** → powered by Fogo's ~40ms blocks

## How It Works

### Lending (Supplying)
1. Deposit supported assets into lending pools
2. Receive interest-bearing tokens representing your position
3. Earn yield as borrowers pay interest
4. Withdraw anytime (subject to utilization)

### Borrowing
1. Deposit collateral (overcollateralized)
2. Borrow up to a percentage of collateral value
3. Pay variable interest rate
4. Repay loan + interest to unlock collateral

## Web Interface

**No public SDK yet** - interact via web app:

1. Go to https://fogolend.com
2. Connect your wallet
3. Select "Lend" or "Borrow"
4. Choose asset and amount
5. Confirm transaction

## Health Factor

Monitor your health factor to avoid liquidation:

```
Health Factor = (Collateral Value × Liquidation Threshold) / Borrow Value

- Health Factor > 1: Safe
- Health Factor ≤ 1: Liquidation risk
```

## DeFi Strategies

### 1. Yield Farming
Supply idle assets to earn lending yield.

### 2. Leveraged Long
1. Deposit FOGO as collateral
2. Borrow USDC
3. Swap USDC for more FOGO
4. Repeat (careful of liquidation risk)

### 3. Leveraged Short
1. Deposit USDC as collateral
2. Borrow FOGO
3. Swap FOGO for USDC
4. Repay when FOGO price drops

### 4. Yield Loop with Liquid Staking
1. Stake FOGO → receive stFOGO or iFOGO
2. Deposit stFOGO/iFOGO as collateral (if supported)
3. Borrow FOGO
4. Stake again
5. Earn staking + lending yield

## Risk Considerations

| Risk | Description | Mitigation |
|------|-------------|------------|
| Liquidation | Collateral seized if health factor drops | Maintain buffer, monitor prices |
| Smart contract | Protocol vulnerabilities | Check audits, start small |
| Utilization | Can't withdraw if pool fully utilized | Monitor utilization rates |
| Interest rate | Variable rates can spike | Monitor borrowing costs |

## Checking Balances

Use standard SPL token balance checks for any interest-bearing tokens received from Fogolend.

```typescript
// Check if you have Fogolend position tokens
const tokenAccounts = await rpc.getTokenAccountsByOwner(
  wallet.address,
  { programId: address("TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA") },
  { encoding: "jsonParsed" }
).send();

// Filter for Fogolend tokens (addresses TBD when SDK available)
```

## Integration Status

| Feature | Status |
|---------|--------|
| Web App | Available |
| SDK | In development |
| Documentation | In progress |

## References

- Fogolend App: https://fogolend.com
- Fogolend Docs: https://docs.fogolend.com/
- Discord: https://discord.gg/wVX4JN8zpx
- GitHub: https://github.com/fogolend-labs
- Related: [fogo-core](../fogo-core/SKILL.md), [brasa](../brasa/SKILL.md), [ignition](../ignition/SKILL.md)
