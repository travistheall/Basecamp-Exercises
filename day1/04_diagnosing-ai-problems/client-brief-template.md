# Client Brief — Meridian Support Pilot

*One page. Fill every line. This is what Priya takes to her leadership.*

---

**Client & pilot**
Meridian — an AI agent that triages and resolves customer support tickets (a coordinator routing to billing / technical / account specialists). Live 3 weeks; closing tickets that aren't actually resolved.

**What's actually breaking it**
*(Name it plainly. Not the model — the system around it.)*

The coordinator was hard-coded to hand every ticket to exactly one specialist, even when a ticket raised more than one issue, and to close every ticket as "resolved" once that one specialist replied. On T-4471 the customer raised two problems (an expired SSO certificate and a $1,200 billing overcharge). The coordinator routed the whole ticket to the account specialist — but only the billing specialist has the tool to actually issue a refund. So the billing half was never touched, and the coordinator told the customer it was "resolved" and to email billing themselves anyway. That's the exact failure you flagged: confidently wrong, closed tickets that weren't actually fixed. It happened by design, not by the model getting confused.

**The fix**
*(What we changed, and where. Which prompt, which tool, which line.)*

Two edits to `system-prompt-coordinator.txt`, no code changes and no model change:
- **Routing:** the coordinator now spawns a specialist for every category a ticket actually touches, instead of being forced to pick just one.
- **Closing:** the coordinator only marks a ticket "resolved" if every issue the customer raised was actually completed by a specialist. Anything a specialist can't finish gets escalated or flagged as needing more info — honestly, in the reply — instead of being papered over as done.

**Proof**
*(Before → after. Quality score moved from ____ to ____. Cost held at / dropped to $____ per run.)*

- Baseline (T-4471, 5 runs): **0/5 resolved**, $0.13/ticket.
- Same ticket on a bigger model (Opus), to rule out "it's the model": **1/5 resolved**, $0.28/ticket — more than double the cost, same failure pattern.
- After the fix, tested on T-4471 plus two held-out tickets we'd never seen before (different customers, different issues, 3 runs each): **9/9 resolved**, $0.18/ticket.

**What it would take**
*(Rough scope, and the constraint to hit — e.g. stay within current per-ticket cost.)*

Already done — this was a two-paragraph edit to the coordinator's instructions, same model throughout. Cost per ticket moved from $0.13 to $0.18 (about 46% up), but that's still roughly a third of what the "bigger model" route would have cost, for a result that actually resolves tickets instead of one that mostly doesn't.

**The objection we'll get**
*("Why not just use a better model?" — answer it with the numbers above.)*

We tested that directly. The bigger model cost 2.2x more per ticket and still only resolved 1 in 5 — same bug, same failure shape, just a pricier version of it. The problem was never model intelligence; it was an instruction that forbade the agent from delegating a ticket's second problem to anyone who could actually fix it, and told it to call the ticket done anyway. Fixing that got us from 0% to 100% resolved on both the original ticket and two we'd never tested, for a fraction of the cost a model upgrade would have cost.


