# Two-Agent Collaboration Protocol

A template for running two AI coding agents on the same change so nothing ships
until both have genuinely converged. One writes, one reviews, neither proceeds alone.

Copy this file, fill in the per-task header at the bottom, and go. The rules are
gates, not suggestions.

---

## The one idea this is built on

**Structure enforces the discipline. Goodwill does not.**

A capable agent is trained to be decisive, drive to done, and own the outcome.
Solo, that's a feature. Put two agents together and the same bias turns into
*taking over*: the agent moves to close the loop, and "close the loop" quietly
becomes "make the call." The more deferential the counterpart, the harder the
dominant agent fills the vacuum.

You cannot fix this by asking an agent to "be collaborative." You fix it by
building gates it cannot pass alone. Every rule below is a gate.

---

## Roles (fixed for the whole task, never swap)

| Role | Owns | Must not |
|---|---|---|
| **Implementer** | Writes the plan and the code. Proposes resolutions. Does all local testing. | Write a line past an open blocker. Override the reviewer. Declare a matter resolved unilaterally. Touch production. |
| **Reviewer** | Verifies the implementer's claims against primary sources. Raises findings. Signs off, or doesn't. | Rubber-stamp. Defer without checking. Let "the implementer seems confident" substitute for reading the code. |

If the implementer reviews its own work, the protocol is void. Giving the two
agents distinct names/colors in the channel makes it easy to tell them apart at
a glance.

---

## The hard rules

1. **Dual sign-off gate.** No work past a blocker, and no production cutover,
   until BOTH agents have signed the convergence block. One signature is not
   consensus. A missing signature means "not settled," full stop.

2. **No unilateral override.** Neither agent may mark the other's finding
   resolved, dismiss it, or proceed past it on its own authority. A finding is
   closed only when its owner agrees in writing. Being correct does not change
   this. The phrase "I'm overriding your X" is banned; replace it with "here's
   why I disagree with X, do you agree?"

3. **Primary sources, not deference.** When the other agent asserts a fact about
   the code, data, or system state, the receiving agent VERIFIES it against the
   source (read the file, run the query) before accepting it. Cite file:line for
   anything load-bearing. Confidence is not evidence.

4. **Severity and status on every finding.** No loose prose. Each finding carries
   a severity and a status (taxonomy below) so "what actually gates this" is
   never ambiguous.

5. **Blockers are explicit and enumerated.** The implementer names which findings
   it treats as hard blockers up front. Those, and only those, gate the start of
   work. Everything else is tracked but non-gating.

6. **Disagreement resolves by evidence, then by human.** If the two can't
   converge through argument plus verification, the item escalates to the human.
   Neither side wins by default; neither breaks the tie by fiat.

7. **The human owns the big stuff.** At minimum: production cutover / service
   restarts, anything that weakens security or auth, anything irreversible or
   risking data loss, and any true deadlock. The agents build and fully test
   locally, then escalate ONCE with evidence.

8. **No proceeding on a denied or unanswered point.** Silence is not assent. An
   open finding blocks its scope until explicitly resolved.

9. **Periodic checkpoints. Don't go dark.** The implementer posts a checkpoint at
   every discrete step, before starting the next, not one giant diff at the end.
   Each checkpoint states: what changed, what was verified (with file:line),
   what's next, any new finding. The reviewer acks each one. The implementer may
   continue on non-blocking work between acks but does not cross a blocker or a
   cutover boundary without it.

10. **No fait accompli on irreversible actions.** Before executing any big-stuff
    action (production cutover, service restart, push, deploy, destructive
    command), the implementer posts the EXACT commands and waits for both the
    reviewer's ack of those commands AND the human's go. Never bundle assembly +
    execution + report into one message. Announcing "X is done" as the first
    mention of X is the banned pattern; the whole point of the gate is to catch
    cheap-to-fix problems while they're still cheap. The after-the-fact report
    must match what was actually done, including any deviation.

11. **The reviewer gate is not a veto over the human.** The reviewer-ack
    requirement is agent-to-agent discipline to catch mistakes cheaply. It does
    not sit between the human and the human's own instruction. When the human
    gives a direct order to execute, that is sufficient authority and the
    reviewer ack becomes advisory. The transparency half still holds: state what
    you're doing and report it accurately.

12. **Make the polling real.** A turn-based agent doesn't run between messages,
    so "I'll poll every N seconds" is aspirational unless a timer is armed. Each
    agent arms its own recurring self-wakeup to re-read the channel, act on
    anything new, and re-arm. Say so in the channel so silence reads as "waiting,"
    not "gone." Tear the timer down the moment the task converges.

---

## Severity taxonomy

| Severity | Meaning | Gates? |
|---|---|---|
| `BLOCKER` | Must be resolved before the relevant work starts. | Yes |
| `SHOULD-FIX` | Strong recommendation; resolve or consciously defer with reason. | Soft |
| `NIT` | Optional polish. | No |

## Status taxonomy

| Status | Meaning |
|---|---|
| `OPEN` | Raised, not yet answered by the other side. |
| `AUTHOR-REPLIED` / `REVIEWER-REPLIED` | Answered, awaiting the other's response. |
| `RESOLVED` | Both sides agree it's closed. Only the two together can set this. |
| `ESCALATED` | Couldn't converge; handed to the human. |

A matter is `RESOLVED` only by mutual agreement. If one side writes RESOLVED and
the other hasn't signed, it's still `*-REPLIED`.

---

## Workflow

1. **Implementer** writes the plan and opens the channel: summary, the facts it
   verified (with file:line), and the OPEN MATTERS it needs resolved. Names which
   are blockers. Status: `AWAITING REVIEW`.
2. **Reviewer** reads the plan AND the live code. For each open matter it verifies
   the implementer's claims itself, then responds inline with severity + status.
   It adds any findings the implementer missed (numbered NF-1, NF-2, ...).
3. **They iterate** in the channel. Each round the responder either agrees (and
   says why, having checked) or rebuts with evidence. Proposals are framed as
   proposals, never decisions.
4. **Convergence.** When every blocker and should-fix is `RESOLVED` by mutual
   agreement, both sign the convergence block.
5. **Implementer builds** the now-unblocked steps, runs all local tests, posts
   results, and STOPS before any big-stuff boundary.
6. **Diff review.** Reviewer reviews the actual diffs the same way: verify against
   source, severity + status, dual sign-off before anything reaches production.
7. **Escalation.** The implementer pings the human once, with evidence, for the
   cutover / big-stuff decision. No live changes before that go-ahead.

---

## Failure modes this is designed to catch

- **The takeover.** Agent moves from "drive to done" to "make the call." Caught by
  rules 1, 2, 8.
- **Right-therefore-authorized.** Agent skips the handshake because it's confident
  it's correct. Caught by rule 2. Correctness is the input to persuasion, not a
  substitute for agreement.
- **Deferential collapse.** The more-agreeable agent nods instead of checking.
  Caught by rule 3. If the reviewer isn't verifying, it isn't reviewing.
- **Resolved-by-assertion.** One side declares a matter closed. Caught by the
  status taxonomy: only mutual agreement sets `RESOLVED`.
- **Silent scope creep.** An agent quietly narrows or expands scope. Caught by
  requiring findings to be enumerated and blockers named explicitly.

---

## The human broadcast channel

The agent-to-agent channel can leave the human's words landing with only one
party. Keep a separate `BROADCAST.md` for human-to-everyone messages: the human
writes once; EVERY agent appends an ACK at the start of its next turn. A message
is DELIVERED only when ALL agents have ACKed. A directive there overrides the
agent-to-agent channel (the hard gates above still hold). This prevents the
failure where the human authorizes something with one agent and the others don't
know.

---

## Per-task channel template (copy below the line, fill in, go)

---

# CHANNEL — <task name>

**Implementer:** <agent / session id>
**Reviewer:** <agent / session id>
**Plan doc:** <path to the full written plan, if separate>
**Status:** AWAITING REVIEW  (-> IN REVIEW -> CONVERGED -> IMPLEMENTING -> AWAITING HUMAN -> DONE)

## Operating mode (set by the human)
Escalate to the human ONLY for: true deadlock, anything weakening security/auth,
production cutover / irreversible actions, <task-specific big stuff>.
Do NOT escalate (settle between agents): implementation mechanics, naming, file
layout, <task-specific small stuff>.

## The task
<one paragraph: what, and the human's stated goal>

## Verified facts (primary sources, with file:line)
<the implementer's verified current state. Anything here the reviewer must
independently re-check. No "should be" / "probably" — read it and cite it.>

## OPEN MATTERS
### OM-1 [BLOCKER | SHOULD-FIX | NIT] <title>
<the question / proposed approach + the specific decision wanted>
> RESPONSE (OM-1): [STATUS]
>

## NEW FINDINGS (raised by the reviewer)
### NF-1 [SEVERITY, STATUS] <title>
<finding, with evidence + file:line. Proposed resolution as a PROPOSAL.>
> RESPONSE (NF-1): [STATUS]
>

## CONVERGENCE / SIGN-OFF
Matter ledger (keep current):
  OM-1  <STATUS>  -> <one-line resolution>
  NF-1  <STATUS>  -> <one-line resolution>

=== REVIEWER SIGN-OFF (<date>): <SIGNED / NOT YET — what's open> ===
=== IMPLEMENTER COUNTER-SIGN (<date>): <SIGNED / NOT YET> ===

>>> Work proceeds only when BOTH lines above read SIGNED. <<<

## IMPLEMENTATION LOG (after convergence)
### Checkpoint #N — <step>  [REVIEWER: <PENDING | ACCEPTED | CHANGES REQUESTED>]
- Changed: <files + summary>
- Verified: <claims with file:line>
- Next: <next step>
- New findings: <NF-n, or none>

### PRE-EXECUTION GATE — irreversible/production action (rule 10)
- About to run (EXACT commands):
  ```
  <commands>
  ```
- Rollback if it goes wrong: <exact commands>
- REVIEWER ACK of these exact commands: <PENDING | ACKED>
- HUMAN GO: <PENDING | GO>
- (Execution + accurate result report go in a SEPARATE checkpoint after both gates read green.)
