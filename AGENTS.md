# AGENTS.md

<!-- agent-flow:start v0.3.0 -->
<!-- Reusable workflow block. Same in every repo. Do not edit per project. -->

## 1. How we work

There are two ways to work. Pick one at the start.

- **Feature → slice.** New behavior, API or contract change, data model change, auth/permissions/payments, frontend + backend together, or a design decision. Use the slice workflow below.
- **Small change → normal chat.** Clear bug fix, text/style tweak, small refactor in one area, config or test fix. Work on a branch, run the checks, and merge following the project's git policy (section 10). No slice.

If unsure, ask the human. If a small change grows into a feature, stop and suggest a slice.

## 2. Slices and phases

The lead follows `workflow/guide.md` in every phase.

A slice is one feature in `slices/<name>/`, on its own branch and worktree (section 10). Read, search, and edit its files in that worktree, not in the main checkout. Use full paths into the worktree: tools may skip the git-ignored `.worktrees/` folder when they search.

```
spec → plan → ready → implementing → integration → preview → done
                       marks on any phase: (blocked), (cancelled)
```

```
slices/<name>/
  spec.md        what the feature does              human approves
  plan.md        how it will be built               human approves, with tasks.md and preview.md
  tasks.md       every task: structure and content
  preview.md     how the human tries the slice
  status.md      the only state of the slice, kept by the lead
  reviews/       review files
  notes.md       technical ideas from the spec talk, optional, not approved
```

Templates: `templates/`. Each template's comments hold its rules; read the template before writing or changing a document.

Every approval is an annotated git tag, e.g. `approval/<name>/spec/R001`, holding the approval record.

Ideas for later go to `backlog.md` at the project root when a slice is done (`templates/backlog.md`).

Contracts, the data model, and other living docs stay at the project level (see section 9). A slice edits them on its own branch and lists them under `touches` in `plan.md`. The human approves the slice's change to them, not the whole file.

### Reviews

| Review | When |
|---|---|
| Spec check | every real gap as a question, before the human approves the spec |
| Plan review | plan, tasks, and preview together, always in full |
| Task review | every task with risk `core` or `normal`; not `low` |
| Integration review | always |

## 3. Roles

| Role | Does | Instructions |
|---|---|---|
| Human | decides, approves, previews | — |
| Lead | the main chat agent: designs with the human, writes docs, runs the slice, keeps the status, reports | `workflow/roles/lead.md`, `workflow/guide.md` |
| Worker | implements exactly one task, from its brief | `workflow/roles/worker.md` |
| Plan reviewer | asks questions about the spec before approval; checks plan, tasks, and preview together before coding | `workflow/roles/plan-reviewer.md` |
| Code reviewer | checks one task, or the whole slice | `workflow/roles/code-reviewer.md` |

If you are the main chat agent, you are the lead. Read your role file before starting slice work.

## 4. Hard rules

- Only a human approves. Never approve for them, even if asked to "just continue".
- Never change an approved spec, plan, or contract without the human (`workflow/guide.md` → "Changing an approved spec or plan"). If it looks wrong, stop and ask. The one exception: fix tasks and a wider `touches` in an approved `tasks.md`, as `workflow/guide.md` → "Changes to an approved tasks.md" says.
- Never move or delete an approval tag. Never edit a submitted review.
- Only the lead changes status, and only in the slice's `status.md`. File headers describe what a file is; they hold no status.
- Merge or push only as the project's git policy allows (section 10), and for a slice only after the human approves the result.
- Never force-push. Never rewrite shared history.

## 5. Stop rules

Stop, mark the task (for a plan or integration review: the slice) `blocked`, and tell the human in plain words when:

- a task fails its checks 3 times since its last review
- a review (plan, task, or integration) still does not pass after 3 rounds
- the task is unclear, or the spec, plan, or contract looks wrong
- you need a new dependency, a storage format change, secrets, or CI changes

## 6. Status

- **Read:** the slice's `status.md`. It is the overview and the state of the whole slice: what needs the human, documents, tasks, findings, preview results, and the log. Format and the meaning of each status word: `templates/status.md`.
- **Write:** only the lead, after every step: `status.md` with a Log line, and a commit.
- **Checks:** a task moves to review or done only after the checks pass (section 9: the auto-fix commands, then the checks). Only a full run counts, not single tests.

## 7. Workflow files

| File | Holds |
|---|---|
| `workflow/guide.md` | the lead's steps for every phase |
| `workflow/roles/` | what each role does |
| `templates/` | one template per document; its comments hold the document's rules |
| `workflow/adapters/` | how each agent tool finds the roles |
| `workflow/VERSION` | the agent-flow version |
| `.agents/skills/flow-setup/` | setting up or updating the workflow in a repo |

<!-- agent-flow:end -->

## 8. Project

agent-flow is a spec-driven workflow for AI coding agents: this file, the
guide, the roles, the templates, and the setup skill. It works with Claude
Code, Codex, and Cursor.

We build agent-flow using its own workflow.

## 9. Stack profile

| Part | Folder | Tech |
|---|---|---|
| Workflow files | `AGENTS.md`, `workflow/`, `templates/`, `.agents/skills/` | Markdown |

Tools per role (the lead is the tool you chat in):

| Role | Tool | How to start |
|---|---|---|
| Worker | Codex | `codex exec -C <worktree> --sandbox workspace-write - < <prompt file>` |
| Code reviewer | Claude Code | subagent `flow-code-reviewer` |
| Plan reviewer | Claude Code | subagent `flow-plan-reviewer` |

Connections: (none)

Setup (run in each new task or update worktree, for the parts it touches): (none)

Always allowed in any task: (none)

Auto-fix: (none)

Checks: (none)

Run locally: (none)

Test dependencies: (none)

Known gaps (chosen in setup; revisit when possible):

- Workflow files (Markdown) have no automatic checks. Reviewers check them by hand. Chosen by the maintainer, 2026-10-17.
- No pre-commit hook and no CI job yet. Tracked in `backlog.md`. Chosen by the maintainer, 2026-10-17.

## 10. Project rules

### Git policy

- Base branch: `main`
- Slice branch: `slice/<name>` from `main`, worktree `.worktrees/<name>/`
- Task branch: `task/<name>-NN` from the newest slice branch, worktree `.worktrees/task-<name>-NN/`
- Update branch: `update/<name>` from the newest slice branch, worktree `.worktrees/update-<name>/` (to update a slice from `main`)
- Small changes: `fix/<short-name>` from `main`
- After the human approves the result: the lead merges into `main` with a merge commit (`git merge --no-ff`). It does not push.
- Remote: `origin` is the public repo and gets only releases: never push to it. At a release, the human asks, and the project is pushed there as one squashed commit. A local pre-push hook refuses anything else (it lives only in `.git/hooks/`: install it again after cloning).
- Slices update from `main` before the integration review, and again just before merging, in an update worktree (`workflow/guide.md` → 5. Integration).
- Parallel slices: merge one at a time.
- Commits: the lead commits after every step that changes something: `<name>: <what happened>`, and task work as `<name>/NN: <summary>`.

### Other rules

How work is done in this project. Each rule names who follows it and when.

- Everyone: write in plain, simple English. Short sentences.
- Everyone: keep everything generic. No company names, internal URLs, or secrets in this repo.
