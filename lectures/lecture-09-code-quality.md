# Lecture 9 — Code Quality

Official notes: https://missing.csail.mit.edu/2026/codequality/

**Big idea:** code quality is **risk management, not aesthetics.** Its job is to make
*future* changes less dangerous — surfacing errors earlier, cheaper, and more
reliably. Writing code the first time is easy; the risk is everything after (bug
fixes, new features, dependency upgrades, a new maintainer a year later).

> Core principle: **automate the mechanical stuff so human review is freed up for
> behavior, architecture, edge cases, and risk.** A check that's too slow, too noisy,
> or too false-positive-heavy just gets bypassed.

## The quality stack — each tool kills a different risk

| Tool | Risk it kills | Examples |
|---|---|---|
| **Formatter** | style arguments + diff noise (layout) | Black, Prettier, gofmt, rustfmt |
| **Linter** | suspicious patterns / bug smells | Ruff, ESLint, Clippy, ShellCheck |
| **Type checker** | interface misuse, wrong data shapes | mypy, pyright |
| **Tests** | behavior breaking (regressions) | pytest |
| **Coverage** | finding *blind spots* (NOT a quality score) | coverage.py |
| **CI** | "passes on my machine only" | GitHub Actions |
| **Pre-commit** | mechanical junk before it's committed | pre-commit + ruff |

## Formatter vs Linter

- **Formatter** answers *"is this laid out consistently?"* — one right answer, so it
  **auto-fixes**. No human should argue about spaces in review.
- **Linter** answers *"does this look like a bug waiting to happen?"* — unused vars,
  dangerous defaults, high complexity, bad imports. **Often needs human judgment.**
- **The mixed-PR fix (L8 callback):** a formatter running automatically means nobody
  ever hand-reformats 2,000 lines inside a feature PR — that noise never gets created,
  so a behavior diff stays a behavior diff and the reviewer sees the real change.
  **A tool beats discipline** (no one has to *remember* to split).
- **Keep lint rules few but useful.** A linter that fires 200 trivial warnings gets
  ignored *entirely* — including the one that mattered. A noisy alarm is a **low-
  precision alarm**: too many false positives and people stop trusting all of them
  (boy-who-cried-wolf). An alarm no one trusts is worse than no alarm.

## Tests and the testing pyramid

Tests **lock in behavior** so future changes can't silently break it. Ordered by a
speed/realism trade-off:

| Level | Checks | Strength | Weakness |
|---|---|---|---|
| **Unit** (most) | one function in isolation | fast, pinpoints failure | can over-mock, drift from reality |
| **Integration** (some) | modules together | catches boundary bugs | more setup |
| **End-to-end** (few) | whole user path | closest to real usage | slow, fragile |

Pyramid shape = **many fast unit tests at the base, few slow e2e at the top.** Invert
it and the suite gets so slow/flaky that people stop running it (disabled alarm again).

**Regression test for a bugfix (L7 red-then-green):**
1. Write a test that reproduces the bug → it **fails (red)**.
2. Fix the code until it **passes (green)**.
3. The test lives in **CI forever** → if anyone ever reintroduces the bug, CI catches
   it automatically. (a) fixes it today; (b) fixes it *forever* — a future dev doesn't
   even need to know the bug existed; it's structurally unable to return.

## CI (Continuous Integration)

**A robot that automatically runs your whole quality checklist every time someone
pushes** — on a clean, standard cloud machine, not anyone's laptop. On GitHub that's
**GitHub Actions**.

Why it's powerful:
1. **Same checks, same environment, for everyone** → kills "works on my machine" (L6).
2. **Runs automatically — nobody can "forget"** to run the tests.
3. **It's a gate** — a PR can't merge until CI is green, so broken code is blocked
   *before* it enters main.

## Pre-commit and "shift left"

**Pre-commit** runs the *cheap, fast* checks (format, lint, secret scan) **instantly
on your own machine at commit time** — so a missing space is fixed in 2 seconds
instead of after a 5-minute CI cycle + a red PR.

Layers, cheapest/earliest first:

| Layer | Runs | Checks |
|---|---|---|
| editor save | editor | format, instant diagnostics |
| **pre-commit** | your machine, at commit | format, lint, **secret scan** |
| pre-push | your machine, at push | fuller tests |
| **CI** | cloud robot, after push | full tests, build, matrix |

**Shift left = catch problems as early as possible, because early = cheap.** Also:
`detect-private-key` as a pre-commit hook is L7's "never commit secrets" **automated**
— the tool enforces the rule so you don't have to remember it. Use the **same command
entry point** locally and in CI (`make check`) so local and CI can't drift apart.

## Coverage — a blind-spot finder, NOT a quality score

Coverage = what % of code lines actually **ran** during tests. The trap: it says
nothing about whether the test **checked** the result. A test that asserts *nothing*
still counts as "covered." So "95% coverage = bug-free" fails from both ends:
- the uncovered part is untested, **and**
- the covered part might be tested meaninglessly (ran, but verified nothing).

> Use coverage to find gaps worth filling (error handling, edge cases, permission
> logic) — never chase the *number*, or you get assertion-free tests that look safe.

## Regex/search on code — find with it, don't blindly edit with it

`grep`/regex is great for *finding* patterns across a big codebase, but **editing code
with regex is dangerous** — plain text doesn't understand the language, so it can match
inside comments, strings, or nested syntax and break things. Use search to find
**candidates**; make real changes with **language-aware tools (AST codemods)**, then
**run the tests and review the diff.**

## Automation templates (from the deck)

- **`make check`** as a single entry point: `setup / format / lint / test / check`
  (`check: lint test`) — same command locally and in CI.
- **pre-commit**: ruff + ruff-format + `check-yaml`, `end-of-file-fixer`,
  `trailing-whitespace`, `detect-private-key`.
- **GitHub Actions**: on `pull_request` + push to `main`, run `uv sync --frozen` then
  `make check`.

## Failure modes

- Formatter changes mixed into a behavior PR.
- High coverage but assertion-free tests.
- CI and local commands differ → problems only appear remotely.
- Too many lint rules / false positives → the team ignores them.
- Only testing the happy path — not errors, permissions, edges, concurrency, migrations.
- Bulk-editing with regex, then not running tests or reading the diff.

## Self-test

- **Why automate the formatter?** It settles mechanical style issues, killing review
  noise and subjective arguments.
- **Is 100% coverage bug-free?** No — coverage shows lines *executed*, not that
  assertions are meaningful or requirements correct.
- **pre-commit vs CI?** pre-commit = fast local interception of mechanical issues;
  CI = full checklist in a uniform environment, as a merge gate.
- **Why is bulk regex editing dangerous?** Plain-text matching ignores language
  semantics — it can wreck comments, strings, or complex structures.
