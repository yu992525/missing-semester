# Lecture 7 — Agentic Coding

Official notes: https://missing.csail.mit.edu/2026/agenticcoding/

**Big idea:** an *agent* can take an action, see the result, and change its next
move based on what happened. That feedback loop is the whole difference — and the
whole danger. Agentic coding is the craft of organizing **goal, context, tools,
permissions, tests, and review** into a reliable human–AI loop.

## The ladder: autocomplete → chat → agent

| Tool | What it sees | What it can *do* |
|---|---|---|
| **Autocomplete** | nearby code | suggest the next line. Can't run anything. |
| **Chat** | context you paste in | explain / design / generate text back. Can't touch files. |
| **Agent** | reads files, errors, whatever it needs | **acts** — edits files, runs commands, checks results, retries. |

**The tell:** does a result come back that changes the next move? Yes = agent
(a *loop*). No = chat/autocomplete (one shot).

## The agent loop (5 stages)

**Observe → Plan → Act → Check → Iterate**

1. **Observe** — read files, error, request. Build a picture. (Look *before*.)
2. **Plan** — break down the task, pick the tool.
3. **Act** — edit a file, run a command.
4. **Check** — run the test, look at the diff/output. (Look *after*.)
5. **Iterate** — failed? go fix. passed? stop.

- **Observe = look before, Check = look after.** A `print`-and-read is *Check*.
- Drop **Check** and "it ran" masquerades as "it works" — the #1 failure mode.
- Longer loop → more guardrails needed. No stopping condition → **scope creep**.
  No test → agent calls "no error" = "done."

## Context engineering: too little it guesses, too much it drowns

A bigger context window is **not** higher quality — more room to hide the
important thing. Give the *right* things, not *everything*.

| Give this | Not this |
|---|---|
| **Goal** — user-visible behavior wanted | "optimize it" |
| **Relevant code** — the entry points that matter | the whole repo pasted in |
| **The error** — full message + repro command | just the last line |
| **Constraints** — what must NOT change (API/file) | let it guess the boundaries |
| **Acceptance** — the command that proves done | "looks fine to me" |

> One skill, two sides: **name the exact symptom** (cures too-little),
> **point at the exact file** (cures too-much). Vagueness kills both ways.

## Rules files: write the correction down once

A small in-project file (like a `CLAUDE.md`) holding test commands, style,
off-limits directories, security rules. Makes unwritten team habits **written and
automatic**, so you stop re-correcting every session.

- Rules must be **short, specific, executable.**
- ❌ "Write elegant code." ✅ "After changing Python, run `pytest tests/unit` and `ruff check`."
- Too long → ignored or self-contradicting.

> Test for any rule: **could an agent break it, and could you tell?** If not, it's decoration.

## Test-driven agent: red then green

AI is great at code that *looks* right; looking right ≠ being right. A test
converts a vague requirement into an **objective, un-arguable check.**

- **Bug workflow: red → green.**
  1. First write a test that reproduces the bug → it **fails** (red).
  2. Then fix code until it **passes** (green).
- Why the order: a self-written passing test proves nothing — it might pass on
  broken code too. **Green only means something if you saw it fail for the right
  reason first.**
- No test possible (UI/docs/refactor)? Demand other objective evidence:
  screenshot, before/after benchmark, lint pass, logs, manual repro.

## Security & responsibility: the loop can run commands

The source of the power (can run commands) is the source of the danger (can drop
tables, delete files, leak secrets, run up bills, message real users).

- **Least privilege by default:** read-only connections, sandboxes, `dry-run`,
  small scoped permissions, backups.
- **Human-in-the-loop** on high-risk ops: production data, billing, deletes,
  external messages, secrets.
- **Secrets:** agent should **never read or print** `.env`, tokens, private keys.
  Use fake values.
- **Final responsibility stays with the human** — *you* review the diff,
  understand the critical logic, confirm tests cover the real risk.

**Defense in depth example** — an agent tries `DELETE FROM users WHERE active=0;`
on prod. Two *independent* layers stop it:
- Human-in-the-loop → someone approves before it runs (decision time).
- Read-only connection → the DB refuses the write (execution time).

## The high-quality task card

A good agent task specifies:
- **Goal** — the user-visible behavior to achieve.
- **Repro** — steps to see the current wrong behavior.
- **Scope** — relevant files; and what must stay unchanged (API, routes, design system).
- **Acceptance** — add/update tests, the exact commands to run (`npm test`, `lint`),
  final report of what changed + verification results.

## Failure modes (spot these)

- Vague wish → lots of effort, doesn't solve the problem.
- One giant change → review costs more than doing it yourself.
- No test → explanation stands in for evidence.
- Secrets / prod access handed to the model.
- Rules too abstract or too long to follow.
- Reviewing AI code for *style only* — not error paths, edge cases, security.

## Self-test

- **Agent vs chat model — biggest difference?** Agent calls tools to observe and
  change external state, then iterates on the result.
- **Why must a task include acceptance criteria?** Otherwise AI only produces
  plausible output; you can't objectively judge "done."
- **What goes in a rules file?** Specific, executable, project-relevant commands,
  boundaries, style, security requirements.
- **AI code passed one test — is it safe?** Not necessarily — check test coverage,
  edge cases, security, and the diff scope.
