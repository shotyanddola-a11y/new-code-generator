# Advanced Git Commands Guide

A comprehensive reference for advanced Git operations including stash, cherry-pick, revert, and reset.

---

## Table of Contents
1. [git stash](#git-stash)
2. [git cherry-pick](#git-cherry-pick)
3. [git revert](#git-revert)
4. [git reset](#git-reset)
5. [Workflow Examples](#workflow-examples)

---

## git stash

The `git stash` command temporarily saves changes that you're not ready to commit, allowing you to switch branches or work on other tasks without losing your work.

### Basic Syntax
```bash
git stash [save "message"]
git stash list
git stash apply [stash@{n}]
git stash pop [stash@{n}]
git stash drop [stash@{n}]
git stash clear
```

### Common Use Cases

#### 1. Save Work in Progress
```bash
# Save current changes with a descriptive message
git stash save "WIP: implementing user authentication"

# Or use the shorthand (Git 2.13+)
git stash push -m "WIP: implementing user authentication"
```

#### 2. List All Stashes
```bash
git stash list
# Output:
# stash@{0}: WIP: implementing user authentication
# stash@{1}: Fix bug in payment processing
# stash@{2}: Update documentation
```

#### 3. Restore Stashed Changes
```bash
# Apply stash without removing it
git stash apply stash@{0}

# Apply and remove the stash (pop)
git stash pop stash@{0}

# Apply latest stash (stash@{0})
git stash pop
```

#### 4. Stash Specific Files
```bash
# Stash only specific files
git stash push -m "UI changes" src/components/Button.jsx src/styles/main.css

# Stash all unstaged changes
git stash push -u -m "Unstaged work"
```

#### 5. Remove Stashes
```bash
# Delete a specific stash
git stash drop stash@{2}

# Delete all stashes
git stash clear
```

### Real-World Example
```bash
# You're working on feature-x but need to switch to fix-critical-bug
git status
# On branch feature-x
# Changes not staged for commit:
#   modified: auth.js
#   modified: utils.js

# Save your work
git stash save "WIP: auth refactoring"

# Switch to fix-critical-bug and make fixes
git checkout fix-critical-bug
git commit -am "Fix critical bug in payment processing"

# Return to feature-x
git checkout feature-x

# Restore your work
git stash pop
```

---

## git cherry-pick

The `git cherry-pick` command allows you to apply specific commits from one branch to another. It's useful for applying bug fixes or features to multiple branches.

### Basic Syntax
```bash
git cherry-pick <commit-hash>
git cherry-pick <commit1> <commit2> <commit3>
git cherry-pick <commit1>..<commit2>
git cherry-pick --continue
git cherry-pick --abort
```

### Common Use Cases

#### 1. Apply a Single Commit
```bash
# Cherry-pick commit abc123 from another branch
git cherry-pick abc123

# This creates a new commit on the current branch with the same changes
```

#### 2. Apply Multiple Commits
```bash
# Apply three specific commits
git cherry-pick abc123 def456 ghi789

# Apply a range of commits (exclusive of start, inclusive of end)
git cherry-pick abc123..ghi789

# Apply a range inclusively
git cherry-pick abc123^..ghi789
```

#### 3. Handle Conflicts
```bash
# Start cherry-pick
git cherry-pick abc123

# If conflicts occur, resolve them manually in your editor
# Then stage the resolved files
git add .

# Continue the cherry-pick
git cherry-pick --continue

# Or abort if you change your mind
git cherry-pick --abort
```

#### 4. Cherry-pick with Options
```bash
# Cherry-pick without committing (for manual review)
git cherry-pick -n abc123

# Edit the commit message during cherry-pick
git cherry-pick -e abc123

# Keep the original author and date
git cherry-pick abc123
```

### Real-World Example
```bash
# Bug fix needed across multiple versions
# On release-v2.0 branch
git log --oneline
# 5e3f2a1 Fix critical security vulnerability
# a1b2c3d Add new feature

# Switch to release-v1.9
git checkout release-v1.9

# Apply the security fix
git cherry-pick 5e3f2a1
# [release-v1.9 f7d8e9c] Fix critical security vulnerability
#  1 file changed, 5 insertions(+)

# Switch to main
git checkout main

# Apply the same fix
git cherry-pick 5e3f2a1
```

---

## git revert

The `git revert` command creates a new commit that undoes the changes from a previous commit. Unlike reset, it preserves history and is safe for shared branches.

### Basic Syntax
```bash
git revert <commit-hash>
git revert <commit1> <commit2>
git revert <commit1>..<commit2>
git revert --no-commit <commit-hash>
git revert --abort
git revert --continue
```

### Common Use Cases

#### 1. Revert a Single Commit
```bash
# Revert the most recent commit
git revert HEAD

# Revert a specific commit
git revert abc123

# This opens your editor to confirm the revert commit message
```

#### 2. Revert Multiple Commits
```bash
# Revert a range of commits (creates multiple revert commits)
git revert abc123^..def456

# Revert commits without automatically committing
git revert -n abc123 def456 ghi789
# Make additional changes if needed
git commit -m "Revert multiple problematic commits"
```

#### 3. Handle Conflicts During Revert
```bash
# Start revert
git revert abc123

# If conflicts occur, resolve them manually
git add .

# Continue the revert
git revert --continue

# Or abort if needed
git revert --abort
```

#### 4. Revert with Edit Options
```bash
# Revert without opening editor
git revert abc123 --no-edit

# Edit the revert commit message
git revert -e abc123
```

### Real-World Example
```bash
# Bad feature was merged into main
git log --oneline
# 7a6b5c4 Merge feature: new payment system
# 3d2e1a0 Add tests
# f1e2d3c Initial commit

# Discover the feature causes issues
# Revert the merge commit
git revert -m 1 7a6b5c4
# [main 9z8y7x6] Revert "Merge feature: new payment system"
#  5 files changed, 12 deletions(-)

# The feature is now undone, but the history is preserved
# This is safe to push to shared branches
git push origin main
```

### Key Differences: Revert vs Reset
```bash
# REVERT: Creates new commit, preserves history
git revert abc123
# Safe for shared/public branches
# History remains intact

# RESET: Modifies history, removes commits
git reset --hard abc123
# DANGEROUS on shared/public branches
# History is lost
```

---

## git reset

The `git reset` command moves the HEAD pointer and can modify the staging area and working directory. Use with caution on shared branches as it rewrites history.

### Basic Syntax
```bash
git reset [--soft | --mixed | --hard] [<commit>]
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

### Three Reset Modes

#### 1. Soft Reset (--soft)
- Moves HEAD pointer
- Keeps changes in staging area
- Keeps changes in working directory
- Safe - no data is lost

```bash
# Undo last commit but keep changes staged
git reset --soft HEAD~1

# Changes remain in git add area, ready to recommit
git status
# Changes to be committed:
#   modified: file.js
```

#### 2. Mixed Reset (--mixed) - Default
- Moves HEAD pointer
- Unstages changes
- Keeps changes in working directory
- Allows re-staging selectively

```bash
# Undo last commit and unstage changes (default behavior)
git reset HEAD~1
# or explicitly:
git reset --mixed HEAD~1

# Changes are now unstaged
git status
# Changes not staged for commit:
#   modified: file.js
```

#### 3. Hard Reset (--hard)
- Moves HEAD pointer
- Discards all changes
- **Permanently deletes** uncommitted work
- **DANGEROUS** - use with caution

```bash
# Undo last commit and discard all changes
git reset --hard HEAD~1

# All changes are lost
git status
# On branch main
# nothing to commit, working tree clean
```

### Common Use Cases

#### 1. Undo Last Commit (Keep Changes)
```bash
# Soft reset - keep staged
git reset --soft HEAD~1

# Mixed reset - unstage (default)
git reset HEAD~1
```

#### 2. Undo Multiple Commits
```bash
# Undo last 3 commits, keep changes
git reset --soft HEAD~3

# Undo last 3 commits, discard changes
git reset --hard HEAD~3

# Undo last 3 commits using commit hash
git reset --soft abc123
```

#### 3. Reset to Specific Commit
```bash
# Reset to a specific commit
git reset --hard abc123

# This moves HEAD to abc123 and discards everything after
```

#### 4. Unstage Files
```bash
# Unstage a specific file
git reset HEAD myfile.js

# Unstage all files
git reset HEAD
```

#### 5. Recover After Hard Reset
```bash
# View recent operations
git reflog
# abc1234 HEAD@{0}: reset: moving to abc123
# def5678 HEAD@{1}: commit: Add new feature
# ghi9012 HEAD@{2}: commit: Fix bug

# Recover to the state before reset
git reset --hard HEAD@{1}
```

### Real-World Example
```bash
# Scenario: Made multiple commits but want to restructure them

git log --oneline
# 3c4d5e6 Add comments
# 2b3c4d5 Fix typos
# 1a2b3c4 Add main feature

# Reset to main feature, keep all changes
git reset --soft 1a2b3c4

# Now all changes are staged together
git status
# Changes to be committed:
#   modified: app.js (includes all 3 commits)

# Recommit with better structure
git reset HEAD
git add app.js
git commit -m "Add main feature"
git add comments.js
git commit -m "Add comments"
```

---

## Workflow Examples

### Scenario 1: Emergency Bugfix on Release Branch
```bash
# Currently on feature-x
git stash save "WIP: new feature"

# Switch to release branch
git checkout release-v2.0

# Create hotfix branch
git checkout -b hotfix/critical-bug

# Make and commit fixes
git commit -am "Fix critical bug"

# Get commit hash
git log -1 --oneline
# a1b2c3d Fix critical bug

# Cherry-pick to main
git checkout main
git cherry-pick a1b2c3d

# Return to feature-x
git checkout feature-x
git stash pop
```

### Scenario 2: Undoing Bad Merge
```bash
# Bad merge was applied to main
git log --oneline
# abc1234 Merge pull request #42: bad feature
# def5678 Previous commit

# Revert the merge (safe for shared branch)
git revert -m 1 abc1234

# Or reset if it's a private branch
git reset --hard def5678
```

### Scenario 3: Restructuring Commits
```bash
# Multiple small commits that should be combined
git log --oneline
# 1a2b3c4 Add validation
# 2b3c4d5 Add error handling
# 3c4d5e6 Add logging

# Reset soft to combine them
git reset --soft HEAD~3

# Restage and recommit with better message
git add .
git commit -m "Add comprehensive user validation with error handling and logging"
```

### Scenario 4: Cherry-picking Hotfix to Multiple Branches
```bash
# Create hotfix on main
git checkout -b hotfix/security-patch main
git commit -am "Fix security vulnerability"
git log -1 --oneline
# z9y8x7w Fix security vulnerability

# Cherry-pick to release-v2.0
git checkout release-v2.0
git cherry-pick z9y8x7w

# Cherry-pick to release-v1.9
git checkout release-v1.9
git cherry-pick z9y8x7w
```

---

## Best Practices

### When to Use Each Command

| Command | Use Case | Safe for Shared? |
|---------|----------|-----------------|
| `stash` | Temporarily save work | ✅ Yes |
| `cherry-pick` | Apply specific commits | ✅ Yes (creates new commits) |
| `revert` | Undo changes publicly | ✅ Yes |
| `reset` | Restructure local commits | ⚠️ No (rewrites history) |

### Safety Tips

1. **Always use `git revert` on shared branches** - it preserves history
2. **Use `git reset` only on private/local branches**
3. **Save work with `git stash`** before switching branches
4. **Know your reflog** - `git reflog` can recover lost commits
5. **Test before pushing** - especially with cherry-pick and revert
6. **Communicate with team** - before force-pushing changes that rewrite history

### Command Cheat Sheet

```bash
# Quick reference
git stash                          # Save current work
git stash pop                      # Restore latest stash
git cherry-pick abc123             # Apply commit to current branch
git revert abc123                  # Create new commit undoing changes
git reset --soft HEAD~1            # Undo commit, keep changes staged
git reset --hard abc123            # Reset to commit, discard changes
git reflog                         # View all recent operations
```

---

## Conclusion

These advanced Git commands provide powerful tools for managing your codebase:

- **`git stash`** - Temporarily save work without committing
- **`git cherry-pick`** - Apply specific commits to other branches
- **`git revert`** - Safely undo changes in shared repositories
- **`git reset`** - Restructure commits on local branches

Master these commands to work more efficiently with Git, especially in team environments where history preservation is important.
