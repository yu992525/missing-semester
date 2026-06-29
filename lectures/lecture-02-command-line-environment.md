# Lecture 2 — Command-line Environment

Official notes: https://missing.csail.mit.edu/2026/command-line-environment/

The theme: making terminal sessions **controllable, recoverable, remote-capable,
and customizable** — so work doesn't get lost or break mysteriously.

## Exit codes (return codes)

Every command leaves behind a hidden number when it finishes:

- **0 = success**, any other number = failure (counterintuitive but key).
- See the last command's code with `echo $?`.
- The error *message* (on screen) and the exit *code* are **separate channels** —
  judge success by the exit code, not by whether something printed.
- `grep` uses its exit code to report matches: `0` = found, `1` = not found.

Conditional chaining uses exit codes:

| Syntax | Meaning |
|--------|---------|
| `A && B` | run B only if A **succeeded** |
| `A \|\| B` | run B only if A **failed** |
| `if cmd; then ... else ... fi` | run `then` if cmd succeeded, else `else` |

```bash
mkdir new && echo "made it"          # echo runs only if mkdir worked
ls /nope || echo "that failed"       # echo runs only because ls failed
if grep -q apple fruits.txt; then echo "have apples"; else echo "none"; fi
```

## Environment variables

A variable is a named box holding a value. **No spaces around `=`**:

```bash
name="Celia"          # NOT  name = "Celia"  (bash would think 'name' is a command)
echo "hello $name"    # $ = "give me the value"
```

Built-in ones: `$HOME`, `$PWD`, `$PATH`. (On Git Bash/Windows `$USER` is often
empty — use `$USERNAME`. Lesson: a blank variable usually means "not set on this
system," not a different meaning.)

### export: shell variable vs environment variable

- A plain variable lives **only in your current shell**.
- `export NAME` promotes it to an **environment variable**, passed down to every
  program (child process) you launch.
- `$PATH` works everywhere precisely because it's exported.

```bash
myvar="hi"
bash -c 'echo $myvar'   # blank — child shell can't see it
export myvar
bash -c 'echo $myvar'   # hi — now the child inherits it
```

### Quoting: who expands the `$`? ("first reader wins")

- **`'single quotes'`** → keep `$` literal (no expansion).
- **`"double quotes"`** → expand `$` now.
- Your current shell reads the line first: double quotes → **parent** expands;
  single quotes → the value passes through literally, so a **child** expands it
  (and only sees exported vars).

### Command substitution

- **`$(command)`** runs the command and drops its **output** in place.
- **`$name`** reads a **variable**; **`$(cmd)`** runs a **command** — different jobs.
- `\$` escapes a dollar sign to print it literally inside double quotes.

```bash
echo "Today is $(date)"     # runs the date command
echo "Today is $date"       # variable 'date' (undefined) -> blank
echo 'Today is $date'       # single quotes -> literal $date
echo "Today is \$date"      # backslash escape -> literal $date
```

## Signals & job control

A running program is a **process**. Signals are messages you send it.

| Action | What it does |
|--------|--------------|
| `Ctrl-C` | SIGINT — interrupt/stop the foreground program |
| `Ctrl-Z` | SIGTSTP — suspend (freeze) it, return to the shell |
| `jobs` | list jobs in this shell |
| `bg` | resume a suspended job in the **background** |
| `fg` | bring a job back to the **foreground** |
| `cmd &` | start a command in the background from the start |
| `kill %1` | signal job number 1 (polite SIGTERM by default) |
| `kill -9 PID` | SIGKILL — forceful, last resort |

- **Job number** `[1]` = local to this shell. **PID** = system-wide process id.
- **SIGTERM (polite)** lets a program clean up (save files, release locks) before
  exiting. **SIGKILL (`-9`)** can't be caught — instant, may leave a mess. Try
  polite first; use `-9` only when something won't die.

```bash
sleep 300 &     # background
jobs            # [1] Running
kill %1         # [1] Terminated
```

## Aliases — shortcuts you invent

```bash
alias ll='ls -l'      # no spaces around =
alias dc='cd'         # typo-fixer
alias                 # list all aliases
```

Aliases vanish when you close the terminal — unless saved in a startup file.

## Dotfiles & .bashrc — make things permanent

Bash reads `~/.bashrc` automatically every time it starts. Anything in there runs
in every new session. Files starting with `.` are **hidden** (`ls -a` to see them);
config files are nicknamed **dotfiles**.

```bash
ls -a ~                                  # reveals .bashrc, .gitconfig, .ssh, ...
echo "alias ll='ls -l'" >> ~/.bashrc     # APPEND (>>) — never > or you erase it!
source ~/.bashrc                         # re-read the file in the current shell
```

Use `~/.bashrc` (anchored to home), not `.bashrc` (relative to where you are) —
bash only reads the one in your home directory.

## Shell functions — aliases on steroids

Functions take input (`$1`, `$2`, ...), run multiple steps, and use logic.

```bash
greet() { echo "Hello, $1!"; }
greet Celia                  # Hello, Celia!

marco() { MARCO_DIR="$(pwd)"; echo "Saved $MARCO_DIR"; }
polo()  { cd "$MARCO_DIR"; }
# marco remembers a spot; polo teleports back from anywhere.
```

No `export` needed here — both functions run in the **same shell**, so the
variable is already shared (export is only for reaching child processes).

## Globbing (wildcards)

The **shell** expands these into matching filenames **before** the command runs
(so the program receives a list of real names — the argv idea from Lecture 1):

| Pattern | Matches |
|---------|---------|
| `*` | any number of any characters |
| `?` | exactly one character |
| `{a,b}` | each listed option |

```bash
ls *.txt        # all .txt files
ls ?.txt        # one-character name + .txt (a.txt yes, fruits.txt no)
echo *          # the shell turns * into the full file list
```

## SSH & tmux (concept — need a remote server to practice)

- **SSH** = log into another computer's shell over the network (`ssh user@host`).
  Use key pairs (`ssh-keygen`) instead of passwords; nickname servers in
  `~/.ssh/config` so `ssh gpu-box` just works.
- **tmux** = keep a session alive on the server even if your connection drops.
  `tmux new -s work`, detach with `Ctrl-b d`, reattach with `tmux attach -t work`.
  Essential for long remote tasks.
