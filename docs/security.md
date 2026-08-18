# Security Policy

> TransferChain security model and vulnerability disclosure process.

## Security Model

### Contract Security

TransferChain Soroban contracts implement the following security measures:

| Mechanism | Implementation |
|-----------|---------------|
| **Access Control** | Role-based authorization via Soroban `require_auth()` |
| **Admin Control** | Single admin address per contract, transferable |
| **Emergency Pause** | Protocol-wide pause via `access_control` contract |
| **Input Validation** | All public entry points validate arguments |
| **Fund Safety** | Escrow holds assets via Soroban token transfers |
| **Authorization** | Transaction-level auth ensures only invokers trigger operations |
| **Immutability** | Contract code is immutable after deployment |

### Soroban Security Properties

- Soroban contracts run in a sandboxed WASM environment
- Storage access is scoped to the contract instance
- Authorization is checked at the protocol level
- Resource fees prevent denial-of-service attacks
- Contract storage is persistent and isolated between contracts

### SDK Security

- Private keys are never stored, logged, or transmitted beyond the configured RPC endpoint
- The SDK validates all inputs before submitting transactions
- Error handling wraps low-level errors in typed SDK errors
- No raw blockchain errors leak to consumers

### Frontend Security

- Wallet keys are managed by the wallet extension, not the application
- The frontend never has direct access to private keys
- All sensitive operations go through the wallet signing flow
- Transaction details are displayed for user confirmation before signing

## Supported Versions

| Component | Supported | Notes |
|-----------|-----------|-------|
| Smart Contracts | Latest | Soroban contracts on Stellar |
| SDK | Latest | TypeScript SDK |
| Frontend | Latest | Next.js web application |

## Reporting a Vulnerability

If you discover a security vulnerability in the TransferChain protocol, **do not report it publicly**.

### How to Report

1. **Do not** open a public GitHub issue for security vulnerabilities
2. **Do not** disclose the vulnerability on social media or public channels
3. **Email** the security team at the address provided in the repository's `SECURITY.md`
4. **Include** the following in your report:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact assessment
   - Suggested fix (if available)

### What to Expect

- **Acknowledgment** within 48 hours
- **Assessment** within 1 week
- **Resolution** timeline communicated upon assessment
- **Credit** in the security advisory (unless you prefer anonymity)

### Scope

In scope:

- Soroban smart contract vulnerabilities
- SDK security issues
- Frontend security issues
- Documentation that could lead to insecure usage

Out of scope:

- Denial of service attacks on infrastructure
- Social engineering
- Issues in third-party dependencies (report upstream)

## Security Best Practices

### For Developers

- Never commit private keys or secrets to version control
- Use environment variables for sensitive configuration
- Validate all inputs at contract boundaries
- Follow the principle of least privilege for role assignments
- Test with the latest versions of all dependencies

### For Users

- Always verify transaction details before signing
- Use hardware wallets for high-value operations
- Keep wallet software up to date
- Be cautious of phishing attempts

## Auditing

TransferChain is committed to security audits before mainnet deployment. Audit reports will be published in the repository.
