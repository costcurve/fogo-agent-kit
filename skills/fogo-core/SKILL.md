# Fogo Core Skill

## Overview

Core Fogo blockchain interactions - wallet setup, RPC connections, transactions, and Fogo Sessions.

## Prerequisites

```bash
npm install @solana/kit dotenv
```

**CRITICAL:** Use `@solana/kit` v2.x, NOT `@solana/web3.js` v1.x. The Fogo ecosystem SDKs are built for Solana Kit v2.

## Network Configuration

| Network | RPC URL | Explorer |
|---------|---------|----------|
| Mainnet | `https://mainnet.fogo.io/` | `https://explorer.fogo.io` |
| Testnet | `https://testnet.fogo.io/` | - |

**Key Specs:**
- Block time: ~40ms
- Confirmation: ~1.3 seconds
- Based on Firedancer client (Jump Crypto)
- 100% Solana compatible

## Token Addresses

| Token | Symbol | Mint Address | Decimals |
|-------|--------|--------------|----------|
| FOGO (native wrapped) | FOGO | `So11111111111111111111111111111111111111112` | 9 |
| USD Coin | USDC | `uSd2czE61Evaf76RNbq4KPpXnkiL3irdzgLFUMe3NoG` | 6 |
| Staked FOGO | stFOGO | `Brasa3xzkSC9XqMBEcN9v53x4oMkpb1nQwfaGMyJE88b` | 9 |
| iFOGO | iFOGO | `iFoGoY5nMWpuMJogR7xjUAWDJtygHDF17zREeP4MKuD` | 9 |

## Common Patterns

### Pattern 1: Create RPC Client

```typescript
import { createSolanaRpc } from "@solana/kit";

const rpc = createSolanaRpc("https://mainnet.fogo.io/");
```

### Pattern 2: Load Wallet from Keypair File

```typescript
import { createKeyPairSignerFromBytes } from "@solana/kit";
import fs from "fs";

async function loadWallet(path: string) {
  const keyPairBytes = new Uint8Array(
    JSON.parse(fs.readFileSync(path, "utf8"))
  );
  return await createKeyPairSignerFromBytes(keyPairBytes);
}

const wallet = await loadWallet("./wallet.json");
console.log("Address:", wallet.address);
```

### Pattern 3: Check Native FOGO Balance

```typescript
const balance = await rpc.getBalance(wallet.address).send();
const fogoBalance = Number(balance.value) / 1e9;
console.log("FOGO Balance:", fogoBalance);
```

### Pattern 4: Check Token Balance (SPL Token)

```typescript
import { address } from "@solana/kit";

const USDC_MINT = "uSd2czE61Evaf76RNbq4KPpXnkiL3irdzgLFUMe3NoG";

const tokenAccounts = await rpc.getTokenAccountsByOwner(
  wallet.address,
  { mint: address(USDC_MINT) },
  { encoding: "jsonParsed" }
).send();

const usdcBalance = tokenAccounts.value.length > 0
  ? BigInt(tokenAccounts.value[0].account.data.parsed.info.tokenAmount.amount)
  : 0n;

console.log("USDC Balance:", Number(usdcBalance) / 1e6);
```

### Pattern 5: Send FOGO Transaction

```typescript
import {
  createTransactionMessage,
  setTransactionMessageLifetimeUsingBlockhash,
  setTransactionMessageFeePayerSigner,
  appendTransactionMessageInstruction,
  signTransaction,
  getSignatureFromTransaction,
  lamports,
  address,
} from "@solana/kit";
import { getTransferSolInstruction } from "@solana-program/system";

async function sendFogo(rpc, wallet, recipient: string, amountFogo: number) {
  const amount = lamports(BigInt(Math.floor(amountFogo * 1e9)));

  const transferIx = getTransferSolInstruction({
    source: wallet,
    destination: address(recipient),
    amount,
  });

  const { value: latestBlockhash } = await rpc.getLatestBlockhash().send();

  let tx = createTransactionMessage({ version: 0 });
  tx = setTransactionMessageLifetimeUsingBlockhash(latestBlockhash, tx);
  tx = setTransactionMessageFeePayerSigner(wallet, tx);
  tx = appendTransactionMessageInstruction(transferIx, tx);

  const signedTx = await signTransaction([wallet.keyPair], tx);
  const signature = getSignatureFromTransaction(signedTx);

  await rpc.sendTransaction(signedTx, { encoding: "base64" }).send();

  return signature;
}
```

### Pattern 6: Transaction Confirmation

```typescript
async function confirmTransaction(rpc, signature: string, maxRetries = 30) {
  for (let i = 0; i < maxRetries; i++) {
    const status = await rpc.getSignatureStatuses([signature]).send();
    const confirmation = status.value[0]?.confirmationStatus;

    if (confirmation === "confirmed" || confirmation === "finalized") {
      return true;
    }

    await new Promise(r => setTimeout(r, 500));
  }
  throw new Error("Transaction confirmation timeout");
}
```

### Pattern 7: Execute Multiple Instructions

```typescript
import { appendTransactionMessageInstructions } from "@solana/kit";

async function executeInstructions(rpc, instructions, wallet) {
  const { value: latestBlockhash } = await rpc.getLatestBlockhash().send();

  let tx = createTransactionMessage({ version: 0 });
  tx = setTransactionMessageLifetimeUsingBlockhash(latestBlockhash, tx);
  tx = setTransactionMessageFeePayerSigner(wallet, tx);
  tx = appendTransactionMessageInstructions(instructions, tx);

  const signedTx = await signTransaction([wallet.keyPair], tx);
  const signature = getSignatureFromTransaction(signedTx);

  await rpc.sendTransaction(signedTx, {
    encoding: "base64",
    skipPreflight: false,
  }).send();

  await confirmTransaction(rpc, signature);
  return signature;
}
```

## Fogo Sessions

Fogo Sessions enable gasless, session-based interactions using account abstraction.

```typescript
// Session signer pattern used by Vortex SDK
const sessionSigner = {
  address: wallet.address,
  signer: wallet,
};

// Pass to SDK functions that support sessions
// signerIsSession = true for session-based transactions
```

## Error Handling

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `insufficient funds` | Not enough FOGO for gas | Add FOGO to wallet |
| `blockhash not found` | Transaction expired | Get fresh blockhash, retry |
| `Transaction simulation failed` | Various | Check logs for specific error |

### Error Handling Pattern

```typescript
try {
  const sig = await executeInstructions(rpc, instructions, wallet);
  console.log("Success:", sig);
} catch (error: any) {
  if (error.context?.logs) {
    console.log("Program logs:", error.context.logs);
  }
  // Check for specific error codes
  if (error.message.includes("insufficient funds")) {
    console.log("Need more FOGO for gas");
  }
}
```

## Environment Variables

```env
# Fogo Mainnet
RPC_URL=https://mainnet.fogo.io/
WALLET_PATH=./wallet.json
```

## References

- Fogo Docs: https://docs.fogo.io/
- Community Docs: https://community-docs.fogo.io/
- Building Guide: https://docs.fogo.io/user-guides/building-on-fogo.html
- Fogo Explorer: https://explorer.fogo.io
- Fogoscan: https://fogoscan.com
