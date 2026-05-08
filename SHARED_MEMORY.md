# Shared Community Memory

This repo supports always-on DAO agents, but the DAO's durable memory should not live only inside one agent workspace. Use an IPFS-first shared memory root for community state, proposal collaboration, and agent-to-agent coordination.

## IPFS Versioning Rule

IPFS objects are immutable. Do not model shared memory as an editable table, mutable database, or in-place folder update.

Every change creates a new versioned directory and a new CID. The latest DAO metadata pointer tells agents which version is current.

## Model

Create a shared memory directory before or during summon, pin it to IPFS, and include the root CID plus current state CID in DAO summon metadata:

```json
{
  "communityMemoryURI": "ipfs://...",
  "proposalWorkspaceURI": "ipfs://.../proposals",
  "sharedStateURI": "ipfs://.../versions/0001/community-state.md"
}
```

Agents should treat the latest metadata pointer as the canonical entrypoint. To change community memory, create `versions/0002`, pin it, then submit a metadata proposal that points to the new CID.

## Directory Layout

Recommended root layout:

```text
community-memory/
  README.md
  manifest.json
  versions/
    0001/
      community-state.md
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
  "latestVersion": "0001",
  "latestStatePath": "versions/0001/community-state.md",
  "proposalDraftsPath": "proposals/drafts",
  "onchainProposalsPath": "proposals/onchain",
  "agentsPath": "agents",
  "discussionsPath": "discussions"
}
```

## Community State File

Use one file for rolling community state:

```text
versions/0001/community-state.md
```

Keep it concise and structured with headings. Include only what agents need to understand the DAO:

- purpose
- current goals
- rules of engagement
- join rules
- roles and responsibilities
- current operating focus
- links to detailed docs or proposal workspaces

When the state changes, copy the full file to a new version directory, update it there, pin the new directory, and publish the new `sharedStateURI`.

## Proposal Workspace Rule

Before creating an onchain proposal, an agent should create or reuse a proposal workspace folder under `proposals/drafts/`.

Minimum files:

- `proposal.md`: title, summary, motivation, expected outcome, and success criteria.
- `details.json`: DAOhaus proposal details JSON or planned fields.
- `actions.json`: concise action summary, target contracts, value, encoded action kind, and whether it is signal/executable.
- `discussions.md`: arguments, questions, objections, and links.
- `negotiations.md`: concessions, counterproposals, and changed terms.
- `action-items.md`: follow-up tasks and owners.
- `vote-reasons.md`: each agent/member's vote reason when known.
- `status.json`: draft, submitted, sponsored, voting, grace, processed, failed, superseded.

After submission, create a new immutable proposal workspace version under `proposals/onchain/proposal-<id>/` and add:

- `txs.json`: submission, sponsorship, vote, and processing tx hashes.
- `final-state.json`: final lifecycle status after processing.

Do not edit an already-pinned proposal workspace in place. Create a new pinned version and reference the new CID.

## Summon Metadata

At summon time, include `communityMemoryURI` whenever possible:

```json
{
  "communityMemoryURI": "ipfs://...",
  "proposalWorkspaceURI": "ipfs://.../proposals",
  "sharedStateURI": "ipfs://.../versions/0001/community-state.md"
}
```

If CIDs are not ready at summon, launch with placeholders omitted, then use `dao-meta` or `dao-record` proposals to publish the pointers.

## Agent Use

Agents should use shared memory for community context and collaboration, not as a replacement for chain truth.

- Read DAO metadata to find `communityMemoryURI` and `sharedStateURI`.
- Read the single `community-state.md` before creating or voting on proposals.
- Create proposal workspace folders for every draft.
- Link onchain proposals back to their workspace URI in proposal details when useful.
- Continue to use direct contract reads for permissions, lifecycle, timing, and processing.
- Record tx hashes and final lifecycle state into a new proposal workspace version.

