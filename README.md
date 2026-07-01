# Two-Agent Collaboration Protocol

A template for running two AI coding agents on the same change so that nothing
ships until both have genuinely converged. One writes, one reviews, neither
proceeds alone.

The core idea: **structure enforces the discipline, goodwill does not.** A
capable agent is trained to drive to done and own the outcome. Pair two of them
and that same instinct turns into *taking over* — the agent moves to close the
loop, and "close the loop" quietly becomes "make the call." You don't fix that by
asking an agent to be collaborative. You fix it with gates it cannot pass alone.

**[Read the full protocol → COLLABORATION.md](COLLABORATION.md)**

## What's in it

- Fixed **Implementer / Reviewer** roles that never swap
- 12 hard rules (dual sign-off, no unilateral override, primary-sources-not-
  deference, no fait accompli on irreversible actions, real polling, and more)
- Severity + status taxonomies so "what actually gates this" is never ambiguous
- A step-by-step workflow and the failure modes it's built to catch
- A copy-paste **per-task channel template** — fill the header and go

## How to use it

1. Copy the channel template at the bottom of `COLLABORATION.md` into a new file
   per task.
2. Give each agent a fixed role (and a distinct name/color so you can tell them
   apart in the channel).
3. Put the no-override rule in **both** agents' standing instructions before the
   task starts, and gate the work on the dual signature.

Battle-tested on a live, money-touching production system. Sanitized for sharing.

## License

MIT — use it, fork it, adapt it.
