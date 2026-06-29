# Lecture 4 — Debugging and Profiling

Official notes: https://missing.csail.mit.edu/2026/debugging-profiling/

The shared goal: turn *"something feels wrong"* into a problem that's
**reproducible, observable, and measurable** — no guessing.

## Debugging is a science, not random guessing

Treat a bug like a lab experiment:

1. **Reproduce** — make it happen reliably (same input, same steps). Can't
   reproduce it → collect info (logs, input, env, version, seed) before changing anything.
2. **Hypothesize** — a specific, testable guess at the cause.
3. **Experiment** — change ONE thing, observe the result.
4. **Root cause** — which assumption was actually broken?
5. **Fix** — the smallest change that works.
6. **Verify** — re-run and confirm; add a regression test so it can't come back.

Ruling a hypothesis *out* (e.g. "would `export` fix it? no, it's all one shell")
is real progress, not a dead end.

Worked example from this course: a script printed a blank where a number should be.
Root cause was a **typo** in a variable name (`$fruit_cout` vs `fruit_count`) — and
an undefined variable expands to blank (Lecture 2). Fix the typo, verify it prints `8`.

## Tools for seeing what's happening

| Tool | What it is | Best for |
|------|-----------|----------|
| print / echo | drop in a line to show a value | quick local "is this what I think?" checks |
| logging | structured, leveled messages kept over time | long-running / remote / production issues |
| debugger | pause the program, inspect variables & call stack | complex control flow, "where exactly does it break?" |
| assert | state an invariant; fails loudly if violated | catching "this should never happen" |

Don't leave temporary `print`s in committed code — promote important ones to logging.

## System-boundary problems → tracing

Some bugs aren't in your code but at the **system boundary**: missing files,
permissions, DNS, sockets, subprocesses, env vars. A language debugger only sees
the language layer; tracing tools show what's happening between the program and the OS.

| Problem | Tool | Shows |
|---------|------|-------|
| file not found | `strace -e openat` / `dtruss` | the path it actually tried to open |
| port in use | `lsof -i :PORT` | which process holds the port |
| network failing | `curl -v`, `dig`, `tcpdump` | DNS, handshake, response stages |
| permission denied | `ls -l`, `id`, errno | user, group, mode bits |

(These are mostly Linux/Mac tools — conceptual for now on Git Bash/Windows.)

## Profiling — measure before you optimize

The #1 mistake: rewriting the code you *assume* is slow. **Measure first** — a
profiler shows where time actually goes, so you fix the real bottleneck.

Simplest profiler: **`time`** in front of any command.

```bash
time sleep 2       # real ~2s
time ls            # real tiny (a fraction of a second)
```

- **`real`** = wall-clock time you actually waited (the key one).
- `user` / `sys` = CPU time in your program vs. the OS.
- `time` is a stopwatch around the command — it measures the command, not itself.

Distinctions that matter when optimizing: latency vs throughput, CPU-bound vs
I/O-bound, average vs the slow tail (P95/P99). A change that lowers the average but
raises P95 may not be a win. Tools: `perf`, `py-spy`, language profilers, `iotop`.

## Benchmarking — one measurement is noisy

A single `time` result isn't trustworthy (CPU load, caches, cold start, background
processes all affect it). Running `time ls` repeatedly gives slightly different
numbers — that wobble is the **noise floor**. If two versions differ by less than
the noise, one run can't tell them apart.

Reliable benchmarking:

| Practice | Why |
|----------|-----|
| fix the input | not measuring data differences |
| warm up (discard first run) | first run is "cold" (empty caches) |
| repeat many times | one run is noise; many show the real picture |
| look at the spread, not just average | an average hides a slow worst case |
| record the environment | so others can reproduce |

Dedicated tool: **`hyperfine`** does warmup + repeats + statistics:
`hyperfine --warmup 3 'commandA' 'commandB'`.

**Takeaway:** "it felt faster" and "one run was faster" aren't proof. Reliable
performance claims come from repeated, controlled measurement.

## A deliverable bug report (template)

```
Symptom:   what error? expected vs actual?
Reproduce: version / command / input / environment
Hypotheses: H1, H2, ...
Experiment: what changed? what was observed?
Root cause: which invariant broke?
Fix:        the smallest change
Verify:     which tests/commands? new regression test?
```
