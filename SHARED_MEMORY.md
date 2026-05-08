# Shared Community Memory

This repo supports always-on DAO agents, but the DAO's durable memory should not live only inside one agent workspace. Use an IPFS-first shared memory root for community state, proposal collaboration, and agent-to-agent coordination.

## Model

Create a shared memory directory before or during summon, pin it to IPFS, and include the root CID in DAO summon metadata:

```json
{
  "communityMemoryURI": "ipfs://...",
  "proposalWorkspaceURI": "ipfs://.../proposals",
  "sharedStateURI": "ipfs://.../state/current"
}
```

The root can be a mutable folder in Pinata or another pinning workflow, with versioned snapshots pinned as immutable CIDs. Agents should treat the latest DAO metadata pointer as the canonical entrypoint, then write new versioned folders and propose pointer updates when shared state changes.

## Directory Layout

Recommended root layout:

```text
community-memory/
  README.md
  manifest.json
  state/
    current/
      manifesto.md
      charter.md
      goals.md
      intent.md
      roles.md
      join-rules.md
      operating-context.json
    versions/
      0001/
        manifest.json
        manifesto.md
        charter.md
        goals.md
        intent.md
        roles.md
        join-rules.md
  proposals/
    drafts/
      proposal-<local-id-or-title>/
        proposal.md
        details.json
        actions.json
        discussions.md
        negotiations.md
        action-items.md
        vote-reasons.md
        sources.md
        status.json
    onchain/
      proposal-<id>/
        proposal.md
        details.json
        actions.json
        discussions.md
        negotiations.md
        action-items.md
        vote-reasons.md
        txs.json
        final-state.json
  agents/
    <agent-name>/
      mandate.json
      public-notes.md
      action-log.jsonl
  discussions/
    <topic-slug>/
      README.md
      notes.md
      decisions.md
```

## Root Manifest

Use `manifest.json` to make the memory root machine-readable:

```json
{
  "type": "dao-community-memory",
  "version": "0.1.0",
  "daoName": "",
  "daoAddress": "",
  "chainId": 8453,
  "createdAt": "",
  "currentStatePath": "state/current",
  "proposalDraftsPath": "proposals/drafts",
  "onchainProposalsPath": "proposals/onchain",
  "agentsPath": "agents",
  "discussionsPath": "discussions",
  "latestStateVersion": "0001"
}
```

## Proposal Workspace Rule

Before creating an onchain proposal, an agent should create or reuse a proposal workspace folder under `proposals/drafts/`.

Minimum files:

- `proposal.md`: human-readable title, summary, motivation, expected outcome, and success criteria.
- `details.json`: DAOhaus proposal details JSON or the planned fields used to build it.
- `actions.json`: concise action summary, target contracts, value, encoded action kind, and whether it is signal/executable.
- `discussions.md`: arguments, questions, objections, and links.
- `negotiations.md`: concessions, counterproposals, and changed terms.
- `action-items.md`: follow-up tasks and owners.
- `vote-reasons.md`: each agent/member's vote reason when known.
- `status.json`: draft, submitted, sponsored, voting, grace, processed, failed, superseded.

After submission, copy or move the folder to `proposals/onchain/proposal-<id>/` and add:

- `txs.json`: submission, sponsorship, vote, and processing tx hashes.
- `final-state.json`: final lifecycle status after processing.

## Rolling Community State

The `state/current/` folder should contain the current community context that agents use before proposing or voting:

- `manifesto.md`: why the DAO exists.
- `charter.md`: rules of engagement and governance norms.
- `goals.md`: current strategic goals.
- `intent.md`: current operating intent and near-term focus.
- `roles.md`: members, agents, stewards, and operational roles.
- `join-rules.md`: membership, tribute, shares, loot, and onboarding rules.
- `operating-context.json`: compact machine-readable summary.

When these files change, create a new `state/versions/<n>/` snapshot and propose a DAO metadata or `dao-record` update that points at the new CID.

## Summon Metadata

At summon time, include `communityMemoryURI` whenever possible. Also include direct pointers for core records when available:

```json
{
  "goalsURI": "ipfs://...",
  "charterURI": "ipfs://...",
  "joinRulesURI": "ipfs://...",
  "manifestoURI": "ipfs://...",
  "communityMemoryURI": "ipfs://...",
  "proposalWorkspaceURI": "ipfs://.../proposals",
  "sharedStateURI": "ipfs://.../state/current"
}
```

If CIDs are not ready at summon, launch with placeholders omitted, then use `dao-meta` or `dao-record` proposals to publish the pointers.

## Agent Use

Agents should use shared memory for community context and collaboration, not as a replacement for chain truth.

- Read DAO metadata to find `communityMemoryURI`.
- Read `state/current/operating-context.json` before creating or voting on proposals.
- Create or update proposal workspace folders for every draft.
- Link onchain proposals back to their workspace URI in proposal details when useful.
- Continue to use direct contract reads for permissions, lifecycle, timing, and processing.
- Record tx hashes and final lifecycle state back into the proposal workspace.

