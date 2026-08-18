# Account Abstraction & Sponsored Transactions

> This document describes the SDK's readiness for Stellar sponsored transaction support. Sponsored transactions are not yet implemented.

## Table of Contents

- [Current State](#current-state)
- [Stellar Sponsored Transactions](#stellar-sponsored-transactions)
- [Future: Native Sponsored Support](#future-native-sponsored-support)
- [Stellar vs EVM Account Abstraction](#stellar-vs-evm-account-abstraction)

---

## Current State

The SDK currently uses a `Keypair` or secret key for transaction submission. Sponsored transactions (where a third party pays fees) are not implemented.

Consumers who want sponsored transactions can build custom transaction flows outside the SDK. The SDK architecture supports adding this as a first-class feature.

---

## Stellar Sponsored Transactions

Stellar has native support for fee sponsorship and authorization delegation, which provides similar benefits to EVM account abstraction but with different mechanics:

### Fee Sponsorship (fee-bumping)

A third-party account can pay the network fee for a transaction:

```typescript
// Sponsor pays the network fee
const baseFee = await server.getBaseFee();
const sponsoredTx = new TransactionBuilder(sponsorAccount, {
  fee: baseFee,
})
  .addOperation(contract.call("register_player", ...))
  .setNetworkPassphrase(networkPassphrase)
  .build();

// Sponsor signs first (authorizes the fee payment)
sponsoredTx.sign(sponsorKeypair);

// Invoker signs second (authorizes the contract call)
sponsoredTx.sign(invokerKeypair);

// Submit
await server.sendTransaction(sponsoredTx);
```

### Authorization Entries

Soroban uses authorization entries to delegate signing authority. A contract can require authorization from multiple parties in a single transaction.

---

## Future: Native Sponsored Support

When sponsored transactions become a product requirement, the SDK can add:

### SponsorManager

A dedicated service for managing fee sponsorship:

```typescript
const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  sponsor: {
    secretKey: "S...", // or wallet adapter
  },
});

// All write operations automatically use fee sponsorship
await tc.marketplace.createListing(params);
```

### Sponsorship Middleware

Built-in middleware for common sponsorship patterns:

- **FeeBumpMiddleware** — Automatically wraps transactions with fee sponsorship
- **AuthorizationMiddleware** — Manages multi-party authorization entries
- **BatchMiddleware** — Combines multiple operations into a single transaction

---

## Stellar vs EVM Account Abstraction

| Aspect | EVM (ERC-4337) | Stellar (Fee Sponsorship) |
|--------|---------------|---------------------------|
| Mechanism | UserOperation + Bundler | Fee-bump transaction + Auth entries |
| Fee payment | Paymaster contract | Sponsor account |
| Authorization | Account contract validates | `require_auth()` per invoker |
| Nonce management | Smart account nonces | Sequence numbers |
| Complexity | High (bundler, paymaster, factory) | Low (native protocol support) |

### What Does NOT Change

- Domain client APIs remain identical
- `TransactionResult` return type is unchanged
- Event decoding is unchanged
- Read methods are completely unaffected
