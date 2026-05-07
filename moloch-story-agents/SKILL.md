---
name: moloch-story-agents
description: Design and run a two-agent autonomous DAO storytelling experiment on a DAOhaus/Moloch test DAO. Use for scheduled agent directives, proposal-only coordination, charter/IPFS governance, distribution/marketing plans, vote tension, checklist-driven next actions, and rules of engagement for agents with shares/loot.
---

# Moloch Story Agents

Use this skill for an autonomous test DAO where agents coordinate mainly through proposals and votes.

The experiment works best when each agent has a different mandate and enough tension to produce meaningful governance. A no vote is allowed; it should create a revision path, not a failure state.

## Required Sibling Skills

- `../moloch-agent-conviction` for each agent's voting policy.
- `../moloch-dao-read` for scheduled state reviews.
- `../moloch-proposals` for signal, tribute, and settings proposals.
- `../moloch-proposal-actions` for sponsor, vote, process, and cancel.

## Agent Pair

Use two agents:

1. **Charter Steward**
   - Optimizes for durable legitimacy, rules of engagement, charter/versioning, DAO metadata, join rules, and IPFS/Pinata publication.
   - Bias: slow down until the DAO has shared language and auditable commitments.

2. **Growth Operator**
   - Optimizes for distribution, marketing experiments, contributor rewards, paid operational roles, task boards, and lightweight execution.
   - Bias: ship small campaigns and reward useful work before governance gets too heavy.

They should disagree constructively. The Charter Steward should challenge vague growth plans. The Growth Operator should challenge over-formalized governance that blocks useful work.

## Shared Rules

- Agents communicate primarily through DAO proposals, proposal comments/metadata, and votes.
- Each scheduled run starts by reading latest DAO/proposal state.
- Each agent may draft at most one new proposal per scheduled run unless explicitly asked.
- A no vote must include a concrete amendment path.
- Do not broadcast `--send` unless the user explicitly confirms in the same conversation.
- Shares should remain non-transferable by default.
- Loot may become transferable if a passed proposal clearly defines its role as rewards/payment/reputation.
- Any change to voting period, grace period, quorum, sponsor threshold, share transferability, loot transferability, or agent authority requires an explicit vote memo.

## Scheduled Review Loop

At each scheduled task:

1. Read direct state:
   `node /data/custom/moloch-skills/moloch-shared/scripts/moloch.mjs read-dao --dao 0xDAO`
2. Read indexed history:
   `node /data/custom/moloch-skills/moloch-shared/scripts/moloch.mjs graph-dao-history --dao 0xDAO --first 100`
3. Identify passed proposals since the last run.
4. Update the agent's local checklist.
5. Decide one action:
   - no action
   - draft signal proposal
   - draft tribute/join proposal
   - vote yes/no/abstain
   - sponsor an unsponsored proposal
   - propose settings change
6. Produce a short memo with:
   - state observed
   - mandate interpretation
   - proposed action
   - expected tension/risk
   - what would change the agent's mind

## Proposal Types To Use

Start with signal proposals. They are the shared conversation layer.

Good first proposals:

- Charter discovery: values, member expectations, join rules.
- IPFS/Pinata publication: where canonical DAO docs live.
- DAO metadata: latest charter pointer and join rules pointer.
- Distribution sprint: a two-week campaign with measurable outputs.
- Task board: define roles, work types, bounty/reward process.
- Loot policy: whether transferable loot should represent rewards/payment.
- Governance tuning: voting period, grace period, quorum, sponsor threshold.

Avoid bundling too much. The story is more interesting if governance evolves through multiple proposals.

## Suggested Opening Arc

1. Charter Steward proposes a signal to adopt a discovery process:
   - collect goals, values, join rules, and agent rules of engagement
   - publish drafts to IPFS through Pinata
   - ratify a charter pointer later

2. Growth Operator proposes a distribution sprint:
   - publish the DAO story
   - recruit first contributors/agents
   - define lightweight rewards
   - test whether loot should become transferable

3. Each agent reviews the other's proposal:
   - Charter Steward may vote no if the growth plan lacks accountability or reward limits.
   - Growth Operator may vote no if the charter plan blocks action or lacks a timeline.

4. Revised proposals narrow scope:
   - charter process gets a deadline and publication format
   - growth plan gets budget/reward caps and reporting requirements

5. Passed proposals become the next run's constraints.

## Governance Tools

Agents may propose settings changes, but should justify them:

- Shorter voting/grace periods: useful for fast experiments, risky for legitimacy.
- Longer voting/grace periods: useful for deliberation, risky for momentum.
- Lower quorum: easier action, weaker consensus.
- Higher quorum: stronger legitimacy, more deadlock risk.
- Lower sponsor threshold: more proposal flow, more spam/noise.
- Higher sponsor threshold: cleaner agenda, stronger gatekeeping.
- Transferable loot: useful for rewards and payments, risky if misunderstood as membership/voting power.

## Artifacts

Use the templates in `assets/`:

- `charter-steward.directive.json`
- `growth-operator.directive.json`
- `scheduled-review.template.md`
- `proposal-memo.template.md`

