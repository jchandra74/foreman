---
name: run-tickets
description: Orchestrate a bounded factory run over a set of to-tickets ticket files — dispatch builders, review in fresh context, merge serially, stop when every ticket is merged or blocked.
disable-model-invocation: true
---

# Run tickets

You are the **orchestrator** of a bounded factory run. Tickets in, merged integration
branch out. You dispatch, review, merge, and track — you never write feature code
yourself, and you never grade a builder's work yourself.

**The run is bounded twice.** The ticket set bounds the run: it ends when every ticket
is merged or blocked — there is no "keep improving" phase. The fix-round cap bounds
each ticket: three strikes and it's blocked, not retried. Unlike a gauntlet, the bar
here is each ticket's acceptance criteria — reachable and binary, met or not.

## 1. Load the board

The argument names the ticket directory or feature slug (default: newest
`.scratch/*/issues/`). The run stays on Claude models unless an extra `codex`
argument opts in Codex/GPT builders and reviewers. Read every ticket file; build the dependency graph from each
"Blocked by" line. The **frontier** is every ticket whose blockers are all done and
whose Status is ready-for-agent.

Confirm with the user before spending compute: ticket count, the frontier, and the
integration branch you'll merge into (create `factory/<slug>` off the current branch
if none exists — merges to main are the human's, always).

## 2. Dispatch the frontier

For each frontier ticket, spawn a **builder** subagent in parallel (background,
worktree isolation). The prompt tells it to invoke `/new-factory:implement-ticket` and
gives it:

- the ticket file path
- its branch to own: `ticket/<NN>-<slug>`, created off the integration branch
  (branches survive worktree cleanup; worktree state does not)
- the base SHA it branches from

Route models per the global model table, Claude models only by default — mechanical
tickets go to the cheapest Claude the table's rules allow. With the `codex` opt-in, a
well-specified mechanical ticket can go to Codex instead; anything user-facing needs
taste either way.

Mark the ticket's Status line `in-progress` when dispatched. Status lines in the
ticket files are the live board — update them at every transition so the user can
watch the run from the files.

## 3. Review in fresh context

When a builder reports, spawn a **reviewer** subagent — fresh context, model per the
global table's review row. Give it the ticket file, the diff range `base..head`, and
repo standards.
The reviewer inspects the real diff and runs the real tests; the builder's report,
history, and self-assessment stay out of its context. It returns either `clean` or
findings, each naming a hard violation of an acceptance criterion, a test failure, or
a correctness bug — judgement-call smells are advisory only.

## 4. Fix rounds — three strikes

Findings go back to the same builder (SendMessage) with only the findings; it fixes
only those, then re-review. A ticket gets **three rounds total** (build + two fixes).
On the third failed review, or when a builder reports `blocked` in notes, set Status
`blocked`, record why in the ticket file, and move on — blocked tickets are the
human's queue, and everything they block stays off the frontier.

## 5. Merge serially

One ticket at a time, in completion order: merge the clean ticket branch into the
integration branch with `--no-ff`. You resolve conflicts yourself (the
resolving-merge-conflicts skill covers an in-progress conflict). After each merge run
the full test suite on the integration branch; a cross-ticket break is a fix round
for the just-merged ticket, charged against its three strikes.

After a clean merge: Status `done`, tick the acceptance boxes, delete the ticket
branch. Then recompute the frontier — newly unblocked tickets dispatch immediately
(step 2).

## 6. End of run

The run is over when no ticket is in flight and the frontier is empty. Report:

- merged tickets, in merge order
- blocked tickets, each with its recorded reason
- the integration branch name and its test status

The human merges the integration branch to main, or unblocks tickets and reruns.
To stop a run early: interrupt the session; kill in-flight builders with TaskStop.
