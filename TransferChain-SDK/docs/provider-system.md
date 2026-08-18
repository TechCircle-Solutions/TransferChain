# Server Manager

> **Migration Notice:** This document describes the target Stellar architecture. The current implementation uses ethers.js v6 ProviderManager as the legacy reference.

## Table of Contents

- [ServerManager](#servermanager)
- [Server Creation](#server-creation)
- [Server Caching](#server-caching)
- [Server Validation](#server-validation)
- [Read vs Write](#read-vs-write)
- [Chain Metadata](#chain-metadata)

---

## ServerManager

The `ServerManager` centralizes all Soroban RPC server lifecycle management. It is an internal service — consumers do not interact with it directly.

### Interface

```typescript
interface IServerManager {
  /** Get or create a server for the given network */
  getServer(networkPassphrase: string): SorobanRpc.Server;

  /** Manually register a server instance */
  setServer(networkPassphrase: string, server: SorobanRpc.Server): void;
}
```

---

## Server Creation

When a consumer provides a raw `rpcUrl`, the manager creates a `SorobanRpc.Server`:

```typescript
const server = new SorobanRpc.Server(config.rpcUrl, {
  allowHttp: config.rpcUrl.startsWith("http://"),
});
```

When a consumer provides a pre-built `Server` instance, it is used directly without wrapping.

If both `rpcUrl` and `server` are provided, `server` takes precedence.

---

## Server Caching

Servers are cached by network passphrase:

```typescript
private cache = new Map<string, SorobanRpc.Server>();
```

| Property | Behavior |
|----------|----------|
| Key | Network passphrase |
| Eviction | Never — servers are stateless connections |
| Memory | One server per unique RPC endpoint |

---

## Server Validation

Validation is deferred to first use:

### Network Validation

On the first RPC call, the SDK verifies the network passphrase matches the configured value.

### Connectivity Validation

The first RPC call also serves as a connectivity check. If the endpoint is unreachable, a `ProviderError` with code `CONNECTION_FAILED` is thrown.

### Why Deferred

- Constructor remains synchronous and fast
- Avoids unnecessary network calls when the SDK is used for type imports only

---

## Read vs Write

Soroban contracts support both read-only (` simulateTransaction`) and write (`sendTransaction`) operations. The `ServerManager` always provides the base server. The `SignerManager` handles keypair-attached transactions for write operations.

---

## Chain Metadata

The SDK ships with Stellar network metadata:

| Network | Passphrase | RPC URL |
|---------|-----------|---------|
| Testnet | `Test SDF Network ; September 2015` | `https://soroban-testnet.stellar.org` |
| Mainnet | `Public Global Stellar Network ; September 2015` | `https://soroban-mainnet.stellar.org` |
| Local | `Standalone Network ; September 2024` | `http://localhost:8000/soroban/rpc` |
