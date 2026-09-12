---
name: git-rebase
description: 現在のブランチを指定したブランチの上に rebase する。競合時は git-resolve-conflicts に委譲する。ユーザーが「rebaseして」「mainの上に乗せ替えて」「最新のmainに追従したい」など、ブランチの再構成や rebase を求めている場合は必ずこのスキルを使う。
---

# Skill: git-rebase

## Arguments

Invoked as `git-rebase <target>`. Interpretation of `<target>`:

- A local branch name like `main` → rebase onto that local `<target>` as-is.
- A remote-tracking branch name like `origin/main` → assumed already fetched; used as-is. **This skill never runs `git fetch` itself.** If the local `origin/main` may be stale, warn the user once before proceeding (no automatic fetch).

If `<target>` is omitted, ask what to rebase onto.

## Preconditions (always check before running)

- Run `git status` and confirm there are no uncommitted/unstaged changes in the working tree.
- If there are changes, do not start the rebase — ask the user to commit or stash (never stash/commit without asking).

## Steps

1. Run `git status` to confirm the working tree is clean.
2. If `<target>` is a remote-tracking branch like `origin/...`, do not fetch — warn once beforehand that the local ref may be stale.
3. Run `git rebase <target>`.
4. Evaluate the result.
   - Success: confirm with `git log --oneline -5`, report, and finish.
   - Conflict: run git-resolve-conflicts, then continue through to `git rebase --continue` automatically once resolved.
5. After the rebase completes, check the final working-tree state with `git status --short`.

## Rules (never violate)

- Running `git rebase --abort` without the user's explicit instruction.
- Rebasing `main` or another shared branch itself (i.e., becoming the branch that gets rebased onto). This skill always confirms it is moving the *current* branch.
- Skipping the warning that unpushed commits will be rewritten.

## Output

- Before rebasing, state in one line which branch is being moved onto which.
- After completion, summarize the commit graph, and report if a force push will be needed to push.
