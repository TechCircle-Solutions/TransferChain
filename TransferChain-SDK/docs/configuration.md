# Configuration

## Table of Contents

- [SDK Initialization](#sdk-initialization)
- [Configuration Interface](#configuration-interface)
- [Deferred Validation](#deferred-validation)
- [Immutable After Construction](#immutable-after-construction)
- [Deployment Manifest](#deployment-manifest)
- [Address Resolution](#address-resolution)
- [Environment Variables](#environment-variables)
- [Transaction Defaults](#transaction-defaults)
- [Metadata Configuration](#metadata-configuration)

---

## SDK Initialization

The `TransferChain` class accepts a configuration object at construction time.

```typescript
import { TransferChain } from "@transferchain/sdk";

const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  secretKey: "S...",
});
```

The SDK operates in read-only mode if no signer is provided:

```typescript
const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  // No secretKey or signer — read-only mode
});
```

---

## Configuration Interface

```typescript
interface SdkConfig {
  /** Stellar network passphrase (required) */
  networkPassphrase: string;

  /** Stellar RPC endpoint URL (required unless server is provided) */
  rpcUrl: string;

  /** Secret key for signing transactions (optional — read-only if omitted) */
  secretKey?: string;

  /** Pre-built Stellar Keypair instance (optional — overrides secretKey) */
  keypair?: Keypair;

  /** Pre-built SorobanRpc.Server instance (optional — overrides rpcUrl) */
  server?: SorobanRpc.Server;

  /** Contract addresses (optional — falls back to built-in manifest) */
  deployment?: DeploymentManifest;

  /** Logger implementation (optional — silent by default) */
  logger?: Logger;

  /** Middleware for the transaction pipeline (optional) */
  middleware?: Middleware[];

  /** Plugins to extend SDK functionality (optional) */
  plugins?: Plugin[];

  /** Metadata resolution configuration (optional) */
  metadata?: MetadataConfig;

  /** Default transaction parameters (optional) */
  transactions?: TransactionDefaults;
}
```

---

## Deferred Validation

Configuration is validated lazily. The constructor does not make RPC calls or validate addresses. Validation happens on first use:

- **Provider connectivity** is verified on the first read call
- **Signer validity** is verified on the first write call
- **Address checksums** are validated when the deployment manifest is first accessed

This keeps initialization fast and synchronous.

---

## Immutable After Construction

The `TransferChain` instance is effectively immutable after construction. Provider, signer, and contract references cannot be changed. To switch chains or signers, create a new instance.

This is a deliberate design choice that eliminates state-management bugs.

---

## Deployment Manifest

The SDK does not hardcode contract addresses. A `DeploymentManifest` maps chain IDs to deployed addresses:

```typescript
interface DeploymentManifest {
  accessControl: string;
  config: string;
  playerRegistry: string;
  clubRegistry: string;
  marketplace: string;
  agreementManager: string;
  escrow: string;
  treasury: string;
}
```

### Built-In Manifest

The SDK ships with a built-in manifest for known deployments:

| Network | Status |
|---------|--------|
| Stellar Testnet | Planned |
| Stellar Mainnet | Planned |

### Custom Manifest

Override or extend the built-in manifest via the `deployment` config option:

```typescript
const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  deployment: {
    accessControl: "C...",
    config: "C...",
    playerRegistry: "C...",
    clubRegistry: "C...",
    marketplace: "C...",
    agreementManager: "C...",
    escrow: "C...",
    treasury: "C...",
  },
});
```

---

## Address Resolution

When a domain client needs a contract address, it calls the `ContractRegistry` which resolves addresses in this order:

1. User-provided deployment manifest (from `SdkConfig.deployment`)
2. Built-in deployment manifest (shipped with the SDK)

If neither contains an address for the requested chain, a `ValidationError` is thrown with code `CHAIN_NOT_SUPPORTED`.

---

## Environment Variables

Environment variables are a development convenience only. They are never read in production builds.

| Environment Variable | Maps To | Description |
|---------------------|---------|-------------|
| `TRANSFERCHAIN_NETWORK_PASSPHRASE` | `config.networkPassphrase` | Stellar network passphrase |
| `TRANSFERCHAIN_RPC_URL` | `config.rpcUrl` | Stellar RPC endpoint URL |
| `TRANSFERCHAIN_SECRET_KEY` | `config.secretKey` | Signer secret key |

Use the optional `fromEnv()` helper to load configuration from environment variables:

```typescript
import { TransferChain, fromEnv } from "@transferchain/sdk";

const config = fromEnv();
const tc = new TransferChain(config);
```

> **Warning:** Never commit private keys to version control. Environment variables are for local development only.

---

## Transaction Defaults

Configure default parameters for all transactions:

```typescript
interface TransactionDefaults {
  /** Timeout in milliseconds for transaction confirmation (default: 120000) */
  timeout?: number;

  /** Whether to simulate transactions before submission (default: true) */
  simulate?: boolean;

  /** Resource fee limit override (in stroops) */
  resourceFeeLimit?: bigint;
}
```

---

## Metadata Configuration

Configure metadata resolution behavior:

```typescript
interface MetadataConfig {
  /** Custom protocol handlers (built-in: IPFS, HTTP) */
  protocols?: ProtocolHandler[];

  /** IPFS gateway URL (default: https://ipfs.io/ipfs/) */
  ipfsGateway?: string;

  /** Metadata cache time-to-live in milliseconds (default: 300000 = 5 minutes) */
  cacheTtl?: number;

  /** Maximum cache entries (default: 1000) */
  cacheMaxSize?: number;
}
```
