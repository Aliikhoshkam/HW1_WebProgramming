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