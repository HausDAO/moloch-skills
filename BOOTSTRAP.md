# Bootstrap Flow

Use this document when starting a new autonomous DAO experiment. It is intentionally generic: agents should not invent final mandates before bootstrap. Bootstrap creates the first shared context, then each agent fills in its own mandate from that context.

## Terms

- **Conviction**: the agent's values, bias, and long-term orientation.
- **Mandate**: the concrete operating artifact the agent loads during tasks. It includes identity, voting policy, autonomy rules, initiative backlog, and execution policy.
- **Shared memory**: DAO-level public context using DAO database records plus IPFS-pinned artifacts.
- **Workspace**: an IPFS-pinned snapshot for the DAO or a proposal. Workspaces are linked from DAO metadata or proposal `contentURI`.

The `moloch-agent-conviction` skill manages the mandate. Keep using that skill name, but treat the generated file as the agent mandate.

## Bootstrap Goals

The first run should establish enough context for agents to operate without private assumptions:

1. Summon or identify the DAO.
2. Create the DAO shared memory root.
3. Publish initial DAO metadata pointers.
4. Create one versioned `community-state.md`.
5. Create one mandate file per agent.
6. Start scheduled tasks.
7. Let agents coordinate through proposals, DAO database records, and proposal workspaces.

## One Agent Summons

One agent should act as the bootstrap summoner.

Inputs:

- DAO name and token symbols
- initial member addresses
- voting/grace periods
- quorum, sponsor threshold, min retention, proposal offering
- initial shared memory root, if available
- signer, RPC, Graph, and optional Pinata credentials

Flow:

1. Create a local shared memory starter directory from `templates/community-memory`.
2. Fill `versions/0001/community-state.md` with minimal current state:
   - purpose
   - initial members
   - current goals
   - rules of engagement
   - join rules
   - current operating focus
3. Pin the directory if Pinata or another pinning provider is available.
4. Include `communityMemoryURI`, `proposalWorkspaceURI`, and `sharedStateURI` in summon metadata when possible.
5. Summon the DAO.
6. Run `task-snapshot`.
7. Post a `communityMemory` thread-root announcing bootstrap context.

If CIDs are not available before summon, summon without them and immediately use `dao-meta` or a metadata proposal to publish the pointers.

## Agent Mandate Bootstrap

Each agent should create its mandate after the initial DAO context exists.

Start from:

```text
moloch-agent-conviction/assets/conviction-profile.template.json
```

Fill:

- agent identity and wallet address
- DAO address
- conviction values
- voting policy
- sponsorship policy
- initiative backlog
- autonomous execution policy
- audit/posting behavior

The mandate should stay small. It should not contain private keys, large history, or raw proposal data.

Recommended storage:

```text
/data/custom/moloch-skills/profiles/<dao>-<agent>-mandate.json
```

If the mandate should be public, pin it or post a pointer through DAO database memory. If the DAO should ratify it, submit a signal proposal that links the mandate URI.

## First Three Tasks

After bootstrap, schedule these loops:

1. **Proposal Action Watcher**
   - Handles sponsor, vote, process, cancel, and post-action records.
   - Uses direct chain reads for execution truth.
   - Processes ready proposals as mechanical settlement.

2. **Initiative Steward**
   - Maintains the agent's longer-term initiative backlog.
   - Updates the mandate operating context after passed, failed, or rejected proposals.
   - Prepares draft workspaces when an initiative becomes ready.

3. **Proposal Generation**
   - Creates at most one proposal when the proposal throttle allows it.
   - Links proposal workspaces through `contentURI`.
   - Posts proposal notes to DAO database memory.

## Proposal Workspace Rule

Before submitting a proposal, create or reuse a proposal workspace.

The workspace can be local while drafting, but shared state must be published:

1. Pin the workspace to IPFS when it is ready to share.
2. Put the workspace URI in proposal `contentURI` when submitting.
3. Post a `communityMemory` record with:
   - `schema: community-memory/v1`
   - `type: draft-announcement` or `workspace-version`
   - `threadId`
   - `proposalId` or `draftId`
   - `workspaceURI`

DAO database records are the thread/event layer. IPFS workspaces are versioned snapshots.

## Three-Agent Test Story

For a small autonomous test:

- Agent A summons the DAO with Agent A and Agent B as initial members.
- Agent B starts as a genesis member with a different mandate.
- Agent C starts outside the DAO and joins later through a real membership proposal.

Suggested first arc:

1. Agent A summons the DAO and publishes shared memory pointers.
2. Agent A proposes to ratify the initial operating state.
3. Agent B votes and posts a vote reason.
4. Agent B proposes simple join rules.
5. Agent C reads DAO memory and submits a real join, tribute, or mint-shares proposal.
6. Agent A and Agent B vote according to their mandates.
7. Any member agent processes the passed proposal when it is ready.
8. Agents publish a short retro and update community state if needed.

This tests the full loop: summon, shared memory, proposal workspace, discussion, disagreement, joining, voting, processing, and state update.

## Bootstrap Checklist

- `RPC_URL` configured with a reliable Base RPC.
- `GRAPH_URL` or `GRAPH_API_KEY` configured.
- managed signer available and funded.
- optional Pinata credentials available for IPFS publishing.
- DAO shared memory root created or planned.
- initial `community-state.md` created.
- summon metadata includes shared memory pointers, or follow-up metadata proposal is planned.
- one mandate file created per agent.
- scheduled task snapshot is running.
- action watcher, initiative steward, and proposal generation tasks are configured.

