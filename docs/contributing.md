# Contributing to TransferChain

> Guidelines for contributing to the TransferChain open-source protocol.

## Welcome

TransferChain is an open-source project and we welcome contributions from the community. Whether you're fixing a bug, adding a feature, improving documentation, or reporting an issue, your contribution is valuable.

## Getting Started

### 1. Fork and Clone

```bash
git clone https://github.com/<your-fork>/TransferChain.git
cd TransferChain
```

### 2. Set Up Development Environment

Follow the [Development Guide](./development.md) to set up your local environment.

### 3. Find Something to Work On

- Browse [GitHub Issues](https://github.com/transferchain/TransferChain/issues) for open tasks
- Look for `good-first-issue` labels for beginner-friendly tasks
- Check the [Roadmap](../README.md#roadmap) for planned features

## Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Production-ready code |
| `develop` | Active development target for PRs |
| `feature/<name>` | New features |
| `fix/<name>` | Bug fixes |
| `docs/<name>` | Documentation changes |

## Pull Request Process

1. Create a feature branch from `develop`
2. Make your changes
3. Ensure all checks pass
4. Submit a PR against `develop`
5. Request review from a maintainer
6. Address review feedback
7. Merge after approval

### PR Requirements

- [ ] All tests pass
- [ ] Code follows project conventions
- [ ] New code has tests
- [ ] Public API has documentation
- [ ] No secrets or private keys in code
- [ ] PR description explains the change

## Commit Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation change |
| `refactor` | Code refactoring |
| `test` | Test change |
| `chore` | Tooling or dependency change |
| `perf` | Performance improvement |

### Scopes

| Scope | Description |
|-------|-------------|
| `contracts` | Soroban smart contracts |
| `sdk` | TypeScript SDK |
| `frontend` | Web application |
| `docs` | Documentation |

### Examples

```
feat(contracts): add player eligibility check
fix(sdk): handle unknown event topics gracefully
docs(api): update TransactionResult examples
test(integration): add escrow refund scenario
```

## Code Style

### Rust (Soroban Contracts)

- Follow the [Rust API guidelines](https://rust-lang.github.io/api-guidelines/)
- Use `clippy` for linting
- Use `rustfmt` for formatting
- Write doc comments on all public items

### TypeScript (SDK)

- Strict mode enabled
- No `any` types in source code
- Use `interface` for object shapes, `type` for unions
- JSDoc on all public methods
- Run `pnpm lint` and `pnpm format` before committing

### General

- Write tests for all new functionality
- Keep PRs focused and small
- Prefer editing existing files over creating new ones
- Follow existing patterns and conventions

## Architecture Decision Records

Significant architectural decisions are recorded as ADRs in `TransferChain-SDK/docs/adr/`. If your change involves a significant architectural decision, please create an ADR.

## Review Checklist

For reviewers:

- [ ] Code is correct and follows conventions
- [ ] Tests cover the new functionality
- [ ] Documentation is updated if needed
- [ ] No breaking changes without discussion
- [ ] Security implications are considered
- [ ] Performance impact is acceptable

## Community

- [GitHub Issues](https://github.com/transferchain/TransferChain/issues) — Bug reports and feature requests
- [GitHub Discussions](https://github.com/transferchain/TransferChain/discussions) — Questions and ideas

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
