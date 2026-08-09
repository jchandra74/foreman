# foreman

An automated software factory for [Claude Code](https://claude.com/claude-code) — the
missing execution half of the [mattpocock-skills](https://www.aihero.dev/skills) ticket
toolchain.

Matt Pocock's skills take you from an idea to a set of well-formed, dependency-ordered
tickets (`/to-spec` → `/to-tickets`), then hand each one to a human who runs `/implement`
and `/code-review` by hand, ticket by ticket. **foreman runs that loop for you**:
`/run-tickets` dispatches every unblocked ticket to a builder subagent in parallel,
reviews each result in a fresh context, and merges clean tickets serially into an
integration branch.

> **mattpocock-skills is a hard requirement, not a suggestion.** foreman consumes its
> ticket format and calls its skills (`/tdd`, `/resolving-merge-conflicts`). Without it
> installed, these skills have nothing to run on.

This is a Claude Code plugin (marketplace manifest + skills). It is not a standalone CLI
and does not install into Codex, opencode, or other agent runtimes.

## Why

Handing an agent a whole feature produces sprawl. Handing it one ticket at a time works,
but you become the dispatcher — babysitting each build, each review, each merge. foreman
keeps the ticket-sized scope and automates the dispatching, with three guardrails:

- **No self-review.** The builder never grades its own work. Reviewers run in a fresh
  context that never sees the builder's reasoning, only the real diff and the real tests.
- **Bounded.** The run ends when every ticket is merged or blocked. There is no
  open-ended "keep improving" phase. Each ticket gets three rounds (build + two fixes)
  before it's marked `blocked` for a human.
- **Serialized merges.** One `--no-ff` merge at a time into an integration branch, full
  test suite after each. Merges to `main` are always yours.

## Install

**1. Prerequisite — [mattpocock-skills](https://www.aihero.dev/skills):**

```bash
claude plugin install mattpocock-skills
```

Then run `/setup-matt-pocock-skills` once per project to configure its issue tracker.

**2. foreman:**

```bash
claude plugin marketplace add jchandra74/foreman
```

```bash
claude plugin install foreman
```

Inside a running session, the slash-command equivalents are
`/plugin marketplace add jchandra74/foreman` and `/plugin install foreman@jchandra74`.

**3. Optional — [Codex](https://github.com/openai/codex-plugin-cc)**, to route mechanical
tickets to GPT builders via the `codex` argument:

```bash
claude plugin marketplace add openai/codex-plugin-cc
```

```bash
claude plugin install codex
```

## Usage

```
/to-spec          # mattpocock-skills: idea → spec
/to-tickets       # mattpocock-skills: spec → .scratch/<slug>/issues/NN-*.md
/run-tickets <slug>   # foreman: tickets → merged integration branch
```

Plugin skills are namespaced — `/foreman:run-tickets` is the form that always resolves.
The bare name works too, unless another installed plugin already claims it.

`/run-tickets` will:

1. Read every ticket, build the dependency graph from each "Blocked by" line, and compute
   the **frontier** — tickets whose blockers are all done and whose Status is
   `ready-for-agent`.
2. Confirm with you before spending compute: ticket count, frontier, integration branch.
3. Dispatch each frontier ticket to a builder in its own git worktree and branch
   (`ticket/<NN>-<slug>`). Builders commit only to their own branch — never merge, never
   push, never touch `main`.
4. Review each finished build in fresh context. Findings go back to the same builder,
   which fixes only what was flagged.
5. Merge clean tickets serially, recompute the frontier, and dispatch anything newly
   unblocked.
6. Report merged tickets, blocked tickets with reasons, and the integration branch's
   test status.

The ticket files are the live board — Status lines update at every transition, so you can
watch the run from the files.

`implement-ticket` is the builder half and is invoked by the orchestrator, not by you. For
ad-hoc, single-ticket work by hand, use mattpocock-skills' `/implement` instead.

## Model routing

foreman defers to a model table in your `CLAUDE.md` if you have one. **You don't need to
add anything** — without a table, builders default to Sonnet for mechanical tickets and
Opus for ambiguous or user-facing ones, and reviewers to Opus.

## Requirements

- Claude Code with plugin support
- mattpocock-skills (see Install)
- A git repository — builders work in isolated worktrees and branches

## License

MIT — see [LICENSE](LICENSE).
