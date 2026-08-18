# Network Layer

> **Migration Notice:** This document describes the target Stellar network architecture. The current implementation uses ethers.js v6 as the legacy reference.

## Table of Contents

- [Design Philosophy](#design-philosophy)
- [Required Server Capabilities](#required-server-capabilities)
- [Server Types](#server-types)
- [Server Validation](#server-validation)
- [Error Handling](#error-handling)

---

## Design Philosophy

The SDK uses `@stellar/stellar-sdk` as its network layer. It wraps Soroban RPC and Horizon API calls behind domain-specific abstractions.

The SDK's value is in the domain model layer above the Stellar SDK — typed contract clients, error normalization, event decoding, and metadata resolution — not in reimplementing network primitives.

---

## Required Server Capabilities

The SDK requires the Stellar RPC server to support these operations:

| Operation | Used By | Purpose |
|-----------|---------|---------|
| `getNetwork` | ServerManager | Network passphrase validation |
| `simulateTransaction` | TransactionManager | Read contract state and simulate writes |
| `sendTransaction` | TransactionManager | Submit signed transactions |
| `getEvents` | EventManager | Query Soroban events |
| `getTransaction` | TransactionManager | Poll for TX confirmation |

---

## Server Types

| Server | Supported | Notes |
|--------|-----------|-------|
| `SorobanRpc.Server` | Yes | Primary target. Works in Node.js and browsers. |
| Custom Soroban RPC | Yes | Any endpoint implementing the Soroban RPC specification |

---

## Server Validation

### Network Validation

On the first RPC call, the SDK validates the network passphrase.

### Connectivity Check

If the first RPC call fails, a `ProviderError` with code `CONNECTION_FAILED` is thrown.

### Why Deferred

- Keeps the constructor synchronous and fast
- Avoids network calls for type-only imports

---

## Error Handling

Network errors are wrapped in `ProviderError`:

| Source Error | SDK Error Code | Message |
|-------------|----------------|---------|
| DNS resolution failure | `CONNECTION_FAILED` | Cannot connect to RPC endpoint |
| Connection timeout | `REQUEST_TIMEOUT` | RPC request timed out |
| Network passphrase mismatch | `CHAIN_MISMATCH` | Expected network X, got Y |
| RPC method not supported | `CONNECTION_FAILED` | Server does not support required method |
| HTTP 429 (rate limit) | `REQUEST_TIMEOUT` | RPC rate limit exceeded |

See [Error Handling](./error-handling.md) for the full error hierarchy.
