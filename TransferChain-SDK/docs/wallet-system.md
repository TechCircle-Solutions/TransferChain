# Wallet System

> **Migration Notice:** This document describes the target Stellar wallet architecture. The current implementation uses ethers.js v6 SignerManager as the legacy reference.

## Table of Contents

- [SignerManager](#signermanager)
- [Signer Sources](#signer-sources)
- [Read-Only Mode](#read-only-mode)
- [Write Operations](#write-operations)
- [Multi-Signer Future](#multi-signer-future)
- [Security](#security)

---

## SignerManager

The `SignerManager` holds the active signing identity and provides it to the `TransactionManager` for write operations.

### Interface

```typescript
interface ISignerManager {
  /** Get the active keypair, or undefined if in read-only mode */
  getKeypair(): Keypair | undefined;

  /** Check if a signer is available */
  hasSigner(): boolean;
}
```

---

## Signer Sources

The SDK accepts signing identity from multiple sources:

### Secret Key

The most common pattern. A Stellar secret key creates a `Keypair`:

```typescript
const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  secretKey: "S...",
});
```

### Pre-Built Keypair

For environments where the SDK does not manage the key:

```typescript
const keypair = Keypair.fromSecret("S...");
const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  keypair,
});
```

### Wallet Adapter (Browser)

For browser applications, use a Stellar wallet adapter (Freighter, Lobstr):

```typescript
import { FreighterWallet } from "@transferchain/sdk/wallets";

const wallet = new FreighterWallet();
const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  wallet,
});
```

### Priority

If both `secretKey` and `keypair` are provided, `keypair` takes precedence. If `wallet` is provided, it takes precedence over both.

---

## Read-Only Mode

When no signer is provided, the SDK operates in full read-only mode:

- All read methods work normally
- All write methods throw `ValidationError` with code `SIGNER_REQUIRED`

---

## Write Operations

When a domain client method requires a write, the flow is:

1. Domain client calls `TransactionManager.execute()`
2. `TransactionManager` obtains the keypair from `SignerManager`
3. The `TransactionManager` builds the Soroban transaction
4. The transaction is simulated, signed, and submitted
5. The result is returned as `TransactionResult<T>`

The domain client never handles the signer directly.

---

## Security

### Key Handling Rules

| Rule | Description |
|------|-------------|
| No storage | Secret keys are never stored in localStorage, cookies, or disk |
| No logging | Secret keys are never logged, even at debug level |
| No transmission | Keys are only sent to the configured RPC endpoint via the Stellar SDK |
| No key generation | The SDK never generates keys internally |

### Environment Safety

For browser applications, use wallet adapters (Freighter, Lobstr) instead of raw secret keys. The SDK never prompts for key input or accesses browser storage.
