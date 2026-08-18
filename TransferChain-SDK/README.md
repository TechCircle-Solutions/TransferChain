# @transferchain/sdk

> **Migration Notice:** This SDK is being migrated from ethers.js v6 (EVM) to `@stellar/stellar-sdk` (Stellar/Soroban). The current implementation serves as the architecture reference. See [docs/stellar-migration.md](../docs/stellar-migration.md) for details.

Official TypeScript SDK for the [TransferChain](https://github.com/transferchain) smart contract protocol on Stellar.

The SDK is the only supported interface between applications and the TransferChain blockchain. Applications never manage blockchain infrastructure directly.

## Features

- **Type-safe** — Full TypeScript coverage for all contracts, events, and errors
- **Framework-agnostic** — Works in Node.js, browsers, and React Native
- **Tree-shakeable** — Import only what you use
- **Stellar-native** — Built on `@stellar/stellar-sdk` for Soroban contract interaction
- **Event system** — Typed subscriptions and historical queries
- **Metadata resolution** — IPFS and HTTP with caching
- **Workflow helpers** — Multi-step orchestration for transfers, listings, registrations

## Installation

```bash
pnpm add @transferchain/sdk
```

## Quick Start

```typescript
import { TransferChain } from "@transferchain/sdk";

const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  secretKey: "S...",
});

// Read on-chain state
const player = await tc.players.getPlayer("GA...");

// Submit a transaction
const result = await tc.marketplace.createListing({
  seller: "GA...",
  playerId: 1n,
  clubId: 1n,
  price: 1000n * 10n ** 7n,
  metadataUri: "ipfs://Qm...",
});

// Subscribe to events
const unsubscribe = tc.events.subscribe("ListingCreated", (event) => {
  console.log(`New listing by ${event.seller}`);
});

// Resolve metadata
const profile = await tc.metadata.resolvePlayer("ipfs://Qm...");

// Full transfer workflow
const transfer = await tc.workflows.transfer({
  agreement: { listingId: 1n, buyer: "GA...", seller: "GA...", ... },
  deposit: { asset: "CAS...", amount: 1000n, payee: "GA..." },
  approver: "GA...",
});
```

## Domain Clients

| Client | Contract | Purpose |
|--------|----------|---------|
| `tc.accessControl` | access_control | Roles, pause/unpause |
| `tc.config` | config | Treasury, fees, assets, emergency mode |
| `tc.players` | player_registry | Player registration, metadata, status |
| `tc.clubs` | club_registry | Club registration, metadata, status |
| `tc.marketplace` | marketplace | Listings and offers |
| `tc.agreements` | agreement_manager | Transfer agreements |
| `tc.escrow` | escrow | Escrow deposits |
| `tc.treasury` | treasury | Protocol treasury |
| `tc.events` | — | Event subscriptions and queries |
| `tc.metadata` | — | Off-chain metadata resolution |
| `tc.workflows` | — | Multi-step workflow orchestration |

## Legacy Prototype

The current SDK implementation uses ethers.js v6 and targets Injective EVM. This serves as the architecture reference for the Stellar re-implementation. The domain API surface (client methods, types, events) is designed to be preserved across the migration.

| Component | Current (Legacy) | Target |
|---|---|---|
| Blockchain client | ethers.js v6 | @stellar/stellar-sdk |
| Provider | JsonRpcProvider | SorobanRpc.Server |
| Signer | ethers.Wallet / ethers.Signer | Keypair / Freighter wallet adapter |
| Contract calls | ethers.Contract + ABI | Stellar SDK Contract class |
| Address format | `0x...` (hex) | `G...` / `C...` (StrKey) |
| Token standards | ERC-20 | Soroban tokens / Stellar assets |

## Documentation

- [Architecture](./docs/architecture.md)
- [Configuration](./docs/configuration.md)
- [Public API](./docs/public-api.md)
- [Error Handling](./docs/error-handling.md)
- [Contributing](./docs/contributing.md)

## License

MIT
