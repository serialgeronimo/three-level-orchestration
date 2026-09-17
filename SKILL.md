---
name: three-level-orchestration
description: "Trigger: $three-level-orchestration, three-level orchestration. Coordinate an Orca root, review leaders, and same-worktree workers."
license: Apache-2.0
metadata:
  author: "Geronimo Serial"
  version: "1.1"
---

# Three-Level Orchestration

## Activation Contract

Use only when explicitly invoked for a well-specified task. Always establish all three levels: root/team leader, review leader, and implementation worker.

## Hard Rules

- Load the `orchestration` skill before acting and follow its version-matched Orca guide for lifecycle, messaging, waits, recovery, and cleanup.
- Use Orca workers only. Never substitute in-process subagents.
- Keep the whole team in the root's current worktree. Level-2 leaders and level-3 workers use `--worktree current`; never create additional worktrees.
- Root owns scope, polling, integration, tests, commits, and the final report. Leaders review and delegate; workers implement. Neither delegated level commits.
- Treat `worker_done --outcome succeeded` as a claim, not evidence. The parent verifies the diff, tests, and acceptance criteria before accepting it.
- Fail closed if Orca is unavailable or level 3 cannot start. Do not collapse the hierarchy or let a leader implement directly.

## Decision Gates

| Situation | Action |
| --- | --- |
| Independent domains | Launch one review leader per domain in the current worktree. |
| Disjoint write scopes | Let leaders run workers in parallel. |
| Overlapping files or dependent edits | Assign one writer and serialize the work. |
| Ambiguous or non-decomposable task | Stop and request a sharper contract; never fake three tiers. |
| Read-only work | Preserve all three levels; return evidence instead of commits. |

## Execution Steps

1. Root creates the parent Run and self-contained leader Tasks with objective, allowed files, constraints, acceptance evidence, and ownership boundaries; start every leader with `--worktree current`.
2. Each leader creates a child Run, converts its domain into bounded worker Tasks, and starts at least one level-3 worker with `--worktree current`. Its primary value is independent review, not implementation.
3. Workers implement only their assigned scope, run focused checks, and send `worker_done` with files, tests, findings, and blockers.
4. Leaders inspect actual diffs and test output, reject or re-dispatch unsupported results, settle and clean up every child Dispatch, then return evidence and review findings to root.
5. Root continuously consumes leader messages. After repeated empty waits, inspect worker state, transcript tail, and diff; `live` alone is not progress.
6. Root reviews the combined diff, runs global verification, commits accepted work, and resolves every retain/release decision before reporting.

## Output Contract

Report each domain's verified outcome, review findings, tests, root-owned commits when applicable, unresolved blockers, and final cleanup state.
