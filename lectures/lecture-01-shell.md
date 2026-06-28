# Lecture 1 — Introduction to the Shell

Official notes: https://missing.csail.mit.edu/2026/course-shell/

## Core ideas

- **Terminal** = the window you type in (just a display).
- **Shell** = the program running inside it that understands commands (e.g. `bash`).
- **Program** = what the shell launches when you type a command (`ls`, `git`, `python`).
- On Windows, use **Git Bash** (a Unix shell), not PowerShell or cmd.
  - Tell you're in bash: the prompt ends in `$` and uses `/` paths like `~/Desktop`.
  - Tell you're in PowerShell: prompt starts with `PS` and ends in `>`.

## Getting around

| Command | What it does |
|---------|--------------|
| `pwd`        | print where you are (current path) |
| `ls`         | list files here |
| `ls -l`      | list with details (permissions, size, date) |
| `cd folder`  | go into a folder |
| `cd ..`      | up one level (relative — from where you are) |
| `cd`         | jump straight home (from anywhere) |

`cd ..` and `cd` are **not** the same — `cd ..` goes up exactly one level,
`cd` always teleports home. They only look alike from `~/Desktop` because
Desktop's parent happens to be home.

Path symbols: `.` = here, `..` = up one, `~` = home, `/` = root (top of everything).

## Reading `ls -l` permissions

```
drwxr-xr-x  1  kouyu  197609  0  Aug 13 2022  ipa
└───┬────┘                                    └─ name
    └─ type + permissions
```

- First letter: `d` = folder, `-` = file.
- Then three `rwx` groups: owner / group / everyone.
- `r` = read, `w` = write, `x` = execute. A `-` means "not allowed."

## The big three: redirection and pipes

| Symbol | Meaning | Memory hook |
|--------|---------|-------------|
| `>`  | redirect output **to a file** — overwrites it! | one arrow = replace |
| `>>` | redirect output **to a file** — appends to end | two arrows = add on |
| `\|`  | pipe output **to another program** | conveyor belt between tools |

- `>` is a little dangerous: it wipes the file first, no undo, no recycle bin.
- Direction matters: `grep word file` **reads from** the file;
  `... > file` **writes to** the file.

## $PATH

`$PATH` is the list of **folders** the shell searches to find a program when
you type its name. It is *not* a list of programming languages. When you type
`ls`, bash walks `$PATH` left to right until it finds a program called `ls`.

```bash
echo $PATH
```

(Variable names are case-sensitive: `$PATH` works, `$path` is empty.)

## The Unix philosophy

Small single-purpose tools, combined with pipes:

- `ls` lists, `grep` filters, `wc` counts, `sort` sorts.
- None of them try to do everything — you assemble them like LEGO.
- A pipe is a chain of dependent steps; each stage only sees what the
  previous one passed. **Order matters.**

`grep` is a *filter* — it can't produce data, only filter what flows in:

```bash
grep word filename     # search INSIDE an existing file
something | grep word  # search whatever is piped in
```

## Putting it together (the capstone)

Count how many items on the Desktop are folders, save the number to a file:

```bash
ls -l                          # see everything
ls -l | grep "^d"              # keep only lines starting with d (folders); ^ = "starts with"
ls -l | grep "^d" | wc -l      # count them
ls -l | grep "^d" | wc -l > folder_count.txt   # save the number
```

Read it as: *list in detail → keep only folders → count → save to a file.*

## Speed tricks

| Key | Does |
|-----|------|
| `↑` / `↓`   | scroll through previous commands |
| `!!`        | rerun the last command |
| `Ctrl + R`  | search command history |
| `Tab`       | autocomplete filenames and commands |
| `Ctrl + C`  | cancel the currently running command |

Copy-paste in a terminal isn't `Ctrl+C/V` (Ctrl+C means "cancel"). Use
right-click, `Shift+Insert`, or `Ctrl+Shift+V` to paste; select text with the
mouse to copy.
