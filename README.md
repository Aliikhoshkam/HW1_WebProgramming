# HW1_WebProgramming
The first homework of Web Programming course at AUT
# Git Assignment

The commands below describe hypothetical scenarios. Branch names and commit IDs are examples and should be replaced with actual values when executing the commands.

## Question 1: Merge vs. Rebase

### Merge

`git merge` integrates another branch into the current branch while preserving existing commits and their identities.

If the branches have diverged, Git normally creates a merge commit with two parents. If the current branch is an ancestor of the other branch, Git can perform a fast-forward merge by moving the branch pointer without creating a merge commit.

git switch main
git merge feature-description

These commands integrate `feature-description` into `main`.

Merge is suitable for integrating shared work because it preserves published history and shows where separate development paths were combined.

### Rebase

`git rebase` replays a branch's commits on a new base. Replayed commits normally receive new IDs because their parent commits change.

git switch feature-description
git rebase main

Suppose both branches start at commit A. The main branch contains A, B, C, while the feature branch contains A, D, E. After rebasing, the feature branch contains A, B, C, D', E'. The main branch itself is unchanged.

Rebase is useful for updating an unpublished feature branch before integration. It places the feature work after the latest main-branch commits and produces a linear history.

I would avoid rebasing commits that teammates already depend on without coordination, because rewriting those commits causes their histories to differ from the rewritten branch.

Both merge and rebase can produce conflicts. The main difference is that merge preserves the original branching history, while rebase changes the ancestry and identities of replayed commits.


## Question 2: Staging Selected Changes

A hunk is a block of nearby changes in a diff, including added or removed lines and surrounding context. A file may contain multiple hunks. A hunk is a textual grouping and does not necessarily correspond to one logical task.

To inspect a file's changes and selectively stage them:

git diff -- README.md
git add -p -- README.md

The `-p` option enables interactive patch selection.

| Choice | Action |
| --- | --- |
| `y` | Stage the current hunk. |
| `n` | Leave the current hunk unstaged. |
| `s` | Split the hunk into smaller hunks, when possible. |
| `e` | Manually edit the patch to select changes more precisely. |
| `q` | Stop; previously selected hunks remain staged. |
| `?` | Show help. |

After selecting the desired changes:

git diff --cached -- README.md
git diff -- README.md
git commit -m "Clarify project description"

The first diff shows staged changes. The second shows changes still outside the staging area.

Assuming no unrelated changes are already staged, the commit contains only the selected changes. Unselected edits remain in the working tree.

This helps create focused commits that each represent one logical change. Such commits are easier to review, debug, and revert. If I use `e`, I carefully inspect the staged patch before committing.