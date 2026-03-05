# Git/GitHub Practical Tutorial

> **Target audience:** Programming beginners using Git for the first time
>
> **Goal:** Learn the complete workflow of version control with Git on your local machine, pushing to and pulling from GitHub (remote), separating work with branches, and tracking only the files you need — all in "command + why + how" format.
>
> Examples are for macOS/Linux, but work almost identically on Windows with Git Bash/PowerShell.

---

## 0. Core Concepts in 10 Seconds

- **Working Directory:** The folder where your actual files live
- **Staging Area:** A holding zone for changes you want in the next commit (also called the Index)
- **Commit:** A snapshot (version). The reference point for reverting and comparing
- **Branch:** A line of work. Separate by feature or experiment
- **Remote:** A server repository like GitHub (usually called `origin`)
- **Push:** Upload from local → GitHub
- **Pull:** Download from GitHub → local and apply (= fetch + merge/rebase)
- **Clone:** Download an entire remote repository to create a local copy

---

## 1. Installation & Initial Setup (One Time Only)

### 1.1 Check Git Installation

```bash
git --version
```

### 1.2 Set User Info (Appears in Commit History)

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### 1.3 Default Editor (Optional)

```bash
git config --global core.editor "vim"   # or "emacs", "code --wait"
```

### 1.4 Verify Settings

```bash
git config --global --list
```

---

## 2. Create a Directory and Start Git (Local Repository)

### 2.1 Create a New Project Folder

```bash
mkdir myproject
cd myproject
```

### 2.2 Initialize a Git Repository (Local)

```bash
git init
```

**Why?**
This makes the folder a "version-controlled project." A hidden `.git/` directory (the history database) is created inside.

### 2.3 Check Current Status (The Command You'll Use Most)

```bash
git status
```

### 2.4 Create .gitignore First (Recommended)

Setting up `.gitignore` at the very beginning saves you the trouble of un-tracking files later.

```bash
cat > .gitignore << 'EOF'
# macOS
.DS_Store

# Python
__pycache__/
*.pyc

# VSCode
.vscode/

# build artifacts
dist/
build/
EOF
```

```bash
git add .gitignore
git commit -m "Add .gitignore"
```

> **Tip:** If a file is already tracked and you add it to `.gitignore` later, it will keep being tracked. To stop tracking it:
> ```bash
> git rm -r --cached dist
> git commit -m "Stop tracking dist"
> ```

---

## 3. Save (Track) Files and Create Versions (add/commit)

### 3.1 Create a File

```bash
echo "# My Project" > README.md
```

### 3.2 Check What Changed

```bash
git status
git diff
```

### 3.3 Stage Changes (add)

```bash
git add README.md
```

**Why does the staging area exist?**
Even if multiple files have changed, you can pick and choose which ones go into this particular commit.

### 3.4 Commit

```bash
git commit -m "Add README"
```

### 3.5 View Commit Log

```bash
git log --oneline --decorate --graph
git log --oneline -5    # Show only the last 5 commits
```

---

## 4. Authenticate with GitHub

> You need to be authenticated before you can push to GitHub. Set this up before connecting a remote.

### 4.1 HTTPS

Instead of a password, GitHub now uses **Personal Access Tokens (PAT)**.
Generate one at GitHub Settings → Developer settings → Personal access tokens, then enter it instead of your password when pushing. Once authenticated, your OS keychain may save it automatically.

### 4.2 SSH (Preferred by Many Developers)

Generate a key:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

View the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Add that key to GitHub Settings → SSH keys, then set the remote URL to SSH:

```bash
git remote set-url origin git@github.com:USER/REPO.git
```

---

## 5. Connect to a Remote GitHub Repository (Preparing to Push)

> There are two scenarios:
> - **Option A:** You created a project locally → push it to GitHub
> - **Option B:** A repository already exists on GitHub → download it locally (clone)

---

## 6. Option A: Push a Local Project to GitHub (First Time)

### 6.1 Create an Empty Repository on GitHub

Go to GitHub and create a New repository.

**Note:** If you already have a README locally, turn off the "Add a README" option on GitHub to avoid conflicts.

### 6.2 Register the Remote (origin)

```bash
git remote add origin https://github.com/USER/REPO.git
```

Verify:

```bash
git remote -v
```

### 6.3 Set the Default Branch Name (main recommended)

```bash
git branch -M main
```

### 6.4 First Push + Set Upstream

```bash
git push -u origin main
```

**Why is `-u` important?**
After this, you can just type `git push` and it will automatically target `origin main`.

---

## 7. Option B: Download a GitHub Repository to Your Computer (clone)

### 7.1 Clone (First Time)

```bash
git clone https://github.com/USER/REPO.git
cd REPO
```

**What clone does for you:**
- Downloads the remote contents and creates a local repository
- Automatically registers `origin` as the remote
- Checks out the default branch (main/master)

---

## 8. Create and Switch Branches (The Most Important Way to Separate Work)

### 8.1 List Branches

```bash
git branch          # Local branches only
git branch -a       # Including remote branches
```

### 8.2 Create a New Branch and Switch to It (Recommended One-liner)

```bash
git switch -c feature/login
```

Legacy compatible command:

```bash
git checkout -b feature/login
```

### 8.3 Switch Between Branches

```bash
git switch main
git switch feature/login
```

### 8.4 Delete a Branch (Local)

```bash
git branch -d feature/login     # Only if already merged
git branch -D feature/login     # Force delete
```

### 8.5 Push a Branch to Remote (First Time)

```bash
git push -u origin feature/login
```

---

## 9. Upload/Download Files: push / pull / fetch

### 9.1 Push Your Changes to GitHub

```bash
git add .
git commit -m "Implement login"
git push
```

### 9.2 Pull Changes from GitHub to Your Computer

```bash
git pull
```

`git pull` = `git fetch` (download) + merge or rebase (apply to your work)

### 9.3 Fetch Only (Safer — Download Without Applying)

```bash
git fetch origin
```

Check the differences after fetching:

```bash
git log --oneline --decorate --graph --all
git diff main..origin/main
```

Then apply (merge example):

```bash
git merge origin/main
```

---

## 10. Important Fact About Tracking Directories (Folders)

### 10.1 Git Does Not Track Empty Folders

Git tracks **files**, not folders themselves.
If you want to keep an empty folder in the repository, the common convention is:

```bash
mkdir data
touch data/.gitkeep
git add data/.gitkeep
git commit -m "Keep data directory"
```

---

## 11. Merge a Branch into main

### 11.1 After Finishing Work on a Feature Branch

```bash
git switch main
git pull
git merge feature/login
git push
```

### 11.2 If a Conflict Occurs

When there's a conflict, the affected file will contain markers like this:

```
<<<<<<< HEAD
Your code (content from the current branch)
=======
Their code (content from the branch being merged)
>>>>>>> feature/login
```

Edit the file to keep only the code you want, then:

```bash
git add resolved-file
git commit
git push
```

---

## 12. Undo/Revert Commands You'll Actually Need

### 12.1 Unstage a File (Undo add)

```bash
git restore --staged README.md
```

### 12.2 Discard Changes in Working Directory (Undo edits)

```bash
git restore README.md
```

### 12.3 Fix the Last Commit Message (Keep the content)

```bash
git commit --amend -m "Better message"
```

### 12.4 Restore a File to a Specific Commit

```bash
git restore --source <commit_hash> -- path/to/file
```

### 12.5 Create a "Reverting Commit" (Safe)

```bash
git revert <commit_hash>
```

When you need to undo a commit already pushed to a public branch (like main), **revert is the safest option** — it doesn't rewrite history.

### 12.6 Temporarily Save Work (stash)

When you need to switch branches but your current changes aren't ready to commit:

```bash
git stash              # Temporarily save current changes
git switch main        # Switch to another branch
# ... handle urgent work ...
git switch feature/x   # Come back to original branch
git stash pop          # Restore the saved changes
```

View the stash list:

```bash
git stash list
```

---

## 13. Manage Remote Branches (Useful for Collaboration)

List remote branches:

```bash
git fetch --all
git branch -r
```

Clean up branches deleted from the remote:

```bash
git fetch --prune
```

---

## 14. Recommended Workflows

### 14.1 Solo Work (Simplest)

1. Create a new branch from main

```bash
git switch main
git pull
git switch -c feature/x
```

2. Work → Commit

```bash
git add .
git commit -m "Work on x"
```

3. Push to remote

```bash
git push -u origin feature/x
```

4. Merge into main

```bash
git switch main
git pull
git merge feature/x
git push
```

### 14.2 Team Work (Recommended Flow)

Feature branch → GitHub Pull Request (PR) → Code review → Merge into main

The commands are the same as above; the "merge decision" happens through the GitHub UI.

---

## 15. Checklist

**Create a directory**
- Local folder: `mkdir`, `cd`
- Git doesn't track empty folders → may need a file like `.gitkeep`

**Create a branch**
- Create + switch: `git switch -c branch-name`
- Push to remote: `git push -u origin branch-name`

**Save / Upload files**
- Check changes: `git status`, `git diff`
- Stage: `git add ...`
- Create version: `git commit -m "..."`
- Upload: `git push`

**Download files**
- First full download: `git clone ...`
- Get latest: `git pull`
- Fetch only: `git fetch`

---

## 16. Minimal Cheat Sheet (The Commands You'll Actually Use)

```bash
git status
git diff
git add .
git commit -m "message"
git log --oneline --graph --decorate
git push
git pull

git switch -c feature/x
git switch main
git merge feature/x

git stash
git stash pop
```
