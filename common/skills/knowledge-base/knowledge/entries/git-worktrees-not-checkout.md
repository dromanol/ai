---
name: git-worktrees-not-checkout
description: In GitHub repos, start each branch in its own git worktree under <repo>.worktrees\ — never switch branches in the main checkout
type: working-style
---

In every GitHub project, new work goes in a **git worktree**, not a `git checkout`/`git switch` of the
main clone.

- **Main clone stays on the default branch** (`master`/`main`), clean. Don't switch it to feature branches.
- **Location:** sibling folder, outside the repo: `<parent>\<repo>.worktrees\<leaf>`, e.g.
  `C:\_dd\git\dd-trace-dotnet.worktrees\my-fix` for branch `dani/aap/my-fix` (use the last segment of
  the branch name as `<leaf>`; follow the repo's branch-naming convention for the branch itself).
- **New branch:** `git fetch origin` then
  `git worktree add -b <branch> ..\<repo>.worktrees\<leaf> origin/<default>`.
- **Existing branch / PR review:** `git worktree add ..\<repo>.worktrees\<leaf> <branch>` (fetch first).
- **Work there:** run builds, tests and commits from the worktree path; state the path in replies.
- **Cleanup when merged/abandoned:** `git worktree remove <path>` (then `git branch -d <branch>`);
  `git worktree prune` for stale entries. Check `git worktree list` before creating — a branch can only
  be checked out in one worktree at a time.
- Don't use Claude Code's `EnterWorktree` default (`.claude/worktrees` inside the repo) unless asked.

**Why:** parallel tasks/sessions don't stomp on each other's working tree, uncommitted work never has to
be stashed to switch context, and keeping worktrees outside the repo avoids polluting `git status`,
recursive globs and builds of the main clone.

**Caveats:** each worktree has its own untracked build output (first build is a full build), and
gitignored local files (`.env`, local settings) aren't copied — set them up per worktree if needed.
