# Lecture 6 — Packaging & Shipping Code

Official notes: https://missing.csail.mit.edu/2026/shipping-code/

**Big question:** your code runs on *your* machine — how do you make it run
reliably on *someone else's* (a teammate's, a server, a user's)?

## The enemy: "works on my machine"

Code that runs for you but breaks for others. It breaks because the other setup
differs in **four ways**:
1. **Environment / OS** (e.g. PowerShell vs Git Bash — commands assumed bash)
2. **Language / runtime** (they don't have Python, or the wrong version)
3. **Libraries + versions** (the code imports libraries they don't have, or a
   different version that behaves differently)
4. **Config / secrets** (API keys, passwords the code needs)

"Shipping" is the craft of killing this problem.

## Dependencies & virtual environments (fixes #3)

- Real code uses outside **libraries**; you must specify *which* ones and *which
  versions*.
- **Problem (dependency hell):** if libraries are installed globally, two projects
  needing different versions of the same library **clash** — you can only have one
  global version at a time.
- **Virtual environment** = an isolated, **per-project box** holding just that
  project's libraries at just the right versions. Each project gets its own.
  - Trade-off: uses *more* disk space (duplicated libraries) to guarantee projects
    don't break each other. (Isolation is the point — NOT saving space.)
- You list dependencies in a **spec file** (`pyproject.toml`, `requirements.txt`,
  `package.json`) so anyone can recreate your setup.

## Lockfiles & reproducibility

- A **lockfile** (`uv.lock`, `package-lock.json`) records the **exact** versions
  (down to the patch number), so anyone rebuilds a **byte-for-byte identical** setup.
- Rule of thumb: **apps pin exact versions; libraries allow ranges.**

## Containers (Docker) — seals the WHOLE environment (fixes #1–3 at once)

A virtual env isolates libraries; a **container** packages the *entire* environment
— mini-OS + language + libraries + your code — into one bundle that runs
**identically anywhere**. The ultimate "works on my machine" killer: you ship the
whole setup with your code (like a standardized shipping container).

Four words (don't mix them up):

| Word | What it is | Analogy |
|------|-----------|---------|
| **Docker** | the tool/software | — |
| **Dockerfile** | recipe: instructions to build | 说明书 / recipe |
| **Image** | the built, frozen package (OS+lang+libs+code) — NOT a photo, NOT just the code | frozen meal-kit |
| **Container** | a running instance of an image | the meal, heated & running |

Pipeline: `Dockerfile ──build──▶ Image ──run──▶ Container` (run many containers
from one image).

```dockerfile
FROM python:3.11-slim   # base: mini-OS that already has Python 3.11
WORKDIR /app
COPY pyproject.toml ./  # copy the dependency list
RUN pip install ...     # install libraries
COPY . .                # copy in the code
CMD ["python", "app.py"] # what to run when the container starts
```

- The OS/shell/language is sealed into the image (via `FROM ...`).
- Secrets are **NOT** baked in — injected at runtime (see below).
- **Docker Compose** runs several containers together (app + database + cache).
- Containers are **lighter than VMs** — they share the host OS kernel, so they
  start in seconds instead of booting a whole OS.

## Config & secrets (fixes #4)

**Principle:** configuration lives *outside* code. The *same code* runs in
dev/staging/prod by changing **config only, never code.**

- **Config** = things that vary by environment (DB URLs, ports, feature flags).
- **Secrets** = sensitive config (API keys, tokens, passwords).

**Iron rule: NEVER commit secrets to version control.**
- A secret is a **live key** to your real systems (your database, paid services,
  your whole account) and your **money** — not just "access to your code" (code
  being public is usually fine). Think **blueprint (code) vs. house key (secret)**.
- Leaked keys are scraped by bots within *minutes*; cloud keys can rack up huge
  bills. Applies to personal projects too — bots don't care how small it is.
- Once committed, a secret is in the **permanent history** — there's no "hide"
  toggle. Prevent it, don't hide it.

**The `.env` pattern:**
- `.env` holds real secrets locally, and is **`.gitignore`d** (never committed).
- `.env.example` **is** committed, with placeholder values, so others know what to
  supply: `API_KEY=your-api-key-here`.
- Secrets load at runtime from **environment variables** or a secret manager.
- If a secret leaks: **rotate it immediately** (generate new, revoke old).

## Versioning — Semantic Versioning (SemVer)

Label releases `MAJOR.MINOR.PATCH` so people know the *risk* of upgrading:

| Part | Bump when… | Example |
|------|-----------|---------|
| **PATCH** | bug fix only | `2.4.1 → 2.4.2` |
| **MINOR** | new feature, old stuff still works (backward-compatible) | `2.4.1 → 2.5.0` |
| **MAJOR** | breaking change (old usage stops working) | `2.4.1 → 3.0.0` |

Bumping MINOR resets PATCH to 0 (e.g. `1.6.3 + feature → 1.7.0`); bumping MAJOR
resets both (`→ 2.0.0`).

## Publishing

Upload your package to a **registry** so others install with one command:
- **PyPI** (Python → `pip install`), **npm** (JS), **crates.io** (Rust).
- Container images → **Docker Hub**, **ghcr.io**.
- You can also install straight from a GitHub repo.
- **Trust:** you're running strangers' code — **package signing** helps verify it's
  authentic.

## Release checklist (from the notes)

1. Bump version + update changelog → 2. run test/lint/typecheck/build →
3. build the artifact/image → 4. keep secrets out → 5. push to registry →
6. deploy to staging → 7. then production.
