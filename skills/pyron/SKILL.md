# Pyron Skill

## Overview

Pyron is the asset productivity layer of Fogo - overcollateralized lending and borrowing protocol.

## Prerequisites

```bash
npm install @pyron-finance/pyron-client @solana/web3.js
```

**Note:** Pyron SDK uses `@solana/web3.js` v1.x (different from Vortex which uses `@solana/kit` v2).

## Configuration

```typescript
import { Connection, Keypair } from "@solana/web3.js";
import { LendrClient, getConfig } from "@pyron-finance/pyron-client";
import { NodeWallet } from "@coral-xyz/anchor";
import bs58 from "bs58";

const connection = new Connection("https://mainnet.fogo.io/", "confirmed");
const keypair = Keypair.fromSecretKey(bs58.decode(PRIVATE_KEY));
const wallet = new NodeWallet(keypair);
const config = getConfig();

const client = await LendrClient.fetch({ config, wallet, connection });
```

## Common Patterns

### Pattern 1: Create Lending Account

```typescript
// Standard account creation
const lendrAccount = await client.createLendrAccount();

// With Fogo Session (gasless)
const lendrAccount = await client.createLendrAccountWithSession();
```

### Pattern 2: Fetch Banks (Markets)

```typescript
// Get bank by token symbol
const fogoBank = client.getBankByTokenSymbol("FOGO");
const usdcBank = client.getBankByTokenSymbol("USDC");

// Get bank by mint address
const bank = client.getBankByMint(address("So11111111111111111111111111111111111111112"));

// Get bank by public key
const bank = client.getBankByPk(address("BANK_ADDRESS"));
```

### Pattern 3: Deposit (Lend)

```typescript
const amount = 1000_000_000n; // 1 FOGO (9 decimals)
const bank = client.getBankByTokenSymbol("FOGO");

// Standard deposit
await lendrAccount.deposit(amount, bank.address);

// With Fogo Session (gasless)
await lendrAccount.depositWithSession(amount, bank.address, SESSION);
```

### Pattern 4: Borrow

```typescript
const amount = 100_000_000n; // 100 USDC (6 decimals)
const bank = client.getBankByTokenSymbol("USDC");

// Standard borrow
await lendrAccount.borrow(amount, bank.address);

// With Fogo Session
await lendrAccount.borrowWithSession(amount, bank.address, SESSION);
```

### Pattern 5: Repay

```typescript
const amount = 100_000_000n; // Repay amount
const bank = client.getBankByTokenSymbol("USDC");

// Standard repay
await lendrAccount.repay(amount, bank.address);

// With Fogo Session
await lendrAccount.repayWithSession(amount, bank.address, SESSION);
```

### Pattern 6: Withdraw

```typescript
const amount = 500_000_000n; // Withdraw amount
const bank = client.getBankByTokenSymbol("FOGO");

// Standard withdraw
await lendrAccount.withdraw(amount, bank.address);

// With Fogo Session
await lendrAccount.withdrawWithSession(amount, bank.address, SESSION);
```

### Pattern 7: Get Account Health

```typescript
const health = await lendrAccount.getHealth();
console.log("Health Factor:", health.healthFactor);
console.log("Total Deposits:", health.totalDeposits);
console.log("Total Borrows:", health.totalBorrows);
```

## Important Rules

1. **Cannot borrow from same bank where you have deposits** - Choose one action per bank
2. **Cannot lend to same bank where you have borrows** - Repay first
3. **Maintain healthy collateral ratio** - Monitor health factor to avoid liquidation

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `Bank not found` | Invalid token symbol | Check available banks |
| `Insufficient collateral` | Health factor too low | Add more collateral |
| `Same bank restriction` | Deposit + borrow in same bank | Use different banks |

## References

- Pyron Docs: https://docs.pyron.fi/
- Pyron SDK: https://www.npmjs.com/package/@pyron-finance/pyron-client
- Pyron GitHub: https://github.com/Pyron-Finance/
- Related: [fogo-core](../fogo-core/SKILL.md)
