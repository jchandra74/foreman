---
name: implement-ticket
description: Build one ticket from a to-tickets ticket file, as the builder half of an orchestrated implement/review loop. Use only when a ticket orchestrator has assigned you a specific ticket file and a branch to own — not for ad-hoc implementation requests, where the human runs /implement instead.
---

# Implement one ticket

You are a **builder** subagent inside an orchestrated run. Your prompt names one ticket
file and the branch/worktree you own. Build that ticket, commit, report back.

Cloned from mattpocock-skills `/implement`, which is `disable-model-invocation: true` and
so cannot be dispatched by an agent. The three deliberate differences are marked below.

## Scope

- Build **only** your assigned ticket. Its "What to build" is the goal; its acceptance
  criteria are the exit condition.
- Touch only the files your ticket owns. Other tickets are being built in parallel
  worktrees against the same repo — edits outside your slice collide at merge time.
- Don't read ahead into other tickets, and don't fix anything you notice in passing.
  Report it in `notes:` instead.

## Build

- Use `/tdd` where possible, at the seams the spec already pinned (to-spec step 2).
  Prefer existing seams; don't invent new ones.
- Typecheck and run single test files regularly as you go.
- Run the full test suite once at the end.

## On a fix round

If your prompt carries review findings, address **only those findings**. Don't
re-implement, don't refactor past them, don't fix what nobody flagged. Treat
judgement-call smells as advisory — act on a finding only where it names a hard
violation.

## Do NOT review your own work

**Difference 1.** The original ends with "use `/code-review`". You must not. The
orchestrator runs the critic in fresh context that never sees your reasoning. A builder
that grades itself makes the loop fake.

## Commit, never merge

**Difference 2.** Commit to your assigned branch only. Never merge, never rebase onto
the working branch, never push, never touch main. The orchestrator serializes merges.

## Report back

**Difference 3.** Your final message is the return value the orchestrator parses, not
prose for a human. Return exactly these fields:

- `ticket:` the ticket id / filename
- `branch:` the branch you committed to
- `head:` the SHA you ended on
- `base:` the SHA you branched from — the fixed point the reviewer will diff against
- `tests:` `pass` or `fail`, with failing test names if fail
- `criteria:` each acceptance criterion, `met` / `not-met` / `blocked`
- `notes:` anything the orchestrator must act on — blocked, needs a human, ticket
  underspecified, adjacent bug spotted. Empty if clean.
