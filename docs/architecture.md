# TransferChain Architecture

> An open-source decentralized football transfer protocol built on Stellar.

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Layer Descriptions](#layer-descriptions)
- [Smart Contract Architecture](#smart-contract-architecture)
- [Data Flow](#data-flow)
- [Security Model](#security-model)

---

## Overview

TransferChain provides transparent, on-chain infrastructure for football player transfers. The protocol separates concerns into independent, auditable modules: identity registries, a transfer marketplace, agreement management, escrow custody, and protocol treasury.

All on-chain logic executes as Soroban smart contracts on the Stellar network. Applications interact with the protocol through the TransferChain SDK, which wraps all blockchain complexity behind domain-specific APIs.

---

## System Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Frontend Layer                     │
│              (React / Next.js / Tailwind)            │
│                                                     │
│  ┌─────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │   UI    │  │  Wallet  │  │  Transaction      │  │
│  │  Pages  │  │ Adapter  │  │  Status Display   │  │
│  └────┬────┘  └────┬─────┘  └────────┬──────────┘  │
│       └─────────────┼────────────────┘              │
└─────────────────────┼───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              TransferChain SDK                       │
│            (TypeScript / @stellar/stellar-sdk)       │
│                                                     │
│  ┌────────────┐  ┌──────────┐  ┌────────────────┐  │
│  │  Domain    │  │ Soroban  │  │ Event Manager  │  │
│  │  Clients   │  │ Contract │  │                │  │
│  │            │  │ Clients  │  │                │  │
│  └─────┬──────┘  └────┬─────┘  └───────┬────────┘  │
│        └───────────────┼───────────────┘            │
└────────────────────────┼────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│              Soroban Contracts                       │
│          (Rust / WebAssembly / Soroban SDK)          │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ access_      │  │ player_      │                 │
│  │ control      │  │ registry     │                 │
│  ├──────────────┤  ├──────────────┤                 │
│  │ config       │  │ club_        │                 │
│  │              │  │ registry     │                 │
│  ├──────────────┤  ├──────────────┤                 │
│  │ marketplace  │  │ agreement_   │                 │
│  │              │  │ manager      │                 │
│  ├──────────────┤  ├──────────────┤                 │
│  │ escrow       │  │ treasury     │                 │
│  └──────────────┘  └──────────────┘                 │
└────────────────────────┼────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│               Stellar Network                        │
│                                                     │
│  ┌──────────┐  ┌────────────┐  ┌────────────────┐  │
│  │ Stellar  │  │  Soroban   │  │  Stellar       │  │
│  │ Core     │  │  Runtime   │  │  RPC / Horizon │  │
│  └──────────┘  └────────────┘  └────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## Layer Descriptions

| Layer | Technology | Responsibility |
|-------|-----------|----------------|
| **Frontend** | React 19, Next.js 16, Tailwind CSS, Stellar wallet adapters | User interface, wallet connection, transaction signing, status display |
| **SDK** | TypeScript, @stellar/stellar-sdk | Type-safe contract interaction, event handling, workflow orchestration |
| **Smart Contracts** | Rust, Soroban SDK, WebAssembly | Protocol logic, state management, access control, escrow |
| **Blockchain** | Stellar Network | Consensus, finality, on-chain execution, asset infrastructure |
| **Metadata** | IPFS, HTTP | Player and club metadata, document storage (pluggable) |

---

## Smart Contract Architecture

```
access_control (hub)
    |
    ├── config                  (fees, assets, emergency controls)
    ├── player_registry         (player identity)
    ├── club_registry           (club identity)
    ├── marketplace             (listings, offers)
    ├── agreement_manager       (transfer agreements, clauses)
    ├── escrow                  (asset custody)
    └── treasury                (protocol revenue)
```

### Contract Responsibilities

| Contract | Domain | Key State | Key Operations |
|----------|--------|-----------|----------------|
| `access_control` | Security | Admin addresses, role assignments, paused flag | Grant/revoke roles, pause/unpause |
| `config` | Configuration | Fee bps, treasury address, supported assets, emergency mode | Update fees, manage supported assets |
| `player_registry` | Identity | Player records, ID counter | Register player, update metadata, set status |
| `club_registry` | Identity | Club records, ID counter | Register club, update metadata, set status |
| `marketplace` | Commerce | Listings, offers | Create/cancel listing, make/reject offer |
| `agreement_manager` | Legal | Agreements, clause sets | Create, approve, reject agreements |
| `escrow` | Finance | Deposits, balances | Deposit, release, refund assets |
| `treasury` | Revenue | Token balances per asset | Deposit/withdraw protocol revenue |

---

## Data Flow

### Player Registration Flow

```
Player Wallet → SDK → player_registry.register_player(name, metadata_uri)
                           │
                           ├── Validates caller == owner
                           ├── Assigns sequential player ID
                           ├── Stores player record
                           └── Emits PlayerRegistered event
```

### Transfer + Escrow Flow

```
1. Seller creates listing:    marketplace.create_listing(player_id, club_id, price)
2. Buyer makes offer:         marketplace.make_offer(listing_id, amount)
3. Seller accepts offer:      (off-chain agreement negotiation)
4. Agreement created:         agreement_manager.create_agreement(...)
5. Buyer deposits escrow:     escrow.deposit(asset, amount, agreement_id, payee)
6. Conditions verified:       (off-chain or oracle verification)
7. Seller releases escrow:    escrow.release(deposit_id)
8. Protocol fee collected:    treasury receives fee via config-managed deduction
```

---

## Security Model

| Mechanism | Implementation |
|-----------|---------------|
| **Access Control** | Role-based authorization via Soroban `require_auth()` and contract-stored role maps |
| **Admin Control** | Single admin address per contract, transferable |
| **Emergency Pause** | Protocol-wide pause via `access_control` contract |
| **Input Validation** | All public entry points validate arguments before state changes |
| **Fund Safety** | Escrow holds assets via Soroban token transfer; only authorized parties can release/refund |
| **Authorization** | Soroban transaction-level auth ensures only the invoker can trigger sensitive operations |
| **Immutability** | Stellar's consensus ensures finalized transactions cannot be reversed |

### Soroban-Specific Security Considerations

- Soroban contracts run in a sandboxed WASM environment
- Storage access is strictly scoped to the contract instance
- Authorization is checked at the protocol level via `require_auth()`
- Resource fees prevent denial-of-service attacks
- Contract code is immutable after deployment (no proxy patterns needed)
