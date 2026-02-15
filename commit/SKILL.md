---
name: commit
description: Group working tree changes into atomic commits with well-crafted messages.
---

# Commit

Analyze changes. Group into atomic commits. Draft messages. Present for review. Never auto-commit. Never push.

A commit is a permanent record of a decision. It will be read by people who have no context — debugging at 2am, reviewing a PR, or running `git blame` years later. Every commit should be understandable in isolation.

## 1. Gather state

```bash
git status --porcelain
git diff --stat
git diff --cached --stat
git log --oneline -10
```

Check log for existing scope and trailer conventions. Match scope/trailer style but always enforce conventional commit format for subjects.

## 2. Analyze diffs

Read full diffs, not just file names. A file list tells you what was touched; the diff tells you what actually changed and why. Check surrounding code when intent isn't obvious from the hunk — the context often reveals the motivation.

## 3. Group into commits

Core test: **what should be rolled back together?** If reverting one change without the other would leave the codebase broken or inconsistent, they belong in the same commit.

Each commit: atomic, minimal, bisectable, coherent. If you need "and" to describe it, it's two commits.

**Why this matters:** `git bisect` finds regressions by testing individual commits. `git revert` undoes a single commit. `git cherry-pick` moves a single commit between branches. All three break when commits mix unrelated changes.

Heuristics:

- Schema/migration with model code, not consuming feature — migrations must be revertable independently
- Renames alone — `git log --follow` only tracks renames when no content changes in the same commit
- Refactors before the feature they enable — keeps the feature commit small and reviewable
- Tests with the code they test — a commit that changes behavior without updating tests is a lie
- Deps with the code that needs them — a dep added without its consumer (or vice versa) breaks the build at that commit
- Split hunks in same file via `git add -p` when they serve different logical changes

**All changes get a commit — staged, unstaged, untracked.** Unrelated changes get their own commit. The only exclusions are artifacts that don't belong in the repo at all (build outputs, `.env` with real secrets, generated files).

If OS/editor artifacts are present (`.DS_Store`, `.idea/`, `.vscode/`, `*.swp`, `Thumbs.db`) and not gitignored, propose a .gitignore commit + `git rm --cached` for any tracked ones.

### Commit ordering

Order determines the story a reader follows and affects bisectability:

1. Required order — migration before code that uses it, refactor before feature
2. Foundational before dependent — shared util before consumers
3. Larger scope before smaller — core change before edge case fix
4. Housekeeping last — .gitignore, formatting, cleanup

### File ordering within a commit

Same principle applied to the file list: required first, foundational first, big to small. A reviewer reads top to bottom — lead with the important change, not the test.

## 4. Draft messages

### Subject

Format: `<type>(<scope>): <description>`. Scope optional. Imperative mood ("add" not "added"). 50 char target, 72 max. No period. Specific and greppable. Intent, not mechanism.

Types: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `chore`, `build`, `ci`, `revert`.

**Why imperative:** the subject completes "If applied, this commit will \_\_\_." This is git's own convention (`Merge branch`, `Revert`).

**Why specific:** `git log --oneline` is how most people read history. Every line must carry meaning. "Fix bug" wastes the reader's time. "Fix null pointer when user has no email" tells them whether this commit is relevant to their problem.

### Body

Blank line after subject. Wrap 72 chars (git tooling assumes this width).

The body is for the reader who has no context — a different team, a future maintainer, someone debugging a production incident. Write for them.

- **State the problem first.** What was wrong, missing, or needed? Why does this change exist at all?
- **Explain the domain.** If the change involves a business rule, protocol, or system behavior that isn't common knowledge, briefly explain it.
- **Describe behavioral changes.** "Previously X. Now Y." Be explicit about what users, callers, or dependent systems will observe differently.
- **Justify the approach.** Why this solution over alternatives? What trade-offs were made?
- **Don't rely on issue trackers.** They migrate, go offline, get deleted. The commit message is the only artifact guaranteed to travel with the code forever.

### Trailers

`Refs:`, `Fixes:`, `Closes:`, `Co-authored-by:` as appropriate. `BREAKING CHANGE: <description>` when changing public API, CLI, config format, wire protocol, or any consumer contract — always include migration path. Match the project's existing trailer convention.

### Bad subjects

- "Fix bug" — which? where?
- "Update code" — content-free
- "Add null check" — what not why
- "Implement X, fix Y, refactor Z" — three commits
- "Use HashMap instead of TreeMap" — mechanism not intent
- "Refactor to use async/await" — describe behavior change

## 5. Housekeeping checks

Before presenting, flag:

- Secrets (`.env`, keys, tokens) — warn, do not commit. Secrets in git history are extremely difficult to fully remove.
- Large binaries — warn. Git stores every version forever; large binaries bloat the repo permanently.
- Lockfile changes without manifest changes — warn. Usually means an accidental regeneration.
- Irreversible migrations — warn. No rollback path if something goes wrong in production.
- Generated files that shouldn't be tracked — exclude.

## 6. Present

No preamble. No postamble after the actions. Jump straight into the first commit.

Status shorthand: `M` modified, `A` added, `D` deleted, `R` renamed, `U` untracked.

```md
## <type>(<scope>): <description>

<body>

<trailers>

M `path/to/file` — verb phrase reason

## <type>(<scope>): <description>

<body>

<trailers>

R `old/path` → `new/path` — verb phrase reason

### Warnings

- <terse warning>

## Commits

[0] <type>(<scope>): <description>
[1] <type>(<scope>): <description>

## Actions

- **all** — `all`, `commit`, `lgtm`, `y`
- **specific** — `0`, `1, 3`
- **range** — `0-2`
```

Adapt numbers to actual commit count. For a single commit, only show:

```md
## Actions

- **all** — `all`, `commit`, `lgtm`, `y`
```

## Output rules

- Commit messages are the permanent artifact — full sentences, proper grammar, written for someone with no context.
- Everything else is disposable scaffolding — terse, no filler.
- File lists: `<STATUS> \`path\` — verb phrase`.
- Verb phrases only: "update import path" not "import path update."
- Warnings section only when there are warnings.
- Flag uncertainty: "`utils.py` touches X and Y — placed in 0, may want to split."
- No preamble before the first commit. No postamble after the actions.
- Numbered summary always present — scannable index and reference for selective commits.

## Pre-commit hooks

Pre-commit hooks run automated checks (linting, formatting, type checking) before git finalizes a commit. They can reject commits that don't pass.

If `git commit` fails due to hooks:

1. Show the hook output
2. If auto-fixable (formatting, linting): apply fixes, re-stage, retry
3. If not auto-fixable: present the error, ask user whether to fix or skip (`git commit --no-verify`)
4. Never `--no-verify` without explicit approval — hooks exist for a reason

## Responses

- **Commit** (`all`, `commit`, `y`, `0, 2`, `0-2`) — execute all or specified commits, leave unspecified as-is
- **Edit** (any other reply) — apply changes, re-present full plan

## Execution

1. Stage precisely per commit (`git add -p` or `git add <file>`)
2. Commit with exact message from plan (`git commit -F -` to avoid shell escaping)
3. Repeat for each commit
4. Show `git log --oneline` of new commits as confirmation
