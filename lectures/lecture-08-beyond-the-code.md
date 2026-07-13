# Lecture 8 — Beyond the Code

Official notes: https://missing.csail.mit.edu/2026/beyondcode/

**Big idea:** software work isn't just writing code in a repo — it's
communication, review, documentation, collaboration, teaching, and judgment. Every
doc/format is an **interface** whose job is to *lower the cost for someone else to
understand your work*. Clear interface → the team stops guessing. Fuzzy interface →
tech debt starts accumulating in the communication itself.

> AI can draft/summarize/translate/check — but it **cannot be responsible for the
> facts.** Hand a vague problem to AI and forward the output → you just amplified
> the noise.

## Async communication: conclusion first

Teams are spread across timezones/schedules. Async writing must let the reader keep
going **without pinging you**. Give the conclusion first, then evidence + context —
don't drag the reader through a stream of consciousness.

| Give | Not just |
|---|---|
| **Background** — why this, why now | "there's a problem" |
| **Goal** — what needs deciding/fixing/confirming | "everyone take a look" |
| **Evidence** — links, logs, screenshots, commands | "I feel like…" |
| **Responsibility** — who does the next step | "we'll deal with it later" |
| **Timing** — when you need feedback | assuming everyone's online now |

## README & contributing guide: the front door

README is read by first-timers, future maintainers, CI, and AI agents. It answers:
what is this, what problem does it solve, how to install, run, test, contribute.

Minimum sections: **Overview** (purpose + boundaries), **Quickstart** (shortest path
from a clean environment), **Development** (deps, tests, lint), **Configuration**
(env vars + examples), **Contributing** (how to file issue/PR, code style),
**Troubleshooting** (common failures).

## Issue / bug report: make the problem reproducible

An issue is a **trackable unit of work** — it has status + context and persists.
That's the difference from chat (temporary, no state).

Bug report high-value fields:

| Field | Why |
|---|---|
| Environment / version | OS, commit — "works on my machine" |
| **Steps to reproduce** | **the single most valuable field** |
| Actual vs Expected | names the gap precisely |
| Logs / screenshots | evidence, not "I feel like" |
| Impact / scope | urgency + who's affected |

> **Impact picks the bug; reproduction unlocks the fix.** You can't fix what you
> can't observe (ties to L4 debugging) — and you can't confirm a fix without a repro
> (ties to L7 red-then-green). No repro = a rumor, not a bug report.

Other issue types: **Feature** (user scenario, goal, non-goals, acceptance);
**Question** (context, what you already checked, the specific blocker);
**Design** (options, trade-offs, risks, the decision needed).

## PR & code review: small, focused, locate the risk

**PR** = a bundle of changes to merge into main. Rule: **small, focused, testable.**
Mixing a feature + a big refactor + reformatting into one PR makes review cost
explode. A good PR description: **Summary / Why / What changed / Validation / Risk.**

**Review** exists to catch risk, spread context, keep quality up. A good comment is
**specific, locatable, explains the risk**, ideally with an alternative:

- ❌ "This is bad / feels off." (vague, no location, attacks taste)
- ✅ "This could fail under `<case>` because `<reason>`. Suggest `<alternative>`.
  Add a test covering `<edge case>` to lock the behavior."

**Security angle (important for the sec track):** a giant mixed PR is a *security
risk*, not just messy. Reviewer fatigue on a 40-file diff → skimming → a dangerous
change (weakened auth check, leaked secret) slips through *because it's buried in
noise*. Hiding a malicious change inside a big boring PR is a real supply-chain /
insider vector. Defense: **separate behavior changes from formatting**, so the
meaningful diff stays small and impossible to miss.

## AI-era communication: own the facts

AI can turn rough ideas into structure, meeting notes into action items, dense
explanations into clear ones. But it also **invents facts, softens uncertainty, and
emits generic templates.** So when AI helps you communicate: keep the source of
truth, flag uncertain parts, and **verify commands, links, numbers, and names.**
Never forward an unverified AI summary as a team decision.

## Failure modes

- Issue with only emotion/conclusion, no repro or evidence.
- PR too big — feature + refactor + formatting mixed.
- Review comments vague / judging the person, not locating the risk.
- README that's a slogan, no install/run/test.
- Important technical decisions only mentioned in chat, never recorded.
- Forwarding AI summaries without verifying facts and links.

## Templates (from the notes)

**Bug report:** `## Summary / ## Environment / ## Steps to reproduce / ## Expected
/ ## Actual / ## Logs/screenshots / ## Notes`

**PR:** `## Summary / ## Why / ## Changes / ## Validation (- [ ] Tests / - [ ] Manual)
/ ## Risks / Rollback`

**Review comment:** "This may fail under `<case>` because `<reason>`. Suggest
`<alternative>`. Add a test covering `<edge case>` to lock behavior."

## Self-test

- **Who reads the README?** New people, maintainers, CI, future maintainers, tools —
  all rely on it to lower understanding cost.
- **What's in a good review comment?** Specific location, the risk reason, a suggested
  fix or the validation that's needed.
- **Issue vs chat message?** Issue = trackable work unit with state, context, evidence;
  chat is temporary.
- **After AI drafts a doc, check what first?** Facts, commands, links, numbers,
  boundaries, and uncertainty.
