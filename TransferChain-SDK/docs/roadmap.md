# Roadmap

## Table of Contents

- [Phase 1: Soroban Contracts (v1.0)](#phase-1-soroban-contracts-v10)
- [Phase 2: Stellar SDK (v1.x)](#phase-2-stellar-sdk-v1x)
- [Phase 3: Frontend Integration (v2.0)](#phase-3-frontend-integration-v20)
- [Phase 4: Developer Experience (v2.x)](#phase-4-developer-experience-v2x)
- [Phase 5: Ecosystem (v3.0)](#phase-5-ecosystem-v30)
- [Versioning Policy](#versioning-policy)
- [Breaking Change Policy](#breaking-change-policy)
- [Contract Version Coupling](#contract-version-coupling)

---

## Phase 1: Soroban Contracts (v1.0)

Re-implement all 8 contract domains as native Soroban Rust contracts.

| Feature | Status |
|---------|--------|
| `access_control` contract | Planned |
| `config` contract | Planned |
| `player_registry` contract | Planned |
| `club_registry` contract | Planned |
| `marketplace` contract | Planned |
| `agreement_manager` contract | Planned |
| `escrow` contract | Planned |
| `treasury` contract | Planned |
| Soroban unit tests | Planned |
| Stellar Testnet deployment | Planned |

**Target:** All 8 contract domains implemented in Rust, compiled to WASM, deployed on Stellar Testnet.

---

## Phase 2: Stellar SDK (v1.x)

Build the TypeScript SDK on `@stellar/stellar-sdk`.

| Feature | Status |
|---------|--------|
| ServerManager (replaces ProviderManager) | Planned |
| SignerManager (Stellar Keypair) | Planned |
| 8 domain clients | Planned |
| Typed event system (Soroban events) | Planned |
| Metadata resolution (IPFS + HTTP) | Planned |
| Error normalization | Planned |
| Workflow helpers (transfer, listing, registration) | Planned |
| Unit tests (90% coverage) | Planned |
| Integration tests (Soroban local network) | Planned |
| Full documentation | Planned |

**Target:** Any developer can read on-chain state, submit transactions, and resolve metadata using a single dependency.

---

## Phase 3: Frontend Integration (v2.0)

Migrate the frontend from EVM wallets to Stellar wallet adapters.

| Feature | Description |
|---------|-------------|
| Stellar wallet connection | Freighter, Lobstr, and other Stellar wallet adapters |
| Soroban contract interaction | Full integration via the TransferChain SDK |
| Transaction status | Real-time status and Stellar Expert explorer links |
| Sponsored transactions | Fee sponsorship for improved UX |

---

## Phase 4: Developer Experience (v2.x)

Enhancements for application developers.

| Feature | Description |
|---------|-------------|
| React hooks | `@transferchain/react` — `usePlayer()`, `useListing()`, `useEvents()` |
| CLI tool | `@transferchain/cli` — contract deployment, address management |
| Batch reads | Multicall support for reading multiple contract states |
| Transaction simulation | Preview transaction effects before submission |
| Improved error messages | Human-readable error messages with suggested actions |

---

## Phase 5: Ecosystem (v3.0)

Platform and community extensions.

| Feature | Description |
|---------|-------------|
| Plugin marketplace | Discover and install community plugins |
| Analytics SDK | `@transferchain/analytics` — transfer analytics and reporting |
| Notification SDK | `@transferchain/notifications` — event-driven notifications |
| TypeScript codegen | Auto-generate SDK clients from Soroban contract specs |
| Mobile SDK | React Native optimized build |

---

## Versioning Policy

The SDK follows [Semantic Versioning](https://semver.org/) 2.0.0:

| Change Type | Version Bump | Example |
|------------|-------------|---------|
| Breaking API change | MAJOR | v1.0.0 -> v2.0.0 |
| New feature (backward-compatible) | MINOR | v1.0.0 -> v1.1.0 |
| Bug fix (backward-compatible) | PATCH | v1.0.0 -> v1.0.1 |

### Pre-release

- Alpha: `v1.1.0-alpha.1` — unstable, API may change
- Beta: `v1.1.0-beta.1` — feature complete, API stable
- RC: `v1.1.0-rc.1` — release candidate

---

## Breaking Change Policy

1. **Deprecate first.** In a minor release, mark the old API as deprecated with a migration guide
2. **Remove later.** In the next major release, remove the deprecated API
3. **Provide migration tools.** Codemods or migration scripts where feasible

---

## Contract Version Coupling

The SDK version is independent of the smart contract protocol version. The `protocolVersion` field on `config` is a protocol-level concern.

The SDK is versioned based on its own API stability. Contract upgrades that add new functions or events are handled as SDK minor releases. Contract upgrades that change function signatures or event structures require SDK major releases.
