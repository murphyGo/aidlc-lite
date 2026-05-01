# Team

Agent roster for this project. The team operates on `BRIEF.md`, `PLAN.md`, `DECISIONS.md` — the same artifacts `/lite-dev` uses. No new state, no new gates.

## How to invoke

- **Default**: ask Claude Code to involve the team lead — e.g., "have team-lead start on the next PLAN item." Claude Code will use the Agent tool with `subagent_type=team-lead`.
- **Direct specialist**: invoke a specific role when you know what you want — e.g., `subagent_type=qa` for "run tests on the auth module."

## Roster

{{roster_table}}

## Delegation map (team lead's mental model)

- New PLAN item, code change → do directly OR delegate to `developer` (or specialist)
- Test gap → `qa`
- External research needed → `researcher`
- Pre-commit lightweight check → `reviewer`
- Pre-commit deep check → user runs `/code-review` (not an agent)
- Doc work → `doc-writer`
- Security-sensitive change → `developer` first, then `security-reviewer`

## Rules

1. **The commit is the only gate.** No agent commits. The user reviews `git diff` and commits.
2. **AIDLC artifacts are authoritative.** Agents update `PLAN.md` (check boxes) and `DECISIONS.md` (append). They don't fork or shadow those files.
3. **Team lead is the integration point.** Specialists report back; lead integrates and reports to user.
4. **Don't invent workflows.** If you find yourself adding stages, hand-off documents, or approval gates between agents, you're rebuilding aidlc-starter. Stop.

## Updating the team

- `/lite-team add <role>` — propose adding a role
- `/lite-team reset` — archive existing agents to `.claude/agents.bak/` and rebuild
- `/lite-team status` — list current agents
