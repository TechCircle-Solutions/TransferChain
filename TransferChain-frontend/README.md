# TransferChain Frontend

> **Migration Notice:** This frontend is being migrated from EVM wallet integration (wagmi + Reown AppKit) to Stellar wallet adapters (Freighter, Lobstr). See [docs/stellar-migration.md](../docs/stellar-migration.md) for details.

## Overview

Modern web application for interacting with the TransferChain protocol on Stellar.

Built with Next.js 16, React 19, and Tailwind CSS.

## Features

- Player and club registration flows
- Marketplace browsing and listing creation
- Wallet connection via Stellar wallet adapters
- Transaction status and explorer links
- Responsive design with Tailwind CSS

## Getting Started

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser.

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Framework | Next.js 16 |
| UI Library | React 19 |
| Styling | Tailwind CSS 4 |
| Wallet | Stellar wallet adapters |
| State | TanStack React Query |
| Language | TypeScript 5 |

## Current Status

The frontend MVP was built with wagmi + Reown AppKit targeting Injective EVM. The active implementation will integrate Stellar wallet adapters for connection and transaction signing.

### What Changes for Stellar

| Component | Current (EVM) | Target (Stellar) |
|---|---|---|
| Wallet connection | Reown AppKit + wagmi | Freighter / Lobstr wallet adapter |
| Network config | Injective EVM Testnet (chainId 1439) | Stellar Testnet |
| ABI files | Solidity JSON ABI | Soroban contract spec |
| Transaction handling | wagmi hooks | Stellar SDK transaction builder |
| Address display | `0x...` | `G...` / `C...` (StrKey) |
| Block explorer | Blockscout | Stellar Expert |

## Development

```bash
npm run dev     # Start development server
npm run build   # Build for production
npm start       # Start production server
npm run lint    # Run linter
```

## License

MIT
