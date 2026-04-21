# AGENTS Improvement Spec

Audit of agent guidance quality for `eigenx-determinal-token-grants`.  
Date: 2026-04-21

---

## What's Good

- **README is thorough**: The integration guide covers all three endpoints, includes curl and TypeScript examples, and documents every request/response field.
- **Domain is narrow and well-scoped**: The grant-auth flow (get message → sign → call API) is simple enough that an agent can reason about it without ambiguity.
- **No conflicting conventions**: The repo is clean — no legacy patterns or mixed tooling to navigate.

---

## What's Missing

### 1. AGENTS.md (now created)
No agent guidance file existed. Agents had no structured entry point for understanding the project, its conventions, or its current state.

### 2. Runnable code
The README contains TypeScript examples but no actual source files, `package.json`, or install instructions. An agent asked to "run the example" has nothing to execute.

### 3. Dev environment setup
`devcontainer.json` uses the 10 GB universal image with no `postCreateCommand`, no installed tools, and no automation tasks. An agent starting the environment gets a blank shell with no project-specific tooling.

### 4. CI / test infrastructure
No test runner, linter, or CI pipeline. An agent cannot verify that code changes are correct.

### 5. Error catalogue
README mentions three error conditions (invalid address, no grant, insufficient tokens) but gives no HTTP status codes, error response shapes, or retry guidance.

### 6. Grant lifecycle documentation
No documentation on: how grants are issued, whether they expire, what happens when tokens run out mid-stream, or how to acquire a new grant.

### 7. Security guidance for agents
No explicit instruction that private keys must never appear in committed code, logs, or agent outputs.

### 8. Skill / rule files
No `.ona/skills/`, `.cursor/rules/`, or equivalent agent-rule files exist. Domain-specific agent behaviour is entirely undocumented.

---

## What's Wrong

### W1. README example uses hardcoded placeholder private key
```typescript
const PRIVATE_KEY = '0x...'; // Your private key
```
This pattern, if copied literally into a real file, risks accidental key exposure. The example should use `process.env.PRIVATE_KEY` and document that the variable must be set before running.

### W2. `seed` is hardcoded in all examples
Every example passes `"seed": 42`. This is fine for reproducibility demos but misleading — agents and users may not realise `seed` is optional and that omitting it produces varied outputs.

### W3. `model` field is undocumented beyond the example value
`gpt-oss-120b-f16` appears in examples but the README never lists available models, explains the naming scheme, or says where to find the current model list.

### W4. `/checkGrant` response shape is incomplete
The documented response includes `tokenCount`, `address`, and `hasGrant`, but the README never states the unit of `tokenCount` (tokens? credits? requests?) or what value triggers "insufficient tokens".

### W5. No versioning or stability signal
The API URL is a production endpoint with no version prefix (`/v1/`, `/v2/`). There is no indication of whether the API is stable, in beta, or subject to breaking changes.

---

## Improvement Spec

### Priority 1 — Immediate (unblock agents)

**1.1 Add `postCreateCommand` to devcontainer**

Install Node.js tooling so agents can run TypeScript examples without manual setup.

```json
"postCreateCommand": "npm install -g tsx viem"
```

Or add a `package.json` at the repo root with `viem` as a dependency and use:

```json
"postCreateCommand": "npm install"
```

**1.2 Fix private key handling in README examples**

Replace:
```typescript
const PRIVATE_KEY = '0x...'; // Your private key
```
With:
```typescript
const PRIVATE_KEY = process.env.PRIVATE_KEY as `0x${string}`;
if (!PRIVATE_KEY) throw new Error('PRIVATE_KEY env var required');
```

Add a note: "Set `PRIVATE_KEY` in your shell environment. Never commit a real private key."

**1.3 Add security rule to AGENTS.md**

Explicit instruction: agents must never write, log, or commit real private keys or Ethereum addresses. All examples must use environment variables or `0x...` placeholders.

---

### Priority 2 — Short-term (improve reliability)

**2.1 Document error response shapes**

Add a table to README:

| Condition | HTTP Status | Response shape |
|---|---|---|
| Missing/invalid `address` | 400 | `{ success: false, error: string }` |
| No active grant | 200 | `{ hasGrant: false, tokenCount: 0 }` |
| Insufficient tokens | 4xx | TBD — needs verification |
| Server error | 500 | TBD |

**2.2 Clarify `tokenCount` units**

State explicitly what one token represents (e.g., one output token, one request, one credit) and what the minimum required balance is to make a call.

**2.3 Document available models**

Either list known model identifiers or link to an endpoint that returns them. If the model list is dynamic, document a `/models` endpoint or equivalent.

**2.4 Mark `seed` and `temperature` as optional in parameter tables**

Update the parameter table to distinguish required vs optional fields.

---

### Priority 3 — Medium-term (agent productivity)

**3.1 Add a runnable TypeScript client**

Create `src/client.ts` (or `examples/grant-client.ts`) that implements the full flow. Include:
- `package.json` with `viem` dependency
- `tsconfig.json`
- Instructions in README: `npm install && npx tsx examples/grant-client.ts`

**3.2 Add grant lifecycle documentation**

Document:
- How grants are issued (who/what grants them, on-chain or off-chain)
- Whether grants expire (time-based or token-based)
- What to do when tokens are exhausted
- Whether a single grant message can be reused across multiple API calls

**3.3 Add API versioning note**

State whether the API is versioned, what the stability guarantee is, and where to watch for breaking changes.

**3.4 Add `.ona/skills/` or `.cursor/rules/` agent rules**

Create domain-specific agent rules covering:
- Always use `viem` for Ethereum signing (not `ethers`)
- Always read grant token balance before making expensive calls
- Never hardcode addresses or keys

**3.5 Add CI with a lint/type-check step**

Once source files exist, add a GitHub Actions workflow that runs `tsc --noEmit` and a linter (e.g., `eslint`) on every PR.

---

## File Checklist

| File | Status | Action |
|---|---|---|
| `AGENTS.md` | ✅ Done | Security rules section added |
| `README.md` | ✅ Done (W1) | Private key examples use `process.env` |
| `.devcontainer/devcontainer.json` | ✅ Done | `postCreateCommand` installs `tsx` + `viem` |
| `package.json` | ❌ Missing | Create with `viem` dependency |
| `src/` or `examples/` | ❌ Missing | Add runnable client |
| `.ona/skills/` | ❌ Missing | Add domain agent rules |
| `.github/workflows/` | ❌ Missing | Add CI after source exists |
