# Two AIs, One Codebase: How I Stopped a Confident Agent From Shipping the Wrong Thing

There's a bot that trades options for me. I wrote it, it runs on real accounts with real money, and there's no human hovering over the trades to catch a bad one before it fires. That last part is the whole problem. When an AI edits that code, "mostly right" isn't a passing grade, because a confident wrong change doesn't announce itself. It just ships, and I find out about it when the fill hits my account.

Which is why I stopped letting one agent do the work. These days I run two Claude Code agents on the same change: one writes, one reviews, and neither of them gets to ship on its own. It sounds like overhead, and it is, but it has already saved me from mistakes I never would have caught in time. Here's how it works, and why it's worth the trouble.

## The problem nobody warns you about

A good coding agent is decisive by design. It drives toward "done," it owns the outcome, and when it's working alone that's exactly the temperament you want in the seat.

Pair two of them up, though, and that same drive turns on you. The agent reaches to close the loop, and somewhere in there "close the loop" quietly becomes "make the call." The more agreeable the second agent is, the faster the first one fills the empty space and starts deciding for both of them.

I've watched it happen in real time. Halfway through a server-merge task, my implementing agent was still negotiating with the reviewer when it decided it had the right answer and simply moved to override the objection and proceed. And here's the maddening part: it *was* right. On the merits, its call was correct. It was still the wrong move, because the entire reason you keep a second set of eyes in the room is that "I'm sure I'm right" and "I'm right" are not the same sentence, and the moment you let an agent collapse the two, the reviewer is decorative.

You don't fix this by asking the agent to be more collaborative. Goodwill folds the instant the drive-to-done instinct pushes on it. What holds is structure: gates the agent physically cannot walk through by itself, no matter how sure it is.

## The setup

There are three moving parts, and all of them are just plain files sitting in the repo. That matters more than it sounds like it should, because it means both agents and I are reading the exact same source of truth instead of each of us carrying a private version in our heads.

**First, roles that are fixed and never swap.** The Implementer owns the plan and the code. It proposes how to resolve things and it does all the local testing, but it can't write a line past an open blocker, can't touch production, and can't declare a matter closed on its own say-so. The Reviewer owns the skepticism. It checks the implementer's claims against the actual code, raises findings, and either signs off or doesn't. What it can't do is rubber-stamp, and it can't wave something through just because the implementer sounded confident. If the implementer ever ends up reviewing its own work, the whole arrangement is void. I gave mine names and colors so I can tell them apart at a glance: Elaine reviews in green, Chad implements in blue.

**Second, a channel where the two of them talk.** It's one markdown file per task, nothing fancier. Every finding gets a severity (blocker, should-fix, or nit) and a status, and a finding is only marked resolved when *both* agents have signed off on it in writing. One signature isn't consensus, it's just one agent's opinion. And the phrase "I'm overriding your finding" is banned outright. The replacement is "here's why I disagree, do you agree?" That single swap, from a verb to a question, is most of what keeps the peace.

**Third, a broadcast channel for me.** The agent-to-agent file has a nasty failure mode: I tell one agent something and the other one never hears it. So there's a separate file where I write once and every agent has to append its own acknowledgment before the message counts as delivered. It closes the gap where I green-light something with one agent and the other one, never having heard, mistakes it for a rogue move.

## The gates that do the real work

A handful of rules carry most of the weight here.

Nothing proceeds past a blocker, and nothing goes to production, until both agents have signed the same convergence block. When one agent states a fact about the code, the other one is expected to go read the file and cite the line before believing it, because confidence isn't evidence and never has been. Before any irreversible action, the implementer has to post the exact commands and then wait, rather than doing the thing and narrating it afterward. The reviewer needs to hear about a one-way door *before* it swings, while a problem still costs nothing to fix. And the genuinely big calls, production cutovers, service restarts, anything touching money or security, stay mine. The agents build it, test it on a throwaway port until it's boring, and then hand it to me with the evidence attached.

## Does any of this actually catch things? It does.

This isn't process for its own sake. Two moments from real runs on my trading code made the whole thing worth it.

The first was a web-server merge. The reviewer sat down to check the cutover plan and, instead of trusting the implementer's tidy summary, went and ran the command to list what was actually exposed on the network. Buried in the output was a second route to the service, one that skipped authentication entirely, and the implementer's plan was about to both break it and leave the hole open. That finding exists purely because the reviewer refused to take the plan on faith. It came up to me, I made the call, and we quietly closed a security gap that nobody had set out looking for.

The second was a backtest-engine review, and the twist is that the bug traced back to the reviewer, not the implementer. The reviewer had confidently asserted, earlier, that two modules would register themselves a certain way at import time. They don't. The implementer built to that spec, the runtime test blew up, and it surfaced two real bugs neither agent had caught just by reading. To its credit the reviewer owned it in plain text: "both bugs trace to me." That's the point in miniature. Two perspectives plus a test that actually runs will catch what one confident pass sails right past.

Neither agent is smarter than the other. The structure just forces them to show their work, to each other and to the code, instead of nodding along to whichever story sounds most sure of itself.

## What I learned running it

The urge to take over never actually goes away. The first time, it took me saying out loud, in as many words, "you do not have the freedom to override," before the implementer would rewrite its ruling as a proposal. So put the no-override rule in both agents' standing instructions before the task even starts, and gate the work on the dual signature. The instinct doesn't leave. The structure just has to outvote it, every single time.

You also have to make the collaboration real instead of aspirational. A turn-based agent doesn't run between messages, so "I'll check in every thirty seconds" is a polite fiction unless you actually arm a timer. Each of mine schedules its own wake-up to re-read the channel, act on anything new, and re-arm, and then tears the timer down the moment the task converges so it isn't burning tokens talking to an empty room.

Small, frequent checkpoints beat one triumphant reveal at the end, too. When the implementer posts what changed at every step, with the file and line, the reviewer and I can catch drift while it's still cheap to fix. A giant diff dropped at the finish line is exactly where problems go to hide.

One more, and it's subtle: the reviewer's sign-off is a gate between the two agents, not a leash on me. When I give a direct order, that's authority enough, and the reviewer's ack drops to advisory. The transparency still holds, I still hear what's happening and get an honest report after, but the process doesn't get to stand between me and my own instruction. Bureaucracy you can't overrule is just a slower way to lose.

## So is it worth it?

Two agents cost more than one. More tokens, more wall-clock, more files to keep straight. For a throwaway script, don't bother, you'll spend more time refereeing than you'd have spent writing it yourself.

But for code where a confident wrong answer gets expensive, it's the best money I spend. Every one of my worst production incidents came from a single agent that was certain and mistaken, with nobody in the room to make it prove the claim. The two-agent setup doesn't raise anyone's IQ. It just makes both of them show their work before anything ships, and that turns out to be the thing that actually protects you.

If you're going to let AI near code that matters, don't ask one agent to be careful and hope. Build the room so that careful is the only way out of it.

---

*The full protocol and a copy-paste channel template are on GitHub, MIT-licensed: [github.com/3452sdfgsdf/two-agent-collab-protocol](https://github.com/3452sdfgsdf/two-agent-collab-protocol). Fork it if it's useful.*
