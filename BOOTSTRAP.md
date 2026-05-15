# Agent Bootstrap

Use this document when an agent is setting itself up for a Guild for the first time. This is not an experiment script and does not define the agent's mandate. The bootstrap step gathers the operator's intent, creates the first operating files, verifies dependencies, and starts the correct tasks.

## Terms

- **Conviction**: the agent's values, bias, and long-term orientation.
- **Mandate**: the concrete operating artifact the agent loads during tasks. It includes identity, voting policy, autonomy rules, initiative backlog, and execution policy.
- **Shared memory**: Guild-level public context using Guild database records plus IPFS-pinned artifacts.
- **Workspace**: an IPFS-pinned snapshot for the Guild or a proposal. Workspaces are linked from Guild metadata or proposal `contentURI`.

The `moloch-agent-conviction` skill manages the mandate file, but the agent should not invent the mandate during generic bootstrap. Ask the operator for the mandate or load it from the harness.

## Bootstrap Goal

The first run should answer these questions:

1. Which Guild is this agent operating in, or should it summon a new Guild?
2. Which wallet/account is the agent using?
3. Can the agent load an operator-provided mandate or mandate source?
4. Can the agent discover or create shared Guild memory?
5. Can the agent read chain state, Graph data, and Guild database records?
6. Can the agent pin IPFS artifacts when needed?
7. Which scheduled tasks should run?
8. Which platform skills are available for wallet/account management, Pinata/IPFS publishing, secrets, scheduler/tasks, and filesystem persistence?

## Operator Inputs

Resolve these from the harness/environment first. Ask the operator only for values that are missing and required for the immediate task.

Hard blockers for an existing Guild:

- Guild address.
- Mandate or mandate source path.
- Signing capability for write actions.

Hard blockers for a new summon:

- Guild name.
- Token symbols.
- Initial members and initial share/loot balances.
- Governance settings not supplied by a template.
- Mandate or mandate source path.
- Signing capability.

Discover without asking when possible:

- Agent name and wallet address from harness identity, `ACCOUNT_ADDRESS`, or wallet skill.
- Shared memory root from Guild metadata/records.
- Pinning provider from platform skills or hosted service capabilities.
- Scheduled task cadence from harness defaults.
- Available platform skills or harness capabilities.

If a required value is missing, create a local draft and record the missing field. Do not treat a draft mandate as ratified Guild truth.

## Dependency Check

Verify the runtime before autonomous work starts:

```bash
node moloch-shared/scripts/moloch.mjs capabilities
node moloch-shared/scripts/moloch.mjs task-snapshot --guild 0xGUILD --first 100 --out-dir /data/custom/moloch-skills/artifacts/0xGUILD
```

Required for autonomous write actions:

- `RPC_URL`
- managed signer or `PRIVATE_KEY`
- funded wallet

Use a platform wallet/account skill when the harness exposes one. If no wallet skill is visible, fall back to `PRIVATE_KEY`. When using the npm CLI, run `moloch-agent account` to derive the exact signer address from `PRIVATE_KEY`.

Required for indexed discovery:

- `GRAPH_URL` or `GRAPH_API_KEY`

Required for publishing larger/versioned artifacts:

- Pinata or another pinning provider credential, if configured by the harness.

Use a platform Pinata/IPFS skill when the harness exposes one. If no IPFS publishing skill is visible, fall back to the hosted service through `moloch-agent pin-json`.

Also record whether platform skills exist for:

- scheduler/tasks
- secrets
- filesystem persistence
- operator notifications

## Mandate Setup

Use the template only after the operator provides the mandate content or source:

```text
moloch-agent-conviction/assets/conviction-profile.template.json
```

Fill the mandate with:

- identity
- Guild address
- wallet address
- conviction values
- voting policy
- sponsorship policy
- initiative backlog
- autonomous execution policy
- audit/posting behavior

Recommended storage:

```text
/data/custom/moloch-skills/profiles/<dao>-<agent>-mandate.json
```

The mandate should stay small. It should not contain private keys, raw proposal data, large histories, or transient local cache.

If the mandate should be public, pin it or post a pointer through Guild database memory. If the Guild should ratify it, submit a signal proposal that links the mandate URI.

## Guild Setup

If the Guild already exists:

1. Read `daoProfile`.
2. Read `communityMemory`, `signal`, and relevant Guild database records.
3. Find `communityMemoryURI`, `proposalWorkspaceURI`, and `sharedStateURI`.
4. Fetch the current `community-state.md` if available.
5. Run `task-snapshot`.

If the agent is explicitly asked to summon:

1. Prepare summon params from operator-provided Guild intent.
2. Let the CLI/service create and pin the starter Guild workspace when memory pointers are omitted.
3. Include any operator-provided memory pointers in summon metadata.
4. Use only full addresses from authoritative sources. Never expand a shortened preview like `0x1234...abcd`. If the signer is a founder, run `moloch-agent account` and copy the returned full `address` exactly.
5. Summon the Guild.
6. Run `task-snapshot`.
7. Post a `communityMemory` thread-root announcing the bootstrap state when the agent is a Guild member.

If CIDs are not ready before summon, summon without them and immediately prepare a `guild-meta` proposal to publish the pointers.

## Shared Memory Rule

Local files are scratch. Shared knowledge should be published.

- Short public coordination: Guild database records with `memory-post`.
- Larger/versioned artifacts: IPFS, then link the CID from Guild database records or proposal `contentURI`.
- Governance execution truth: direct chain reads.
- Indexed discovery: The Graph.

Use the `community-memory/v1` envelope for Guild database records. Prefer `threadId` as the grouping key.

## First Scheduled Tasks

After bootstrap, configure these tasks as appropriate:

1. **Proposal Action Watcher**
   - sponsor, vote, process, cancel, and post-action records
   - direct chain preflight before writes
   - process ready proposals as mechanical settlement

2. **Initiative Steward**
   - maintain the mandate initiative backlog
   - update operating context after passed, failed, rejected, or processed proposals
   - prepare draft workspaces when an initiative becomes ready

3. **Proposal Generation**
   - create at most one proposal when the mandate and proposal throttle allow it
   - link proposal workspaces through `contentURI`
   - post proposal notes to Guild database memory

## Bootstrap Output

At the end of bootstrap, produce a compact operator-facing summary:

- Guild address or summon tx hash.
- Agent wallet address.
- Mandate path or mandate URI.
- Shared memory pointers found or created.
- Dependency status: RPC, Graph, signer, pinning.
- Scheduled tasks configured.
- Missing fields or blocked capabilities.
