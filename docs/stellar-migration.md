# Stellar Migration Guide

> TransferChain migration from Injective EVM (Solidity/Foundry) to Stellar (Soroban/Rust).

## Table of Contents

- [Overview](#overview)
- [Migration Strategy](#migration-strategy)
- [Contract Migration Map](#contract-migration-map)
- [Domain Model Mapping](#domain-model-mapping)
- [Asset and Escrow Design](#asset-and-escrow-design)
- [Authorization Model](#authorization-model)
- [Event System](#event-system)
- [SDK Changes](#sdk-changes)
- [Frontend Changes](#frontend-changes)
- [What Remains from the Prototype](#what-remains-from-the-prototype)

---

## Overview

TransferChain was originally designed and prototyped as Solidity smart contracts deployed on Injective EVM. The project is now migrating to Stellar's Soroban smart contract platform.

**Key architectural difference:** Soroban contracts are written in Rust and compiled to WebAssembly. The existing Solidity contracts cannot run natively on Stellar. They serve as the **canonical business-logic reference** for the Soroban re-implementation.

---

## Migration Strategy

| Strategy | Description |
|----------|-------------|
| **A. Native Soroban (preferred)** | Re-implement all 8 contract domains in Rust, compiled to WASM via Soroban SDK |
| **B. Legacy retention** | Preserve Solidity source as reference implementation for audit trail |

The project follows **Strategy A** — native Soroban Rust contracts. The Solidity code is retained as a legacy prototype under `TransferChain-Contracts/` and is clearly marked as such.

---

## Contract Migration Map

| Existing Contract | Soroban Contract | Responsibility | Status |
|---|---|---|---|
| `TransferChainAccessControl` | `access_control` | Protocol-wide RBAC, pause/unpause | **PLANNED** |
| `TransferChainConfig` | `config` | Fee parameters, payment assets, emergency mode | **PLANNED** |
| `PlayerRegistry` | `player_registry` | Player identity registration, metadata pointers | **PLANNED** |
| `ClubRegistry` | `club_registry` | Club identity registration, metadata pointers | **PLANNED** |
| `TransferMarketplace` | `marketplace` | Listing lifecycle, offers, negotiation | **PLANNED** |
| `TransferAgreementManager` | `agreement_manager` | Transfer agreements, commercial clauses | **PLANNED** |
| `Escrow` | `escrow` | Stellar asset fund custody, release/refund flows | **PLANNED** |
| `Treasury` | `treasury` | Protocol revenue, controlled withdrawals | **PLANNED** |

### Status Definitions

- **IMPLEMENTED** — Code is written, tested, and deployed on Stellar testnet
- **IN DEVELOPMENT** — Code exists but is not yet production-ready
- **PLANNED** — Design is documented, implementation has not started

---

## Domain Model Mapping

### Access Control

| Solidity (OpenZeppelin) | Soroban Equivalent |
|---|---|
| `AccessControl` (role-based) | Soroban `#[authorities]` macro + contract-owned `StorageMap` for role assignments |
| `DEFAULT_ADMIN_ROLE` | Admin identity stored in contract instance storage |
| `PAUSER_ROLE` | Dedicated pauser identity in contract storage |
| `Pausable` | Contract-level `paused` flag in persistent storage |

Soroban uses `require_auth()` and `require_auth_except()` for authorization checks. Role-based access is implemented via a `StorageMap<Symbol, Address>` mapping role identifiers to authorized addresses.

### Configuration

| Solidity | Soroban Equivalent |
|---|---|
| `Ownable` | Contract admin stored in instance storage; admin-only helper function |
| `mapping(address => bool) supportedPaymentTokens` | `StorageMap<Address, bool>` for supported Stellar assets |
| `uint256 marketplaceFeeBps` | `StorageKey<u64>` for fee basis points |
| `bool emergencyMode` | `StorageKey<bool>` for emergency flag |

### Player Registry

| Solidity | Soroban Equivalent |
|---|---|
| `mapping(address => Player) players` | `StorageMap<Address, Player>` |
| `mapping(uint256 => address) playerById` | `StorageMap<u64, Address>` |
| `mapping(address => bool) registeredPlayers` | `StorageMap<Address, bool>` |
| `uint256 nextPlayerId` | `StorageKey<u64>` auto-incrementing ID |
| `block.timestamp` | Soroban `env.ledger().timestamp()` |

### Club Registry

Same pattern as Player Registry with additional fields (country, city, league, logoUri, website).

### Marketplace

| Solidity | Soroban Equivalent |
|---|---|
| `mapping(uint256 => Listing) listings` | `StorageMap<u64, Listing>` |
| `mapping(uint256 => mapping(address => Offer)) offers` | `StorageMap<u64, StorageMap<Address, Offer>>` |
| `block.timestamp` | `env.ledger().timestamp()` |

### Transfer Agreement Manager

| Solidity | Soroban Equivalent |
|---|---|
| `mapping(uint256 => Agreement) agreements` | `StorageMap<u64, Agreement>` |
| Complex `ClauseSet` struct | Same struct encoded via Soroban `#[contracttype]` |

### Escrow

| Solidity | Soroban Equivalent |
|---|---|
| `IERC20` / `SafeERC20` | Stellar asset transfers via `token::transfer()` or `StellarAssetClient` |
| `token.safeTransferFrom(payer, this, amount)` | `token::transfer(env, token_address, &payer, &escrow_address, &amount)` |
| `mapping(uint256 => Deposit) deposits` | `StorageMap<u64, Deposit>` |

### Treasury

| Solidity | Soroban Equivalent |
|---|---|
| `mapping(address => uint256) tokenBalance` | `StorageMap<Address, i128>` (Soroban uses `i128` for amounts) |
| `IERC20` transfers | Stellar asset transfers via Soroban token interface |

---

## Asset and Escrow Design

### Settlement Asset

On Stellar, assets are represented as **trustline-based assets** (created via `Asset::new()`) or **Soroban tokens** (smart-contract-based tokens). TransferChain escrow will support:

| Asset Type | Description | Status |
|---|---|---|
| **Stellar-native USDC** (Circle) | Bridged USDC on Stellar via Circle CCTP | **PLANNED** |
| **Soroban token contracts** | Any Soroban-compatible token contract | **PLANNED** |
| **Custom protocol tokens** | TransferChain-specific settlement token | **FUTURE** |

### Escrow Flow

```
Buyer Club                          Escrow Contract                      Seller Club
    |                                    |                                    |
    |-- deposit(token, amount) --------->|                                    |
    |   [token::transfer to escrow]      |                                    |
    |                                    |-- emit DepositFunded ------------->|
    |                                    |                                    |
    |                                    |<-- release(depositId) -------------|
    |                                    |   [token::transfer to payee]       |
    |                                    |-- emit DepositReleased ----------->|
```

### Authorization

- **Deposit**: Only the payer (buyer club) can authorize the deposit
- **Release**: Only the payee (seller club) can authorize release after conditions are met
- **Refund**: Only the original payer can authorize a refund
- **Admin Override**: Treasury admin can trigger dispute resolution

### Failure Handling

- If a deposit fails (insufficient balance, asset not trusted), the transaction aborts
- If a release is unauthorized, `require_auth()` fails the transaction
- Dispute resolution is handled by the escrow manager role

---

## Authorization Model

Soroban uses a different authorization model than EVM:

| EVM Concept | Soroban Equivalent |
|---|---|
| `msg.sender` | `env.invoker()` or `Address` passed as argument + `require_auth()` |
| `tx.origin` | Not directly available; not needed for contract calls |
| `onlyOwner` modifier | `require_auth()` against stored admin address |
| Role-based (`hasRole`) | Custom `StorageMap<Symbol, Address>` + `require_auth()` |
| `Ownable` | Admin address in instance storage |

**Critical difference:** Soroban transactions can invoke multiple contracts in a single transaction, with each contract's authorization checked independently via `require_auth()`. This enables powerful multi-contract workflows.

---

## Event System

| Solidity | Soroban Equivalent |
|---|---|
| `emit EventName(args...)` | `env.events().publish(tag, data)` |
| Indexed parameters | Soroban events use a `Symbol` topic + `Bytes` data |
| `eth_getLogs` for historical queries | Soroban event queries via Stellar RPC `getEvents()` |
| WebSocket subscription | Stellar RPC SSE (Server-Sent Events) or polling |

Soroban events are published with a `Symbol` topic tag and an `xdr::Bytes` data payload. The SDK will decode these into typed TypeScript events.

---

## SDK Changes

The SDK must migrate from ethers.js to Stellar/Soroban tooling:

| Component | Current (EVM) | Target (Stellar) |
|---|---|---|
| **Blockchain client** | ethers.js v6 | `@stellar/stellar-sdk` (Horizon + Soroban RPC) |
| **Provider** | `ethers.JsonRpcProvider` | `SorobanRpc.Server` |
| **Signer** | `ethers.Wallet` / `ethers.Signer` | `Keypair` (server-side) or `Freighter` wallet adapter (client-side) |
| **Contract interaction** | `ethers.Contract` + ABI | `@stellar/stellar-sdk` `Contract` class + Soroban contract spec |
| **Transaction building** | `contract.functionName(args)` | `contract.call(method, ...args)` with `TransactionBuilder` |
| **Transaction signing** | `signer.sendTransaction(tx)` | `tx.sign(keypair)` or wallet adapter |
| **Transaction submission** | `provider.sendRawTransaction(tx)` | `server.sendTransaction(tx)` |
| **Event listening** | `contract.on('Event', callback)` | `server.getEvents()` with SSE/polling |
| **Address format** | `0x...` (20 bytes, hex) | `G...` / `C...` (Stellar StrKey) |
| **Token standards** | ERC-20 | Soroban tokens / Stellar assets |
| **Gas model** | Gas limit + gas price | Soroban resource fee (compute + storage) |
| **Network config** | Chain ID + RPC URL | Network passphrase + Horizon/Soroban RPC URL |

### SDK Domain API (preserved)

The high-level domain API remains the same:

```typescript
sdk.players.register(...)
sdk.players.get(...)
sdk.clubs.register(...)
sdk.marketplace.createListing(...)
sdk.agreements.create(...)
sdk.escrow.deposit(...)
sdk.escrow.release(...)
```

Under the hood, these methods will call Soroban contracts via the Stellar SDK.

---

## Frontend Changes

| Component | Current (EVM) | Target (Stellar) |
|---|---|---|
| **Wallet connection** | Reown AppKit + wagmi | Stellar wallet adapter (Freighter, Lobstr, etc.) |
| **Network config** | Injective EVM Testnet (chainId 1439) | Stellar Testnet / Mainnet |
| **ABI files** | Solidity JSON ABI | Soroban contract spec (auto-generated from Rust) |
| **Transaction handling** | wagmi `useWriteContract` | Stellar SDK transaction builder |
| **Address display** | `0x...` | `G...` / `C...` (StrKey format) |
| **Block explorer** | Blockscout | Stellar Expert |

---

## What Remains from the Prototype

The following are preserved as the legacy Injective EVM prototype:

| Artifact | Location | Purpose |
|---|---|---|
| Solidity contracts | `TransferChain-Contracts/src/` | Business logic reference |
| Foundry tests | `TransferChain-Contracts/test/` | Behavior specification |
| Deploy script | `TransferChain-Contracts/script/` | Deployment reference |
| Deployment manifest | `TransferChain-Contracts/deployments/1439.json` | Historical deployment record |
| EVM ABIs | `TransferChain-SDK/abi/`, `TransferChain-frontend/src/abis/` | Reference ABIs (to be replaced with Soroban specs) |
| ethers.js SDK | `TransferChain-SDK/src/` | Architecture reference (to be re-implemented with Stellar SDK) |
| wagmi frontend | `TransferChain-frontend/src/` | UI reference (to be re-implemented with Stellar wallet adapters) |

All of these are clearly documented as the **legacy prototype** and are not the active target implementation.
