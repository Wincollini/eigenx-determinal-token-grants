# AGENTS.md

Agent guidance for the `eigenx-determinal-token-grants` repository.

## Project Overview

This repository documents and demonstrates the **deTERMinal token grant API** — a grant-based authentication system for the EigenArcade chat completions API. Users authenticate via Ethereum wallet signatures (EOA) instead of API keys.

**API base URL:** `https://determinal-api.eigenarcade.com`

## Repository Structure

```
.
├── README.md                  # Integration guide (primary documentation)
├── AGENTS.md                  # This file
└── .devcontainer/
    └── devcontainer.json      # Dev container config (universal image)
```

No source code exists yet. The repo currently contains only documentation.

## Key Domain Concepts

- **Grant**: A token allocation tied to an Ethereum address, consumed when making API calls.
- **Grant message**: A server-issued string the user signs to prove wallet ownership.
- **Grant signature**: The EIP-191 personal signature over the grant message.
- **EOA**: Externally Owned Account — a standard Ethereum wallet address.

## API Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/message` | GET | Fetch a message to sign; param: `address` |
| `/api/chat/completions` | POST | Make a chat completion using grant auth |
| `/checkGrant` | GET | Check token balance; param: `address` |

### Auth fields on `/api/chat/completions`

```json
{
  "grantMessage":   "<string from /message>",
  "grantSignature": "<EIP-191 signature>",
  "walletAddress":  "<0x...>",
  "messages":       [...],
  "model":          "gpt-oss-120b-f16",
  "max_tokens":     150,
  "seed":           42
}
```

## Development Environment

The devcontainer uses `mcr.microsoft.com/devcontainers/universal:4.0.1-noble`. No build steps or install commands are defined yet.

If adding TypeScript examples, the expected dependency is `viem` for wallet signing.

## Security Rules

These rules are non-negotiable. Violating them risks irreversible loss of user funds.

- **Never write, log, commit, or output real private keys or seed phrases** — in code, comments, test fixtures, or agent responses.
- **Never hardcode Ethereum addresses** that belong to real wallets. Use `0x...` placeholders in examples.
- **All code examples must read secrets from environment variables** (`process.env.PRIVATE_KEY`, etc.), not from literals.
- If asked to handle a private key as part of a task, refuse and instruct the user to set it as an environment variable instead.

## Agent Conventions

- **No source code yet**: Do not assume a build system, test runner, or package manager exists.
- **README is the source of truth** for API behaviour until a backend implementation is added.
- When adding code, prefer TypeScript with `viem` (matches existing README examples). Run with `tsx`.
- Commit messages: imperative mood, lowercase, e.g. `add typescript client example`.

## What's Missing (known gaps)

- No source code, tests, or runnable examples
- No package.json / dependency manifest
- No CI/CD configuration
- No error-handling documentation beyond HTTP status codes
- No token expiry or grant lifecycle documentation
