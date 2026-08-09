# foreman

A bounded ticket factory for [Claude Code](https://claude.com/claude-code). `/run-tickets`
orchestrates parallel builder subagents, reviews their work in fresh context, and merges
clean tickets serially into an integration branch — no self-review, no open-ended
"keep improving" loop.

This is a **Claude Code plugin** (marketplace manifest + skills). It is not a standalone
CLI and doesn't install into Codex, opencode, or other agent runtimes.

## What it does

- **You** write tickets (one file per ticket, with a "What to build", acceptance
  criteria, and "Blocked by" dependencies) into a `to-tickets` directory.
- **`/run-tickets`** (orchestrator) reads the board, dispatches every unblocked ticket
  to a builder subagent in its own worktree/branch, and never writes feature code
  itself.
- **`implement-ticket`** (builder) builds one ticket, commits to its own branch, and
  reports back — it never reviews its own work or merges.
- The orchestrator sends each builder's diff to a reviewer in a fresh context (no
  access to the builder's reasoning). Findings go back for up to two fix rounds; a
  ticket that fails three times is marked `blocked` for a human.
- Clean tickets merge one at a time (`--no-ff`) into an integration branch, with the
  full test suite run after each merge. You merge that branch to `main` yourself.

## Install

Add this repo as a plugin marketplace, then install the plugin:

```
/plugin marketplace add jchandra74/foreman
/plugin install foreman@foreman
```

## Usage

1. Write your tickets into a directory (default: newest `.scratch/*/issues/`), each
   with a Status line, acceptance criteria, and "Blocked by" references to other
   tickets.
2. Run the orchestrator, pointing it at that directory or a feature slug:
   ```
   /foreman:run-tickets <ticket-dir-or-slug>
   ```
3. Confirm the ticket count, frontier, and integration branch when asked.
4. Watch progress via the Status lines in the ticket files as builders, reviewers,
   and merges run.
5. When the run ends, merge the reported integration branch into `main` yourself, or
   unblock any blocked tickets and rerun.

`implement-ticket` is invoked by the orchestrator, not run directly — a human doing
ad-hoc implementation should use their own `/implement` workflow instead.

## Requirements

- Claude Code with plugin support.
- A git repository for the target project (builders work in isolated worktrees and
  branches).

## License

MIT — see [LICENSE](LICENSE).
