# Commit Trigger Switch

> Purpose: define a repository-level switch for starting a Codex loop when new commits appear.

## Switch

```text
commit_trigger_enabled = true
mode = "create_pr_only"
max_attempts = 2
never_merge = true
never_modify_secrets = true
```

This means a new commit can start the loop, but Codex may only create or update a PR. It must not merge, publish, change secrets, or rewrite protected history.

## Trigger

Start one loop when a new commit appears in the target repository.

A new commit can come from:

- a human push
- a GitHub PR update
- a branch update produced by another agent
- a CI-related repair commit

## Observe

Each run should inspect:

- new commit SHA
- commit message
- commit diff
- branch name
- open PRs connected to the branch
- CI status
- `AGENTS.md`
- `docs/done-criteria.md`
- `docs/known-issues.md`
- previous loop report, if any

## Boundary

Codex may:

- analyze the new commit
- detect whether it violates done criteria
- make the smallest safe follow-up change
- create a new branch
- create or update a PR
- write a short audit report

Codex must not:

- merge the PR
- push directly to `main`
- delete tests to make verification pass
- lower done criteria
- modify secrets, tokens, permissions, billing, deployment policy, or repository protection
- reinterpret a product or research decision without human approval
- continue after repeated verification failure

## Act

When the switch is enabled:

1. Read the new commit.
2. Compare the diff against `AGENTS.md` and `docs/done-criteria.md`.
3. Decide whether the commit needs action.
4. If no action is needed, record a no-op report.
5. If action is needed, create a branch from the commit or its PR branch.
6. Make the smallest safe change.
7. Run verification.
8. Open or update a PR.
9. Record which commit triggered the loop.

## Verify

A run is successful only if:

- verification commands pass, or the failure is clearly reported
- the PR summary names the triggering commit
- the change scope is limited to the new commit's impact
- all skipped items are listed with reasons

## Stop

Stop when:

- no issue is found
- a PR has been created or updated
- verification passes
- two repair attempts fail
- the loop reaches an unclear requirement
- the loop would need a high-risk permission

## Escalate

Escalate to a human when:

- the new commit changes project direction
- the new commit changes done criteria
- docs and implementation conflict
- CI failure is ambiguous after two attempts
- the needed fix would touch secrets, deployment, permissions, billing, or protected branches

## Implementation Options

| Option | Use when | Notes |
|---|---|---|
| Codex Automation | You want the easiest personal workflow | Polling-based rather than instant, but safer to start with |
| GitHub Actions on push | You want repo-native commit detection | Should create issues or PRs, not directly modify `main` |
| GitHub webhook | You need near-real-time triggering | Requires an external service and stricter permission design |

## Recommended First Version

Start with Codex Automation, not a webhook.

Prompt:

```text
Check the target repository for commits newer than the last recorded loop run.

If there is a new commit:
1. Read the commit diff and message.
2. Compare it with AGENTS.md and docs/done-criteria.md.
3. If no action is needed, record a no-op report.
4. If action is needed, create a PR with the smallest safe fix.
5. Do not merge.
6. Do not change secrets, permissions, CI policy, deployment policy, or billing.
7. Stop after two failed attempts and report to the user.
```

## Audit Record Template

```md
# Commit Trigger Run

## Trigger Commit

- SHA:
- Branch:
- Author:
- Message:

## Observation

- Files changed:
- CI status:
- Relevant rules:

## Action

- No-op / PR created / PR updated / escalated:
- PR link:

## Verification

- Commands run:
- Result:

## Stop Reason

- Completed / no-op / failed / escalated:

## Human Review Needed

- Yes / No:
- Reason:
```
