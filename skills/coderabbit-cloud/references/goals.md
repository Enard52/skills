# Goal tasks

Check `coderabbit code new --help` and `coderabbit code goal --help` first. Older
CLIs or servers may not support goals. Never retry task creation without its goal.

```sh
coderabbit code new --agent --goal "<objective>"
coderabbit code new --agent --goal --goal-objective "<objective>" --resume "<instructions>"
coderabbit code goal <task> --agent
coderabbit code goal <task> --agent --set "<objective>"
cat <goal-file> | coderabbit code goal <task> --agent --set -
coderabbit code goal <task> --agent --clear
```

- `--goal` uses the prompt as the objective; `--goal-objective` overrides it and
  requires `--goal`. Objectives contain 1–4,000 Unicode characters after trimming.
- `--plan` conflicts with `--goal`. Switch existing tasks to code mode before
  setting a goal. There are no public budget or usage settings.
- Creation is billed work: run once, only at the user's request. Follow the
  `task_submitted` receipt rules in [follow.md](follow.md). Never duplicate a task
  because a later goal read failed or has not caught up.
- Reading is repeatable. `--set` and `--clear` change shared state and require an
  explicit request. They are mutually exclusive. A successful result is an
  acknowledgement; do not repeat a mutation merely because persisted state lags.
- Edits preserve paused, blocked, and limited states and existing budgets. An
  edit of a completed goal activates it again. Clearing removes the goal; it does
  not cancel the current turn.
- Stop a turn with `code cancel`. Send new instructions using `code resume -m`;
  server limits still apply. Read-only `resume` does not restart the goal.

## Results

Goal commands emit `complete` with `status: goal_shown`, `goal_set`, or
`goal_cleared`, `taskId`, `url`, and `goalState`. The state is `{ goal, revision? }`.
The goal has `objective`, `status`, `createdAt`, and `updatedAt`. Its status is
`active`, `paused`, `blocked`, `usageLimited`, `budgetLimited`, or `complete`.
`{ goal: null }` means no stored goal. The creation goal appears after runtime
initialization; an empty initial read does not prove creation failed.

`show` and follow/new task records include `task.goalState`; null means the
optional read was unavailable. Task lists do not fetch goals. Follow emits goal
updates as `trace` with `kind: goal` and `goalState`; newer revisions supersede
older ones, including deletion. `turn_completed` is not goal completion. Follow
retains its existing turn boundaries and wait limit; read `goal` for goal status.

For `server_unsupported`, report that the server needs updating. Do not fall back
to an ordinary task. Apply the shared authentication/access rules in SKILL.md.
After a transport error on an edit, read the goal to check its state before
considering another write; do not automatically repeat it.

Interactive equivalents are `/goal`, `/goal set <objective>`, `/goal clear`.
`/mode` identifies goal tasks. Clear the goal before `/mode plan`; `/mode code`
does not clear it. Recommend these only when the user asks for the interactive
view; use the agent commands for local agent operations.
