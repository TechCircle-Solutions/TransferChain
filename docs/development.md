# Development Guide

> Setting up a local development environment for TransferChain.

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 18+ | JavaScript runtime |
| pnpm | 9+ | Package manager (SDK) |
| npm | 10+ | Package manager (Frontend) |
| Rust | 1.82+ | Soroban contract compilation |
| Soroban CLI | Latest | Contract build and deployment |
| Git | Latest | Version control |

## Repository Structure

```
TransferChain/
├── TransferChain-Contracts/     # Smart contracts (Soroban/Rust + legacy Solidity)
├── TransferChain-SDK/           # TypeScript SDK
├── TransferChain-frontend/      # Next.js web application
├── docs/                        # Project documentation
└── README.md                    # Root README
```

## Smart Contracts Development

### Soroban Contracts (Target)

```bash
cd TransferChain-Contracts

# Build Soroban contracts
soroban contract build

# Run Soroban contract tests
cargo test

# Start local Soroban network
soroban lab Local V3

# Deploy to local network
soroban contract deploy --wasm target/wasm32-unknown-unknown/release/<contract>.wasm -- --network local
```

### Legacy Solidity Contracts (Prototype)

```bash
cd TransferChain-Contracts

# Install Foundry dependencies
forge install

# Build
forge build

# Test
forge test -vvv

# Start local Anvil node
anvil
```

## SDK Development

```bash
cd TransferChain-SDK

# Install dependencies
pnpm install

# Build the SDK
pnpm build

# Run unit tests
pnpm test:unit

# Run all tests
pnpm test:all

# Type checking
pnpm typecheck

# Linting
pnpm lint
pnpm format
```

### SDK Integration Testing

Integration tests run against a local Soroban network:

```bash
# Terminal 1: Start local Soroban
soroban lab Local V3

# Terminal 2: Run integration tests
cd TransferChain-SDK
pnpm test:integration
```

## Frontend Development

```bash
cd TransferChain-frontend

# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Lint
npm run lint
```

## Environment Variables

### SDK

| Variable | Description | Required |
|----------|-------------|----------|
| `TRANSFERCHAIN_NETWORK_PASSPHRASE` | Stellar network passphrase | Yes |
| `TRANSFERCHAIN_RPC_URL` | Stellar RPC / Horizon URL | Yes |
| `TRANSFERCHAIN_SECRET_KEY` | Secret key for signing (dev only) | No (read-only if omitted) |

### Frontend

| Variable | Description | Required |
|----------|-------------|----------|
| `NEXT_PUBLIC_STELLAR_NETWORK` | `testnet` or `mainnet` | Yes |
| `NEXT_PUBLIC_STELLAR_RPC_URL` | Stellar RPC endpoint | Yes |

## Testing Strategy

| Layer | Framework | Coverage Target |
|-------|-----------|----------------|
| Soroban contracts | `cargo test` | 90%+ |
| SDK unit tests | Vitest | 90%+ |
| SDK integration tests | Vitest + local network | Key flows |
| Frontend | Manual + future E2E | Critical paths |

## Debugging

### Soroban Contracts

- Use `soroban contract invoke` for manual testing
- Check contract state via `soroban contract read`
- Logs are available in Soroban RPC responses

### SDK

- Set `logger: console` in `SdkConfig` for debug logging
- Use `pnpm test:watch` during development

### Frontend

- Use Next.js developer tools
- Check browser console for transaction errors
- Stellar Expert for testnet transaction inspection
