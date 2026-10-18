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


## Question 3: Finding the Commit That Introduced a Bug

`git bisect` searches between a known good commit and a known bad commit. Git selects intermediate commits for testing, and each good or bad result narrows the search range.

I start with a clean working tree and a repeatable test for the bug.

Suppose the current commit is broken and `a1b2c3d` is a known good commit:

git bisect start
git bisect bad HEAD
git bisect good a1b2c3d

Git checks out a candidate commit. I run the relevant test.

If the bug is absent, I run:

git bisect good

If the bug is present, I run:

git bisect bad

I execute only the command matching the test result and repeat this process until Git identifies the first bad commit.

If a candidate cannot be tested, I can use:

git bisect skip

Skipping commits can prevent Git from identifying one exact introducing commit.

After recording the reported commit ID, I inspect it and exit bisect:

git show b2c3d4e
git bisect reset

Here, `b2c3d4e` represents the reported bad commit. `git bisect reset` ends the search and returns to the original checkout.

For example, if a login test passed in an older release but fails now, bisect helps locate the commit that introduced the failure. With a consistent good-to-bad transition, it requires roughly logarithmic testing instead of checking every commit.

Bisect identifies the introducing commit; it does not fix the bug.

## Question 4: Reviewing Changes Before Committing

### 4.1 Checking repository status and changes

git status
git diff -- README.md
git diff --cached -- README.md

`git status` reports staged, unstaged, and untracked changes.

| Command | Comparison | Meaning |
| --- | --- | --- |
| `git diff` | Working tree versus staging area | Changes not yet staged |
| `git diff --cached` | Staging area versus HEAD | Changes prepared for the next commit |

`--staged` is an alias for `--cached`.

The staging area, also called the index, stores the selected snapshot for the next commit. Editing a file after staging it does not automatically update that snapshot.

Therefore, `README.md` can have both staged and unstaged changes at the same time.

Ordinary diffs do not show the contents of untracked files in `tmp/`.

### 4.2 What an ordinary commit includes

git commit -m "Update README"

An ordinary commit without `-a` or file arguments records the staging area.

Therefore:

- Previously staged README changes are included.
- Later unstaged README edits remain in the working tree.
- Other already-staged changes are also included.
- Untracked files in `tmp/` are not included.

The reason is that an ordinary commit records the index, not every current working-tree edit.

### 4.3 Ignoring the temporary directory

Add the following line to `.gitignore` in the repository root:

/tmp/

This ignores the root-level `tmp` directory and its contents.

To record the ignore rule:

git add .gitignore
git commit -m "Ignore temporary files"

Any other changes already staged would also be included, so I inspect the staging area first if this should be a separate commit.

Ignore rules do not stop tracking files that are already tracked.

For an already-tracked temporary file, I would add the ignore rule and remove the file from the index while preserving its local copy:

git rm --cached -- tmp/cache.txt
git add .gitignore
git commit -m "Stop tracking temporary cache"

This additional removal is unnecessary for the assignment's temporary files because they are already untracked.


## Question 5: Resolving Merge Conflicts in Teamwork

### 5.1 Why the conflict occurs

The branches changed the same line differently relative to their common ancestor. Git cannot automatically determine the intended result.

Two people editing the same file does not always cause a conflict. Non-overlapping changes can usually merge automatically.

### 5.2 Identifying conflicts

While the merge is stopped on `main`:

git status
git diff --name-only --diff-filter=U

Open `README.md`. With the default conflict style, the conflicting versions appear between markers:

<<<<<<< HEAD
This project provides a web programming tutorial.
=======
This project includes practical Git exercises.
>>>>>>> feature-description

The upper block belongs to the current branch, `main`. The lower block comes from the incoming branch, `feature-description`.

### 5.3 Preserving both changes and completing the merge

Replace the entire marked block with coherent combined text:

```text
This project provides a web programming tutorial with practical Git exercises.


Remove all conflict markers and review the complete file.

Check for leftover markers and whitespace problems, stage the resolution, and review it:

```bash
git diff --check
git add README.md
git diff --cached --check
git diff --cached -- README.md
git status

Fix any reported problems before proceeding. Status must show no unmerged paths.

Then complete the merge:

git commit -m "Merge feature-description and resolve README conflict"
git status

### 5.4 Canceling the merge

Before completing it:

git merge --abort

Given the initially clean working tree, this restores the pre-merge state.



## Question 6: Undoing Changes

These are independent situations, not consecutive steps.

### 6.1 Discard unstaged changes in notes.txt

git restore -- notes.txt

By default, this restores the working-tree file from the staging area.

In this scenario, the staged version matches the last commit, so the unwanted edits are discarded.

Effects:

- Working tree: `notes.txt` is restored.
- Staging area: unchanged.
- Commit history: unchanged.
- Other files: unchanged.

The discarded unstaged edits are not saved by this command.

### 6.2 Unstage notes.txt while preserving its edits

git restore --staged -- notes.txt

With `--staged`, the default source is `HEAD`.

This restores only the file's staging-area entry to its last committed version. The edits remain in the working-tree file.

Effects:

- Working tree: edits are preserved.
- Staging area: the file's entry matches `HEAD`.
- Commit history: unchanged.

The `--` separator indicates that the following argument is a file path.

### 6.3 Reverse an incorrect commit already shared on main

Assume I am on the shared `main` branch with a clean working tree. The example ID `b2c3d4e` identifies the incorrect ordinary commit:

git revert b2c3d4e
git push origin main

Here, `origin` is the assumed remote name. If Git opens an editor for the revert commit message, save and close it to complete the commit.

Under the stated no-conflict assumption, revert creates a new commit that applies the inverse of the selected commit's changes.

Effects:

- Working tree: updated to the reversal result.
- Staging area: matches the new commit.
- Commit history: gains a new reversal commit; existing commits remain unchanged.

This is appropriate because teammates have already received the original commit.

Deleting the commit with a history-rewriting reset and force-push would make their existing histories disagree with the remote and could disrupt subsequent work. Revert instead produces a normal new commit that teammates can receive without rewriting their existing history.

| Situation | Command | Working tree | Staging area | History |
| --- | --- | --- | --- | --- |
| Discard unstaged edits | `git restore -- notes.txt` | Restored from index | Unchanged | Unchanged |
| Unstage but keep edits | `git restore --staged -- notes.txt` | Edits preserved | File restored from HEAD | Unchanged |
| Reverse a shared commit | `git revert b2c3d4e` | Updated | Matches new commit | New reversal commit |


