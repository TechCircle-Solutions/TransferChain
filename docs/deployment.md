# Deployment Guide

> Deploying TransferChain contracts and infrastructure to Stellar networks.

## Network Configuration

| Network | Passphrase | RPC URL | Explorer |
|---------|-----------|---------|----------|
| **Testnet** | `Test SDF Network ; September 2015` | `https://soroban-testnet.stellar.org` | [Stellar Expert (testnet)](https://stellar.expert/testnet) |
| **Mainnet** | `Public Global Stellar Network ; September 2015` | `https://soroban-mainnet.stellar.org` | [Stellar Expert (mainnet)](https://stellar.expert) |
| **Local** | `Standalone Network ; September 2024` | `http://localhost:8000/soroban/rpc` | N/A |

## Deploying Soroban Contracts

### Prerequisites

```bash
# Install Soroban CLI
cargo install --locked soroban-cli

# Configure stellar CLI for testnet
stellar network add testnet --rpc-url https://soroban-testnet.stellar.org --network-passphrase "Test SDF Network ; September 2015"

# Create a deployer account (if needed)
stellar keys generate deployer --network testnet
```

### Deploy Contracts

```bash
cd TransferChain-Contracts

# Build all contracts
soroban contract build

# Deploy each contract
# 1. Access Control
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/transferchain_access_control.wasm \
  --network testnet \
  --source deployer

# 2. Treasury (deploy before Config)
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/treasury.wasm \
  --network testnet \
  --source deployer

# 3. Config (requires treasury address)
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/transferchain_config.wasm \
  --network testnet \
  --source deployer

# 4. Player Registry
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/player_registry.wasm \
  --network testnet \
  --source deployer

# 5. Club Registry
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/club_registry.wasm \
  --network testnet \
  --source deployer

# 6. Marketplace
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/marketplace.wasm \
  --network testnet \
  --source deployer

# 7. Agreement Manager
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/agreement_manager.wasm \
  --network testnet \
  --source deployer

# 8. Escrow
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/escrow.wasm \
  --network testnet \
  --source deployer
```

### Initialize Contracts

After deployment, initialize each contract with the required configuration:

```bash
# Initialize Config with treasury address and marketplace fee
soroban contract invoke \
  --id <CONFIG_CONTRACT_ID> \
  --fn initialize \
  --arg admin:<DEPLOYER_ADDRESS> \
  --arg treasury:<TREASURY_CONTRACT_ID> \
  --arg marketplace_fee_bps:250 \
  --arg protocol_version:1 \
  --network testnet \
  --source deployer

# Grant roles to the deployer
soroban contract invoke \
  --id <ACCESS_CONTROL_CONTRACT_ID> \
  --fn grant_role \
  --arg role:admin \
  --arg address:<DEPLOYER_ADDRESS> \
  --network testnet \
  --source deployer
```

## Deployment Manifest

After deployment, record all contract addresses in a deployment manifest:

```json
{
  "network": "testnet",
  "networkPassphrase": "Test SDF Network ; September 2015",
  "deployer": "G...",
  "deployedAt": "2026-XX-XX",
  "contracts": {
    "access_control": "C...",
    "treasury": "C...",
    "config": "C...",
    "player_registry": "C...",
    "club_registry": "C...",
    "marketplace": "C...",
    "agreement_manager": "C...",
    "escrow": "C..."
  }
}
```

## SDK Configuration

Once deployed, configure the SDK with the deployment manifest:

```typescript
import { TransferChain } from "@transferchain/sdk";

const tc = new TransferChain({
  networkPassphrase: "Test SDF Network ; September 2015",
  rpcUrl: "https://soroban-testnet.stellar.org",
  secretKey: "S...",
  deployment: {
    accessControl: "C...",
    treasury: "C...",
    config: "C...",
    playerRegistry: "C...",
    clubRegistry: "C...",
    marketplace: "C...",
    agreementManager: "C...",
    escrow: "C...",
  },
});
```

## Frontend Configuration

```env
# .env.local
NEXT_PUBLIC_STELLAR_NETWORK=testnet
NEXT_PUBLIC_STELLAR_RPC_URL=https://soroban-testnet.stellar.org
NEXT_PUBLIC_ACCESS_CONTROL_CONTRACT=C...
NEXT_PUBLIC_CONFIG_CONTRACT=C...
NEXT_PUBLIC_PLAYER_REGISTRY_CONTRACT=C...
NEXT_PUBLIC_CLUB_REGISTRY_CONTRACT=C...
NEXT_PUBLIC_MARKETPLACE_CONTRACT=C...
NEXT_PUBLIC_AGREEMENT_MANAGER_CONTRACT=C...
NEXT_PUBLIC_ESCROW_CONTRACT=C...
NEXT_PUBLIC_TREASURY_CONTRACT=C...
```

## Verification

After deployment, verify contracts on Stellar Expert:

1. Navigate to `https://stellar.expert/testnet/contract/<CONTRACT_ID>`
2. Verify contract code matches the deployed WASM
3. Test contract functions via the invoke interface
4. Confirm event emission for state-changing operations

## Legacy Deployment (Injective EVM)

> The following is preserved for historical reference.

The Solidity prototype was deployed on Injective EVM Testnet (Chain ID 1439). The deployment manifest is at `TransferChain-Contracts/deployments/1439.json`. This deployment is no longer the active target.
