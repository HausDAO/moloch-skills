---
name: moloch-agent
description: Operate an autonomous DAOhaus/Moloch V3 agent with minimal setup using the HausDAO hosted moloch service for Graph reads and IPFS pinning. Use when the operator wants to provide only a DAO, mandate, and local signing wallet instead of managing Graph/Pinata credentials.
---

# Moloch Agent

Use this skill as the low-friction entry point for autonomous DAOhaus/Moloch agents on Base.

This skill assumes the hosted service handles Graph reads and IPFS pinning:

```bash
export MOLOCH_SERVICE_URL="${MOLOCH_SERVICE_URL:-https://moloch-service-production.up.railway.app}"
```

The service must never receive private keys. Signing stays local in the agent runtime.

Use the npm CLI for hosted-service operations:

```bash
npm install -g @raidguild/meta-clawtel
moloch-agent capabilities
```

## Minimal Operator Inputs

Resolve these from the harness/environment first. Ask the operator only for values that are missing and required for the immediate task.

Required only when acting on an existing DAO:

- DAO address.
- Agent mandate or mandate source.
- Signing capability: platform wallet skill, managed signer, or `PRIVATE_KEY`.

Required only when summoning a new DAO:

- DAO name.
- Token symbols.
- Initial members and initial share/loot balances.
- Governance settings not already supplied by a template.
- Agent mandate or mandate source.
- Signing capability.

Do not ask for agent voice, review mode, watch-only mode, no-action rules, or shared memory pointers during the first prompt unless the operator already made those part of the task. Default to autonomous operation. Discover or create shared memory pointers during bootstrap.

Do not ask the operator for:

- The Graph API key.
- Pinata JWT.
- DAOhaus subgraph id.
- Poster contract tags.

Use the hosted service for those dependencies.

## Skill Stack

This skill may be the only skill the agent is explicitly told to use. During bootstrap, inventory the harness/platform skills and tools that are available in the current agent environment, then prefer them where they are stronger than the generic fallback.

Expected stack:

- Moloch skill: this `moloch-agent` skill.
- Runtime CLI: `@raidguild/meta-clawtel`, exposed as `moloch-agent`.
- Platform wallet/account skill, if available: use it for managed signer access and account status.
- Platform Pinata/IPFS skill, if available: use it for larger artifact publishing and retrieval.
- Platform scheduler/task skill, if available: use it to register recurring DAO checks.
- Platform secrets skill, if available: use it for `PRIVATE_KEY`, managed wallet config, or managed RPC credentials.

Fallback behavior:

- If no platform Pinata/IPFS skill is visible, use `moloch-agent pin-json`.
- If no platform wallet skill is visible, use local `PRIVATE_KEY`.
- If no platform scheduler skill is visible, write the recommended task prompts from `AGENT_TASKS.md` for the operator or harness to register.
- If no managed RPC is visible, the CLI falls back to public Base RPC for light operation.

Record the detected platform capabilities in the bootstrap output before scheduling autonomous work.

## Runtime Assumptions

Primary CLI:

```text
moloch-agent
```

Install:

```bash
npm install -g @raidguild/meta-clawtel
```

The npm CLI handles hosted service reads, pinning, proposal lifecycle checks, summon transaction building, proposal transaction building, lifecycle actions, and local signing for core actions. Use the shared runtime script only as an advanced fallback for commands not yet exposed by `moloch-agent`.

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

If an advanced fallback command is needed, check the local transaction runtime:

```bash
node /data/custom/moloch-skills/moloch-shared/scripts/moloch.mjs capabilities
```

If using local signing, require:

```bash
export PRIVATE_KEY=0x...
```

`RPC_URL` defaults to the public Base RPC (`https://mainnet.base.org`) so the CLI works without extra setup. Prefer setting a managed Base RPC URL for real scheduled operation because the public endpoint can rate limit.

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
2. Detect platform skills and local CLI/runtime capabilities.
3. Detect signer/account status from platform wallet skill, `ACCOUNT_ADDRESS`, `PRIVATE_KEY`, or `moloch-agent account`.
4. Load the operator-provided mandate or mandate source.
5. Discover existing shared memory pointers from DAO metadata, or create starter pointers when summoning.
6. Run a task snapshot once a DAO exists.
7. Configure scheduled tasks when a scheduler is available.
8. Report only hard blockers.

Do not invent the mandate. If the operator has not provided a mandate, create a local draft with missing fields and continue only with read/setup work until the mandate exists. Do not ask for watch-only or review-only mode unless the operator requests dry-run operation.

## Summon

For a new DAO, create a small params file and summon through the npm CLI:

```bash
moloch-agent summon --params summon.json
```

The summon params should include initial members, raw initial share/loot base-unit balances, token names, voting/grace periods, quorum, sponsor threshold, and min retention. If DAO metadata pointers such as `communityMemoryURI`, `proposalWorkspaceURI`, or `sharedStateURI` are omitted, the CLI creates and pins a starter DAO workspace and includes its `ipfs://...` URI in summon metadata.

Use whole-number percentages for `quorum` and `minRetention`. Use 18-decimal base units for shares, loot, offering, and sponsor threshold when the value represents Baal token units.

Address rule: never expand shortened address previews such as `0x1234...abcd`. Use only full `0x` 40-hex-character addresses from `moloch-agent account`, environment variables, wallet output, chain/Graph reads, checked JSON files, or explicit full user input. For founder/member/payment/mint params, fetch the exact address again instead of reconstructing it from a preview.

Before summon, run:

```bash
moloch-agent account
```

If the signer should be an initial member, copy the full returned `address` exactly into `memberAddresses`.

## Hosted Service Helpers

Read DAO state through the npm CLI:

```bash
moloch-agent dao --dao 0xDAO
moloch-agent daohaus-url --dao 0xDAO
moloch-agent links --dao 0xDAO --proposal 12
moloch-agent read-dao --dao 0xDAO
moloch-agent balances --dao 0xDAO
moloch-agent balances --address 0xADDRESS --token 0xERC20
moloch-agent proposals --dao 0xDAO
moloch-agent proposal-lifecycle --dao 0xDAO --proposal 12
moloch-agent process-queue --dao 0xDAO
moloch-agent members --dao 0xDAO
moloch-agent records --dao 0xDAO --table communityMemory
```

`daohaus-url` returns the DAOhaus Admin proposals URL, for example:

```text
https://admin.daohaus.club/molochv3/0x2105/0xDAO/proposals
```

Use `links` when BaseScan address/code or transaction URLs are also useful.

Pin JSON artifacts through the npm CLI:

```bash
moloch-agent pin-json --file community-state.json --name community-state-v1
moloch-agent workspace-create --kind dao --dao 0xDAO --title "DAO Workspace"
moloch-agent workspace-create --kind proposal --dao 0xDAO --title "Proposal Workspace"
```

Use returned `ipfs://...` values in:

- `contentURI`
- `workspaceURI`
- `stateURI`
- DAO metadata pointers

Update DAO metadata pointers through a governance proposal:

```bash
moloch-agent dao-meta \
  --dao 0xDAO \
  --title "Update DAO memory pointers" \
  --community-memory-uri ipfs://... \
  --proposal-workspace-uri ipfs://...
```

This writes DAOhaus-compatible metadata through Poster so indexed clients can discover shared memory pointers.

## Autonomous Task Loop

Run these task types from `AGENT_TASKS.md`:

1. Proposal Action Watcher
2. Initiative Steward
3. Proposal Generation

Default behavior:

- Broadcast when mandate and live preflight point to action.
- Transaction commands broadcast by default.
- Use `--build-only` only when a task explicitly needs an unsigned transaction artifact.
- Do not wait for operator approval.
- Process ready proposals as mechanical settlement.
- Post concise DAO database memory records after meaningful actions.
- Keep proposal/action output compact. Do not print full calldata, ABI fragments, or raw Graph JSON unless asked.

## Proposal Creation Rules

When creating proposals:

1. Read current DAO state and memory.
2. Check the mandate and initiative backlog.
3. Do not create a new proposal if 3 or more proposals are currently in voting.
4. Let the CLI create and pin a proposal workspace automatically. Do not pass `--link` or `--content-uri` unless it is already an IPFS workspace URI for this proposal.
5. Use `ipfs://...` workspace links by default. Set `IPFS_GATEWAY_URL` only when gateway URLs should be used as proposal links.
6. Keep proposal workspaces small and versioned; IPFS artifacts are immutable.
7. Use the correct proposal path:
   - text-only intent: `signal`
   - token tribute / join / swap: `tribute`, `join-dao`, `swap`, or `token-swap`
   - direct share grant: `mint-shares`
   - direct non-voting loot grant: `mint-loot`
   - treasury ETH/ERC-20 payment: `payment`
   - governance settings: `gov-settings`
   - token pause/transfer settings: `token-settings`
   - mapped action not yet first-class: `custom-proposal`
8. Broadcast by default if preflight passes.

Do not use `signal` for a real membership, shares, loot, tribute, token swap, or treasury payment action.

Common proposal commands:

```bash
moloch-agent signal --dao 0xDAO --title "..." --description "..."
moloch-agent gov-settings --dao 0xDAO --params gov-settings.json
moloch-agent token-settings --dao 0xDAO --pause-shares false --pause-loot false
moloch-agent custom-proposal --dao 0xDAO --title "Custom action" --actions actions.json
moloch-agent wrap-eth --amount 0.01
moloch-agent approve-token --token 0x4200000000000000000000000000000000000006 --amount 0.01
moloch-agent treasury-tokens --dao 0xDAO
moloch-agent ragequit --dao 0xDAO --to 0xRECIPIENT --shares 1 --loot 0 --tokens ETH --confirm-ragequit
moloch-agent join-dao --dao 0xDAO --token 0xERC20 --amount 1000000 --shares 10000
moloch-agent tribute --dao 0xDAO --token 0xERC20 --amount 1000000 --shares 10000
moloch-agent swap --dao 0xDAO --token 0xERC20 --amount 1000000 --shares 0 --loot 100
moloch-agent payment --dao 0xDAO --recipient 0xPAYEE --amount 0.01
moloch-agent payment --dao 0xDAO --recipient 0xPAYEE --token 0xERC20 --amount 100 --decimals 6
moloch-agent mint-shares --dao 0xDAO --to 0xMEMBER --amount 1
moloch-agent mint-loot --dao 0xDAO --to 0xMEMBER --amount 100
```

In normal operation, omit `--link` on proposal commands. The CLI will pin a proposal workspace and set proposal details `contentURI` automatically.

For `mint-shares` and `mint-loot`, `--amount 1` means 1 full DAO token and encodes as `1000000000000000000`. Use `--amount-raw` only when the exact base-unit value is intended.

For treasury `payment`, native ETH uses decimal ETH in `--amount`. ERC-20 payments require `--amount-raw` or `--decimals` because token decimals vary.

## DAO Database Memory

Use `memory-post` for public coordination records:

```bash
moloch-agent memory-post \
  --dao 0xDAO \
  --type vote-reason \
  --thread-id proposal-12 \
  --proposal 12 \
  --body "Reason for vote."
```

Use the `community-memory/v1` envelope. Prefer `threadId` for grouping.

For votes, prefer the combined vote command with a reason:

```bash
moloch-agent vote \
  --dao 0xDAO \
  --proposal 12 \
  --approved false \
  --reason "I voted no because the proposal needs clearer deliverables."
```

This posts a `vote-reason` memory record linked to the proposal workspace, then submits the vote.

## Processing Rule

Processing is not a subjective mandate decision.

When `process-queue` identifies the oldest ready proposal and live chain preflight passes:

- use exact indexed `proposalData`
- process oldest ready proposal first
- do not block because of proposal category, value, shares, loot, membership, payments, or settings
- reread state after processing
- post a concise result record

Preferred command:

```bash
moloch-agent process-ready --dao 0xDAO
```

`process-ready` uses the queue helper, selects the oldest ready proposal, and includes a gas limit derived from proposal `baalGas` when available. If a proposal was submitted with `baalGas` too low, the process transaction can still need a larger outer transaction gas limit.

## References

Use these repo docs as supporting references:

- `BOOTSTRAP.md`
- `AGENT_TASKS.md`
- `MEMORY_LAYER.md`
- `VOTE_DECISION_FLOW.md`
- `PRISM.md`

For advanced commands and troubleshooting, use:

- `moloch-shared`
- `moloch-dao-read`
- `moloch-proposals`
- `moloch-proposal-actions`
- `moloch-summon`
- `moloch-agent-conviction`
