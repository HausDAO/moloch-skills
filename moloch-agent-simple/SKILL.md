---
name: moloch-agent-simple
description: Operate an autonomous DAOhaus/Moloch V3 agent with minimal setup using the HausDAO hosted moloch service for Graph reads and IPFS pinning. Use when the operator wants to provide only a DAO, mandate, and local signing wallet instead of managing Graph/Pinata credentials.
---

# Moloch Agent Simple

Use this skill as the low-friction entry point for autonomous DAOhaus/Moloch agents on Base.

This skill assumes the hosted service handles Graph reads and IPFS pinning:

```bash
export MOLOCH_SERVICE_URL="${MOLOCH_SERVICE_URL:-https://moloch-service-production.up.railway.app}"
```

The service must never receive private keys. Signing stays local in the agent runtime.

Use the npm CLI for hosted-service operations:

```bash
npm install -g @raidguild/moloch-agent
moloch-agent capabilities
```

## Minimal Operator Inputs

Ask the operator or harness for:

- DAO address, or explicit instruction to summon a new DAO.
- Agent name.
- Agent wallet address.
- Local signing method, usually `PRIVATE_KEY` or a managed wallet integration.
- Agent mandate: values, voting policy, initiative goals, and autonomous action rules.

Do not ask the operator for:

- The Graph API key.
- Pinata JWT.
- DAOhaus subgraph id.
- Poster contract tags.

Use the hosted service for those dependencies.

## Runtime Assumptions

Primary CLI:

```text
moloch-agent
```

Install:

```bash
npm install -g @raidguild/moloch-agent
```

The npm CLI currently handles hosted service reads and pinning. Use the shared runtime script for transaction builders/actions that are not yet in the npm package.

Preferred runtime asset path:

```text
/data/custom/moloch-skills
```

Shared CLI:

```text
/data/custom/moloch-skills/moloch-shared/scripts/moloch.mjs
```

If the CLI is missing, install runtime assets from:

```text
https://github.com/HausDAO/moloch-skills
```

For Prism, use the Prism-managed install flow from `PRISM.md`; do not install only into `CODEX_HOME`.

## Dependency Check

First, check the hosted service:

```bash
moloch-agent health
moloch-agent capabilities
```

Expected capabilities:

- `graph.configured: true`
- `pinning.configured: true`
- `signing.handledByService: false`

Then check local transaction runtime when write actions are needed:

```bash
node /data/custom/moloch-skills/moloch-shared/scripts/moloch.mjs capabilities
```

If using local signing, require:

```bash
export PRIVATE_KEY=0x...
```

If direct contract reads or sends need a local RPC, use the default public Base RPC only for light tests. Prefer a managed RPC in real scheduled operation.

## Source Authority

Use layered sources of truth:

- Direct chain/RPC reads: execution truth, proposal lifecycle, voting/process preflight.
- Hosted moloch service: Graph discovery and Pinata uploads.
- DAO database records: shared communication and memory events.
- IPFS CIDs: larger/versioned artifacts and proposal workspaces.
- Local files: scratch, checkpoints, and task continuity only.

Graph data can lag. Never use hosted Graph data alone to decide that a transaction is executable. Re-read chain state before write actions.

## Bootstrap

Use `BOOTSTRAP.md` as the generic first-run flow.

Bootstrap should:

1. Confirm DAO address or summon intent.
2. Confirm local signer and agent wallet.
3. Load or create the operator-provided mandate.
4. Discover or create shared memory pointers.
5. Run a task snapshot.
6. Configure scheduled tasks.
7. Report missing fields or blocked capabilities.

Do not invent the mandate. If the operator has not provided a mandate, create a local draft and ask for the missing mandate fields.

## Hosted Service Helpers

Read DAO state through the npm CLI:

```bash
moloch-agent dao --dao 0xDAO
moloch-agent proposals --dao 0xDAO
moloch-agent members --dao 0xDAO
moloch-agent records --dao 0xDAO --table communityMemory
```

Pin JSON artifacts through the npm CLI:

```bash
moloch-agent pin-json --file community-state.json --name community-state-v1
```

Use returned `ipfs://...` values in:

- `contentURI`
- `workspaceURI`
- `stateURI`
- DAO metadata pointers

## Autonomous Task Loop

Run these task types from `AGENT_TASKS.md`:

1. Proposal Action Watcher
2. Initiative Steward
3. Proposal Generation

Default behavior:

- Broadcast with `--send` when mandate and live preflight point to action.
- Do not wait for operator approval.
- Process ready proposals as mechanical settlement.
- Post concise DAO database memory records after meaningful actions.
- Keep proposal/action output compact. Do not print full calldata, ABI fragments, or raw Graph JSON unless asked.

## Proposal Creation Rules

When creating proposals:

1. Read current DAO state and memory.
2. Check the mandate and initiative backlog.
3. Do not create a new proposal if 3 or more proposals are currently in voting.
4. Create or reuse a proposal workspace.
5. Pin the workspace or key artifact through the hosted service.
6. Put the workspace URI in proposal `contentURI` when useful.
7. Use the correct proposal path:
   - text-only intent: `signal`
   - token tribute / join: `tribute` or `join-dao`
   - direct share grant: `mint-shares`
   - governance settings: `gov-settings`
   - token pause/transfer settings: `token-settings`
8. Broadcast by default if preflight passes.

Do not use `signal` for a real membership, shares, loot, or tribute action.

## DAO Database Memory

Use `memory-post` for public coordination records:

```bash
node /data/custom/moloch-skills/moloch-shared/scripts/moloch.mjs memory-post \
  --dao 0xDAO \
  --table communityMemory \
  --type vote-reason \
  --thread-id proposal-12 \
  --proposal 12 \
  --body "Reason for vote." \
  --send
```

Use the `community-memory/v1` envelope. Prefer `threadId` for grouping.

## Processing Rule

Processing is not a subjective mandate decision.

When `process-queue` identifies the oldest ready proposal and live chain preflight passes:

- use exact indexed `proposalData`
- process oldest ready proposal first
- do not block because of proposal category, value, shares, loot, membership, payments, or settings
- reread state after processing
- post a concise result record

## References

Use these repo docs as supporting references:

- `BOOTSTRAP.md`
- `AGENT_TASKS.md`
- `SHARED_MEMORY.md`
- `VOTE_DECISION_FLOW.md`
- `PRISM.md`

For advanced commands and troubleshooting, use:

- `moloch-shared`
- `moloch-dao-read`
- `moloch-proposals`
- `moloch-proposal-actions`
- `moloch-summon`
- `moloch-agent-conviction`
