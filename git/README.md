# GIT Cheatsheet

Complete, detailed reference guide for Git version control and collaboration.

## 🎯 Git Setup & Configuration

### Initial Configuration

User Configuration:

```bash
git config --global user.name "John Doe"
git config --global user.email "john@example.com"
git config --global core.editor "vim"
git config --global init.defaultBranch main
git config --global core.autocrlf true   # Windows
git config --global core.autocrlf input  # Linux/Mac
```

View & Manage Config:

```bash
git config --list                    # All settings
git config --global --list           # Global settings
git config user.name                 # Specific value
git config --global --edit           # Edit in editor
git config --list --show-origin      # Show file location
```

### Git Aliases

Useful Aliases:

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'restore --staged'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --graph --oneline'
```

## 📦 Repository Basics

### Create & Clone

Initialize Repository:

```bash
git init                             # Create in current directory
git init my-project                  # Create with name
git init --bare my-repo.git          # Bare repository
git init --template=<template-dir>   # Use template
```

Clone Repository:

```bash
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git
git clone https://github.com/user/repo.git my-folder
git clone --depth 1 <url>            # Shallow clone (faster)
git clone --single-branch <url>      # Single branch only
git clone --filter=blob:none <url>   # Partial clone
```

### Staging & Committing

Stage Changes:

```bash
git add file.txt                    # Stage specific file
git add *.js                        # Stage by pattern
git add .                           # Stage all
git add -p                          # Interactive staging
git add -u                          # Stage tracked files only
git add -A                          # Stage everything
```

Committing:

```bash
git commit -m "Message"              # Simple commit
git commit -am "Message"             # Add tracked & commit
git commit --amend                   # Modify last commit
git commit --amend --no-edit         # Amend without message change
git commit -m "Msg" --allow-empty    # Empty commit (for CI)
git commit --date="2 days ago" -m "Message"  # Backdate commit
```

## 🌿 Branches

### Branch Management

List Branches:

```bash
git branch                          # Local branches
git branch -a                       # All branches
git branch -v                       # With last commit
git branch -vv                      # With tracking info
git branch -r                       # Remote branches only
git branch --list "feature/*"       # Pattern matching
```

Create & Switch:

```bash
git branch feature                  # Create branch
git checkout feature                # Switch branch
git checkout -b feature             # Create & switch
git switch -c feature               # Modern syntax (Git 2.23+)
git switch feature                  # Switch (modern)
git checkout --track origin/feature # Track remote branch
```

Rename & Delete:

```bash
git branch -m old-name new-name     # Rename local
git branch -m new-name              # Rename current
git branch -d feature               # Delete (safe)
git branch -D feature               # Force delete
git branch -d -r origin/feature     # Delete tracking branch
git push origin -d feature          # Delete remote
```

## 🔄 Merging & Rebasing

### Merge Operations

Basic Merge:

```bash
git checkout main
git merge feature                   # Merge feature to main
git merge --no-ff feature           # Merge with merge commit
git merge --squash feature          # Squash all commits
git merge --abort                   # Cancel merge if conflict
```

Handle Conflicts:

```bash
# Conflict markers:
<<<<<<< HEAD
Current branch code
=======
Incoming branch code
>>>>>>> feature

# After resolving:
git add resolved-file
git commit -m "Resolved conflicts"
```

### Rebase Operations

Linear History:

```bash
git rebase main                     # Rebase on main
git rebase -i HEAD~3                # Interactive rebase last 3
git rebase --continue               # Continue after conflict
git rebase --abort                  # Cancel rebase
git rebase --skip                   # Skip current commit
```

Interactive Rebase Commands:

- `pick` — Use commit
- `reword` — Change commit message
- `edit` — Modify commit
- `squash` — Combine with previous
- `fixup` — Like squash, discard log
- `drop` — Remove commit
- `exec` — Run shell command

## 📤 Remote Operations

### Push & Pull

Push Changes:

```bash
git push origin main                # Push to remote
git push origin feature             # Push feature branch
git push -u origin feature          # Push with upstream
git push -f origin main             # Force push (dangerous!)
git push origin --all               # Push all branches
git push origin --tags              # Push all tags
git push -d origin branch           # Delete remote branch
```

Pull Changes:

```bash
git pull origin main                # Fetch & merge
git pull --rebase                   # Fetch & rebase
git pull --no-ff                    # Ensure merge commit
git pull --ff-only                  # Fast-forward only
git pull origin feature             # Pull specific branch
```

### Remote Management

Manage Remotes:

```bash
git remote -v                       # List remotes
git remote add upstream https://... # Add new remote
git remote remove origin            # Remove remote
git remote rename origin myremote   # Rename
git remote set-url origin new-url   # Change URL
git remote show origin              # Detailed info
```

## ⚡ Advanced Git

### Stash & Cherry-Pick

Stashing Changes:

```bash
git stash                           # Stash current changes
git stash save "message"            # Stash with message
git stash list                      # List all stashes
git stash show stash@{0}            # Show stash contents
git stash pop                       # Apply & remove
git stash apply stash@{0}           # Apply specific
git stash drop stash@{0}            # Delete stash
git stash branch new-branch         # Create branch from stash
```

Cherry-Pick:

```bash
git cherry-pick commit-hash         # Pick single commit
git cherry-pick commit1..commit2    # Range (exclusive)
git cherry-pick commit1^..commit2   # Range (inclusive)
git cherry-pick hash1 hash2 hash3   # Multiple commits
git cherry-pick --continue          # Continue after conflict
git cherry-pick --abort             # Cancel cherry-pick
```

### Reset & Revert

Undo Changes:

```bash
# Reset types:
git reset --soft HEAD~1             # Undo, keep staged
git reset --mixed HEAD~1            # Undo, keep unstaged
git reset --hard HEAD~1             # Undo, discard changes
git reset --hard origin/main        # Reset to remote

# Revert (safe - creates new commit):
git revert commit-hash              # Create undo commit
git revert HEAD                     # Revert last commit
git revert --continue               # Continue after conflict
```

## 📝 History & Logs

### Viewing History

Log Basics:

```bash
git log                             # Full commit log
git log --oneline                   # Compact format
git log --graph --all --decorate    # Visual graph
git log -n 10                       # Last 10 commits
git log --follow file.txt           # File history
git log --stat                      # Show statistics
git log -p                          # Show diffs
```

Filter Logs:

```bash
git log --author="John"             # By author
git log --since="2 weeks ago"       # Date range
git log --until="2025-01-01"        # Before date
git log --grep="keyword"            # Search message
git log main..feature               # In feature, not in main
git log --all --grep="bug"          # All branches, search
```

### Blame & Diff

Find Changes:

```bash
git blame file.txt                  # Who changed what
git blame -L 10,20 file.txt         # Specific lines
git log -p file.txt                 # Full file history
git log --follow -p file.txt        # Including renames
git diff                            # Unstaged changes
git diff --staged                   # Staged changes
git diff HEAD~2 HEAD                # Between commits
```

## 🏷️ Tags & Releases

### Tag Management

Create Tags:

```bash
git tag v1.0.0                      # Lightweight tag
git tag -a v1.0.0 -m "Release"      # Annotated tag
git tag -s v1.0.0 -m "Release"      # Signed tag (GPG)
git tag v1.0.0 commit-hash          # Tag specific commit
git tag -d v1.0.0                   # Delete local
git push origin v1.0.0              # Push tag
git push origin --all --tags        # Push everything
```

List & Show Tags:

```bash
git tag                             # List all
git tag -l "v1.*"                   # Pattern match
git tag -n                          # With messages
git show v1.0.0                     # Show tag details
git verify-tag v1.0.0               # Verify signature
```

## 🚀 Git Workflows

### Feature Branch Workflow

Complete Flow:

```bash
git checkout -b feature/user-auth
# Make changes
git add .
git commit -m "Add user authentication"
git push origin feature/user-auth
# Create Pull Request on GitHub/GitLab
# After review:
git checkout main
git pull origin main
git merge feature/user-auth
git push origin main
git branch -d feature/user-auth
git push origin -d feature/user-auth
```

### Git Flow Model

Branches:

- `main` — Production ready code
- `develop` — Development branch
- `feature/*` — New features
- `release/*` — Release preparation
- `hotfix/*` — Production fixes

Example:

- `feature/login`
- `feature/payment`
- `release/v1.0.0`
- `hotfix/security-issue`

## 🔐 Security & Best Practices

### Security Setup

SSH Configuration:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
# Add public key to GitHub/GitLab
# Test: ssh -T git@github.com
```

GPG Signing:

```bash
gpg --gen-key                       # Generate key
git config --global user.signingkey KEY_ID
git config --global commit.gpgsign true
git commit -S -m "Signed commit"
git verify-commit commit-hash
git tag -s v1.0.0                   # Signed tag
```

### Commit Best Practices

Guidelines:

- Atomic commits (single unit of work)
- Clear, descriptive messages
- Present tense: "Add feature" not "Added feature"
- Reference issues: "Fix #123"
- Keep commits small and focused
- Test before committing

Never commit:

- Secrets or API keys
- Large binary files
- Passwords or tokens
- Generated files (unless needed)
- IDE configuration

## 🐛 Troubleshooting

### Common Issues

Undo Mistakes:

```bash
# Undo uncommitted changes:
git checkout -- file.txt            # Discard changes
git restore file.txt                # Modern syntax

# Undo committed changes:
git revert commit-hash              # Safe - creates undo
git reset --soft HEAD~1             # Undo - keep changes
git reset --hard HEAD~1             # Undo - lose changes

# Recover deleted branch:
git reflog
git checkout -b branch-name hash    # Recreate from reflog
```

Merge Conflicts:

1. Pull latest from remote
2. Fix conflicts in editor
3. `git add conflicted-files`
4. `git commit -m "Resolve conflicts"`
5. `git push origin branch`

Tools: `git mergetool`

Visual: VS Code, GitHub Desktop, Meld

### Performance & Cleanup

Repository Maintenance:

```bash
git gc --aggressive                  # Optimize repository
git prune                            # Remove unreachable
git fsck --full                      # Check integrity
git reflog expire --expire=now --all # Clean reflog
git repack -d -l                     # Repack objects

# Large repository cloning:
git clone --depth 1 <url>            # Shallow clone
git clone --filter=blob:none <url>   # Partial clone
git sparse-checkout set path/        # Checkout partial
```

## 📋 Quick Reference Table

| Command | Purpose | Example |
|---|---|---|
| `git init` | Create repo | `git init my-project` |
| `git clone` | Copy repo | `git clone https://...` |
| `git add` | Stage | `git add .` |
| `git commit` | Commit | `git commit -m "msg"` |
| `git push` | Upload | `git push origin main` |
| `git pull` | Download | `git pull origin main` |
| `git branch` | Branches | `git branch feature` |
| `git checkout` | Switch | `git checkout main` |
| `git merge` | Combine | `git merge feature` |
| `git rebase` | Reorder | `git rebase main` |
| `git stash` | Save WIP | `git stash` |
| `git log` | History | `git log --oneline` |

## ✅ Best Practices Checklist

### Before Committing

- Review changes with `git diff`
- Run tests before commit
- Use meaningful commit messages
- Make atomic commits
- Check for secrets/passwords

### Branch Management

- Use descriptive branch names
- Keep branches short-lived
- Delete merged branches
- Track upstream branches
- Protect main branch

### Collaboration

- Pull before pushing
- Communicate with team
- Do code reviews
- Use pull requests
- Keep history clean

### Repository Health

- Regular garbage collection
- Monitor repository size
- Document workflows
- Maintain `.gitignore`
- Tag releases

### 💡 Pro Tips

- Use git aliases for frequent commands
- Configure merge strategy
- Set up pre-commit hooks
- Use `git bisect` to find bugs
- Learn advanced rebasing

### ⚠️ Never

- Force push to shared branches
- Commit secrets or keys
- Ignore merge conflicts
- Mix unrelated changes
- Rewrite public history

---
*Source: adapted from the GIT cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
