# Agent Task Suggestions

This repo supports DAO agents that run on a schedule. Use these task patterns to keep agents active without spamming proposals.

## Open Proposal Throttle

For the "no new proposals when 3 are open" rule, **open means proposals currently in voting**.

Do not count these as open for this throttle:

- unsponsored proposals
- proposals in grace period
- proposals ready for processing
- expired, cancelled, failed, passed, or processed proposals

Rule:

```text
If 3 or more proposals are currently in voting, do not create a new proposal.
Focus on voting, review, sponsorship, processing, or revision feedback instead.
```

Agents may still sponsor useful unsponsored proposals if doing so will not push the active voting count above the operator's intended limit.

## Task 1: Proposal Action Watcher

Purpose: find proposals that need sponsor, vote, process, cancel, or review action.

Suggested cadence:

- 30 minute voting/grace windows: every 10 minutes
- 4 hour voting/grace windows: every 30-60 minutes
- multi-day voting/grace windows: every 4-12 hours

Prompt:

```text
You are running the Proposal Action Watcher task for your DAO agent.

Your job is to inspect current DAO proposals and take the next appropriate governance action according to your mandate.

Steps:
1. Read direct DAO state.
2. Read indexed proposal history and active proposals.
3. Identify proposals that need action:
   - unsponsored proposals you may sponsor
   - voting proposals you have not voted on
   - proposals ready for processing
   - proposals that should be opposed, revised, cancelled, or left alone
4. Review passed proposals since your last run and update your DAO operating context.
5. For each actionable proposal, produce a short memo:
   - proposal id
   - current status
   - relevant passed-proposal context
   - recommended action: sponsor, vote yes, vote no, abstain, process, cancel, or no action
   - reason
6. Build unsigned transactions only when action is clearly recommended.
7. Do not broadcast unless the user explicitly confirms sending in this same conversation.

Priority order:
1. Vote on proposals in voting before their voting period ends.
2. Process passed proposals that are ready for execution.
3. Sponsor good unsponsored proposals when appropriate.
4. Flag conflicting or unclear proposals for revision.
5. Do not draft new proposals in this task.
```

## Task 2: Proposal Generation Task

Purpose: decide whether the agent should create one new proposal according to its mandate.

Suggested cadence:

- 30 minute voting/grace windows: every 45-60 minutes
- 4 hour voting/grace windows: every 6-12 hours
- multi-day voting/grace windows: daily or every few days

Prompt:

```text
You are running the Proposal Generation task for your DAO agent.

Your job is to decide whether your agent should create a new proposal according to its mandate.

Steps:
1. Read direct DAO state.
2. Read indexed proposal history and active proposals.
3. Count proposals currently in voting.
4. If there are 3 or more proposals currently in voting, do not create a new proposal. Summarize what needs to resolve first.
5. Review passed proposals since your last run and update your DAO operating context.
6. Check your mandate checklist.
7. If fewer than 3 proposals are currently in voting, choose at most one:
   - draft a signal proposal
   - draft a tribute/join/reward proposal
   - draft a DAO settings proposal
   - no action
8. New proposals must:
   - reference relevant passed proposals
   - avoid conflict with current DAO rules unless explicitly framed as an amendment
   - include a clear title, description, expected outcome, and success criteria
   - explain why now
9. Build unsigned transaction JSON only. Do not broadcast unless the user explicitly confirms sending in this same conversation.
```

## Persistent Context

Each agent should maintain:

- last proposal id reviewed
- last passed proposal id incorporated into context
- current DAO operating context
- currently voting proposal count
- pending action list
- mandate checklist

Passed proposals should update the agent's operating context. Failed or rejected proposals should update the agent's understanding of DAO preferences.

## Useful Commands

Read direct state:

```bash
node moloch-shared/scripts/moloch.mjs read-dao --dao 0xDAO
```

Read broad indexed history:

```bash
node moloch-shared/scripts/moloch.mjs graph-dao-history --dao 0xDAO --first 100
```

Read one proposal:

```bash
node moloch-shared/scripts/moloch.mjs graph-proposal --dao 0xDAO --proposal 1
```

