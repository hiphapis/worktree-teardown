> [!IMPORTANT]
> This repository has moved to [hiphapis/claude-plugins](https://github.com/hiphapis/claude-plugins/tree/main/plugins/worktree-teardown) and is archived. Install: `/plugin marketplace add hiphapis/claude-plugins` → `/plugin install worktree-teardown@hiphapis`

# worktree-teardown

[한국어](README.ko.md)

A Claude Code skill that audits a git worktree **before you delete it**, so nothing goes with it.

Deleting a worktree rarely loses commits, because git refuses to remove a dirty worktree. What actually gets lost is elsewhere:

- **gitignored assets**: agent handoff notes, plans, generated images/video, a `.env` you edited only there
- **skipped wrap-up**: the issue is still "in progress", the workspace status never changed, docs never updated
- **stale records**: notes say "unmerged" when it was merged, or the reverse

The skill checks 11 items, grades each one, runs the wrap-up you approve, re-audits, and only then **shows** you the delete command. It never deletes anything itself.

## Install

As a plugin (recommended):

```
/plugin marketplace add hiphapis/worktree-teardown
/plugin install worktree-teardown@worktree-teardown
```

Or copy the skill folder by hand:

```bash
git clone https://github.com/hiphapis/worktree-teardown
cp -r worktree-teardown/skills/worktree-teardown ~/.claude/skills/
```

## Use

Inside the worktree (or anywhere in the repo), ask:

> can I delete this worktree?
> 이 워크트리 지워도 돼?

Or invoke it directly with `/worktree-teardown:worktree-teardown`. To audit a different worktree, give its path or branch.

## What it checks

| # | Item | Blocks deletion when |
| --- | --- | --- |
| 1 | Uncommitted tracked changes | any |
| 2 | Untracked files (not ignored) | any |
| 3 | Unpushed commits | commits have no remote copy |
| 4 | Unmerged commits (squash-aware via `git cherry`) | you choose merge / keep / discard |
| 5 | Stash, including orphans | branch-owned stash and the branch is discarded |
| 6 | Non-regenerable ignored assets (blacklist, `.env` judged by `diff`) | preserve-class assets exist |
| 7 | Issue or PR state (GitHub) | per project labels |
| 8 | Orca workspace status, crossed with the issue | issue closed but status not `completed` |
| 9 | Child worktrees (Orca) | any |
| 10 | Canonical-doc drift: removed facts still stated in docs | found in canonical paths |
| 11 | Record consistency (judgment, never auto-blocks) | — |

Grades: 🔴 BLOCKED · 🟡 check · ✓ OK · **N/A**. N/A means a tool your project doesn't use (no Orca, non-GitHub remote), and it doesn't block. A tool you *do* use whose call failed is 🟡, because unverified never counts as OK.

End states: `SAFE` (delete command shown) · `BLOCKED(n)` · `HANDOFF` (merge first, after the wrap-up) · `ABORT`.

## Requirements

- `git` (required)
- `gh`, logged in, for the issue checks. Without it, those checks are N/A (no GitHub) or 🟡 (GitHub, but the call failed).
- Orca (optional). Used if the worktree is Orca-managed; plain `git worktree` otherwise.
- `superpowers:finishing-a-development-branch` (optional). Used for the merge hand-off if installed.

## Project policy (optional)

Labels, completion steps, which docs are canonical and which are history: these differ per project. Put them in `.claude/worktree-teardown.md` in your repo ([example](examples/worktree-teardown.md), [format](skills/worktree-teardown/references/project-config.md)). Without that file, the skill looks for a lifecycle rule in your `CLAUDE.md` / `AGENTS.md` / `CONTRIBUTING.md`, and falls back to generic defaults.

## Safety contract

Never deletes the worktree · never commits for you · never merges, rebases or pushes · never deletes or moves files (preserves by copying) · never closes an issue without approval, and never closes one as `not planned` · never reports a failed check as OK.

The full rulebook is [`skills/worktree-teardown/references/audit-rules.md`](skills/worktree-teardown/references/audit-rules.md).

## License

MIT
