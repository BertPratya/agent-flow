# AGENTS.md

<!-- agent-flow:start v0.2.0 -->
<!-- Reusable workflow block. Same in every repo. Do not edit per project. -->

## 1. How we work

There are two ways to work. Pick one at the start.

- **Feature → slice.** New behavior, API or contract change, data model change, auth/permissions/payments, frontend + backend together, or a design decision. Use the slice workflow below.
- **Small change → normal chat.** Clear bug fix, text/style tweak, small refactor in one area, config or test fix. Work on a branch, run the checks, and merge following the project's git policy (section 10). No slice.

If unsure, ask the human. If a small change grows into a feature, stop and suggest a slice.

## 2. Slices and phases

A slice is one feature in `slices/NNNN-<name>/`, on its own branch and worktree (section 10). Read, search, and edit its files in that worktree, not in the main checkout. Use full paths into the worktree: tools may skip the git-ignored `.worktrees/` folder when they search.

```
spec → plan → ready → implementing → integration → preview → done
                       any phase → blocked | cancelled
```

When each phase starts: `workflow/formats/lifecycle.md`.

```
slices/NNNN-<name>/
  spec.md        what the feature does        human approves
  plan.md        how it will be built         human approves
  notes.md       technical ideas from the spec interview, optional, not approved
  status.md      overview and state, kept by the lead
  tasks/         one file per task
  reviews/       review reports
  history/       approved snapshots
```

Templates: `workflow/templates/`.

Ideas parked for later go to `backlog.md` at the project root when a slice is done.

Contracts, the data model, and other living docs stay at the project level (see section 9). A slice edits them on its own branch and lists them under `touches` in `plan.md`. The human approves the slice's change to them, not the whole file.

### Tracks

The lead proposes a track in `plan.md`. The human approves it with the plan.

- **complex:** the slice changes a contract or the data model, touches more than one part (e.g. backend and frontend), or involves security.
- **simple:** everything else.

| | simple | complex |
|---|---|---|
| Spec check | every real gap as a question, before the human approves the spec; the lead asks them in rounds | same |
| Plan review | light: checks 1-4 in `workflow/roles/plan-reviewer.md` | full |
| Task review | core tasks only | every task |
| Integration review | always | always |

## 3. Roles

| Role | Does | Instructions |
|---|---|---|
| Human | decides, approves, previews | — |
| Lead | the main chat agent: designs with the human, writes docs, runs the slice, reports | `workflow/roles/lead.md` |
| Worker | implements exactly one task | `workflow/roles/worker.md` |
| Plan reviewer | asks questions about the spec before approval; checks spec, plan, contract, tasks together before coding | `workflow/roles/plan-reviewer.md` |
| Code reviewer | checks one task, or the whole slice | `workflow/roles/code-reviewer.md` |

If you are the main chat agent, you are the lead. Read your role file before starting slice work.

## 4. Hard rules

- Only a human approves. Never approve for them, even if asked to "just continue".
- Never change an approved spec, plan, or contract. If it looks wrong, stop and ask.
- Never edit anything in `history/`, or a submitted review.
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

- **Read:** the slice's `status.md`. It is the overview and the state of the whole slice: what needs the human, documents, tasks, findings from all reviews, and the log. Format: `workflow/formats/status.md`.
- **Write:** only the lead, after every step: `status.md` with a dated Log line, and a commit (`workflow/roles/lead.md` → Keeping the status).
- **Lifecycles** of slice phases, documents, tasks, findings, and checks: `workflow/formats/lifecycle.md`. Read it before changing any status.
- **Document checks** for spec, plan, and task files: `workflow/formats/document-checks.md`.
- **Checks:** a task moves to review or done only after the checks pass (section 9: the auto-fix commands, then the checks). Only a full run counts, not single tests.

## 7. Skills

| Skill | Use when |
|---|---|
| `flow-setup` | checking whether a repo is ready for the workflow, and setting up what is missing: workflow files, checks, test dependencies, hooks, CI |
| `flow-spec` | turning an idea into an approved spec |
| `flow-plan` | writing plan, contract, and tasks, and getting them reviewed |
| `flow-implement` | running the tasks with workers and reviewers |
| `flow-review` | starting a review (spec, plan, task, or whole slice) in a fresh context, and acting on its verdict |

<!-- agent-flow:end -->

## 8. Project

agent-flow is a spec-driven workflow for AI coding agents: this file, the
roles, templates, formats, and skills. It works with Claude Code, Codex,
and Cursor.

We build agent-flow using its own workflow.

## 9. Stack profile

| Part | Folder | Tech |
|---|---|---|
| Workflow files | `AGENTS.md`, `workflow/`, `.agents/skills/` | Markdown |

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
- Slice branch: `slice/NNNN-<name>` from `main` (or from another slice's branch when stacked), worktree `.worktrees/NNNN-<name>/`
- Task branch: `task/NNNN-NN` from the newest slice branch, worktree `.worktrees/task-NNNN-NN/`
- Update branch: `update/NNNN` from the newest slice branch, worktree `.worktrees/update-NNNN/` (to update a slice from `main`)
- Small changes: `fix/<short-name>` from `main`
- After the human approves the result: the lead merges into `main`. It does not push.
- Remote: `origin` is the public repo and gets only releases: never push to it. At a release, the human asks, and the project is pushed there as one squashed commit. A local pre-push hook refuses anything else (it lives only in `.git/hooks/`: install it again after cloning).
- Slices update from `main` before the integration review, and again just before merging, in an update worktree (`flow-implement` step 4.1).
- Parallel slices: merge one at a time.
- Commits: the lead commits after every step that changes something: `NNNN: <what happened>`, and task work as `NNNN/NN: <summary>`.

### Other rules

How work is done in this project. Each rule names who follows it and when.

- Everyone: write in plain, simple English. Short sentences.
- Everyone: keep everything generic. No company names, internal URLs, or secrets in this repo.
