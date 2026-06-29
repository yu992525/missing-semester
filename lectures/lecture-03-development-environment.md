# Lecture 3 — Development Environment and Tools

Official notes: https://missing.csail.mit.edu/2026/development-environment/

The theme: your dev environment is a **feedback loop** — understand → change →
run → verify → commit. Good tools make that loop fast and reliable. This lecture
is more about *mindset and tools* than terminal commands.

## Modal editing (Vim)

Vim is the course's example for one idea: **editing actions combine like a
language.** Vim has *modes*, and the same key does different things in each.

- **Normal mode** (the default when Vim opens): keys are **commands** (move,
  delete, copy). You are NOT typing text.
- **Insert mode**: you type text like a normal editor. Enter with `i`, leave with `Esc`.

**Survival skills (learn these first so you're never stuck):**
- `Esc` → always returns to Normal mode (your reset button).
- `:wq` then Enter → save and quit.
- `:q!` then Enter → quit WITHOUT saving (the escape hatch).
- Bottom-left shows `-- INSERT --` in insert mode, blank in normal mode.

**The transferable idea — actions + objects combine:**
- `dw` = **d**elete + **w**ord
- `dd` = delete a line
- `x` = delete a character
- `u` = undo (safety net)
- `.` = repeat last edit
- Move with arrow keys (fine!) or `h j k l` (left/down/up/right).

You don't have to use Vim daily. The takeaway: think "what object, what action" —
smaller, cleaner edits.

**More modes (from the official lecture — beyond the two essentials above):**
- **Visual** — select text (then act on the selection).
- **Replace** — type over existing text.
- **Command-line** — the `:` commands like `:wq`, `:q!`, search/replace.

The official lecture also walks through a worked example fixing a "fizzbuzz" program.

## LSP — how an editor "understands" code

Smart features (jump to definition, error underlines, autocomplete, safe rename)
don't come from the editor itself. A separate **Language Server** understands the
language, and the editor talks to it over a standard protocol, **LSP** (Language
Server Protocol).

- Editor = receptionist (UI, your typing).
- Language server = the expert who actually knows the language.
- LSP = the standard "phone line" between them.

Because the protocol is standard, **any editor can use any language's server**.
Practical payoff: when the editor "isn't smart" (no autocomplete, can't jump to a
definition), the problem is usually the **language server not running** or the
project not set up right — not the editor's theme or UI.

## A project should have a single "front door"

A project should answer — without anyone remembering — how to **install, run,
test, format, and check** it. Write those answers down (README, Makefile, package
scripts) instead of leaving them in someone's shell history.

```
make setup   # install dependencies
make dev     # run locally
make test    # run tests
make format  # auto-format
make lint    # static checks
```

Standard entry points help new people, CI, and AI agents all work the same way.

## AI tools — three kinds, by how much they do

| Type | Does | Best for | Risk |
|------|------|----------|------|
| Autocomplete | finishes your current line/snippet | boilerplate, predictable code | looks right but edge/variable wrong |
| Chat | explains, drafts, compares options | understanding, design | drifts from your real project |
| Agent | reads files, runs commands, edits across project | multi-file work, running tests | scope creep — touches the wrong things |

**Golden rule:** AI's weakness is a vague goal and no way to verify. When you
assign a task, state: the **goal**, the **relevant files**, what **not to touch**,
and **how to check** it worked. When you accept output, **read the diff and run
the tests** — don't trust that code "looks right."

## IDE extensions (from the official lecture)

- **Dev containers** — package the whole dev environment so it runs the same
  anywhere (no "works on my machine").
- **Remote SSH development** — edit code that lives on a remote server, through
  your local editor (connects to Lecture 2's SSH).
- **Collaborative editing** — multiple people editing the same file live, like
  Google Docs for code.

> Note: the "single front door / Makefile" and "development loop" sections below
> are solid engineering practice (from the study notes) that complement the
> official lecture, whose headline topics are Vim, LSP, AI tools, and IDE extensions.

## The development loop — small steps, feedback, evidence

Keep shortening: **understand → change → verify → commit.** Make each change small
enough to explain, review, and undo. When a test fails, a small change makes it
obvious what caused it. Don't make a giant change and pray — small step, run, see
it pass, commit, repeat.
