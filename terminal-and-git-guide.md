# Terminal and Git Quick Guide

## 1. Shell Basics (navigation, viewing, searching)

```bash
# who/where
whoami            # current user
pwd               # print working directory
ls                # list files
ls -lah           # list incl. hidden, human sizes
```

```bash
# move around
cd /path/to/folder
cd ..             # up one
cd -              # back to previous dir
open .            # open current folder in Finder
```

```bash
# make / copy / move / delete
mkdir -p "New Folder/Deeper"          # -p makes parents
touch notes.txt                        # new empty file
cp file.txt file.bak                   # copy file
cp -a "Src Folder" "Dst Folder"        # copy folder, preserve attrs
mv oldname.txt newname.txt             # move/rename
rm file.txt                            # delete file
rm -rf "Folder To Delete"              # force-remove folder (careful!)
```

```bash
# look inside files
cat file.txt
less file.txt          # q to quit
head -n 20 file.txt
tail -n 50 -f log.txt  # follow a growing log
```

```bash
# find things
find . -name "*.ipynb"
grep -R "search phrase" .             # find text in files
grep -Rni "pattern" .                 # with line numbers, case-insensitive
```

```bash
# disk usage
du -sh .               # size of current dir
du -sh * | sort -h     # sizes of items in dir
```

```bash
# zip / tar
zip -r project.zip project/
tar -czf project.tgz project/
tar -xzf project.tgz
```

```bash
# permissions (rare, but handy)
chmod +x script.sh     # make executable
```

```bash
# processes / ports
ps aux | grep python
lsof -i :8000          # what's using port 8000?
kill -9 <PID>          # force kill a process (last resort)
```

```bash
# clipboard + quick tricks
pbcopy < file.txt
pbpaste > file.txt
open file.pdf          # open with default app
```

## 2. Safe copying & syncing (big folders, Google Drive paths)

Reliable for long paths with spaces; shows progress; resumes.

```bash
# one-way sync (no deletions on dest)
rsync -avh --progress "/Users/jas.../Shared drives/11a. Teachable Platform - Students/" \
  "/Users/jas.../My Drive/0. Data Love Co Shared Drive/Market Research/Rebel/"

# mirror (deletes on dest if removed in src) -> use carefully!
rsync -avh --delete --progress "SRC/" "DST/"
```

## 3. Python/Conda (since you use Anaconda)

```bash
# envs
conda info --envs
conda create -n dlc-py311 python=3.11 -y
conda activate dlc-py311
conda deactivate

# packages
conda install pandas numpy -y
pip install -U jupyterlab matplotlib

# cleanup
conda clean -a -y           # clears caches (frees space)
```

If you need a plain venv inside a repo:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## 4. Homebrew (CLI tools)

```bash
# install (if missing)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew update && brew upgrade
brew install gh git-lfs tree jq
```

## 5. Git – everyday workflow

### One-time setup (identity + line endings)

```bash
git config --global user.name "Jasmine Motupalli"
git config --global user.email "jasmine@dataloveco.com"
git config --global pull.rebase true      # prefer rebase over merge on git pull
git config --global init.defaultBranch main
```

### Start / connect a repo

```bash
# new repo
git init
git add .
git commit -m "Initial commit"

# link to GitHub
git remote add origin https://github.com/data-love-co/REPO.git
git branch -M main
git push -u origin main
```

### Daily loop

```bash
git status                       # what changed?
git add -A                       # stage all
git commit -m "Clear, scoped message"
git pull --rebase origin main    # update local main w/out merge commits
git push                         # upload your commits
```

**Note:** Always pull before pushing to avoid conflicts!

### Branching & PRs

```bash
git checkout -b feat/notebook-cleanup     # new branch
# ...edit...
git add -A && git commit -m "Cleanup nb outputs"
git push -u origin feat/notebook-cleanup  # push branch (make PR on GitHub)
```

### Keep your feature branch current

```bash
git fetch origin
git rebase origin/main                    # replay your commits on latest main

# resolve conflicts -> edit files -> then:
git add <fixed-file>
git rebase --continue

# if rebase gets messy:
git rebase --abort
```

### Merge vs. Rebase (TL;DR)

- **Pull with rebase** keeps history linear (`git pull --rebase`).
- Use **merge** when preserving exact integration history is important:
  ```bash
  git merge origin/main
  ```

### Stash (quickly shelve changes)

```bash
git stash push -m "WIP: analysis"
git pull --rebase
git stash pop
```

### Undo / fix mistakes

```bash
git restore --staged path/file.py         # unstage
git checkout -- path/file.py              # discard local file changes
git commit --amend                        # edit last commit message/files
git reset --soft HEAD~1                   # undo last commit, keep changes staged
git reset --hard HEAD~1                   # drop last commit & changes (careful!)
```

### Tags & releases

```bash
git tag -a v0.1.0 -m "First cut"
git push origin v0.1.0
```

### Diff & blame

```bash
git diff                                  # unstaged vs working tree
git diff --staged                         # staged vs HEAD
git blame path/file.py                    # who changed each line
```

## 6. Fixing the common "rejected – fetch first" push error

When you see:

```
Updates were rejected because the remote contains work that you do not have locally...
```

**What happened:** Someone (maybe you from another machine) pushed commits to GitHub that you don't have locally yet.

**The fix:**

```bash
# Step 1: Get the latest changes from GitHub
git fetch origin

# Step 2: Integrate them into your branch
git pull --rebase origin main        # or your current branch name

# Step 3: If conflicts appear, resolve them in your editor, then:
git add <fixed-file>
git rebase --continue

# Step 4: Now push your changes
git push
```

**Pro tip:** Make this your habit before every push:

```bash
git pull --rebase origin main
git push
```

This avoids most push rejections!

## 7. Switching GitHub credentials / PATs on macOS

### HTTPS with PAT (recommended for simplicity)

**Step 1:** Ensure Keychain helper is active:

```bash
git config --global credential.helper osxkeychain
```

**Step 2:** Clear the cached credential:

```bash
# Method A: via helper
printf "protocol=https\nhost=github.com\n\n" | git credential-osxkeychain erase

# Method B: remove via Keychain Access app
#   - Open "Keychain Access" → search "github.com" → delete the "internet password"
```

**Step 3:** Next `git push` will prompt for:
- **Username:** your GitHub handle
- **Password:** paste your **Personal Access Token** (not your actual password!)

### Force a different token just once (quick hack)

```bash
git remote set-url origin https://<TOKEN>@github.com/data-love-co/REPO.git

# Later, set it back to the clean URL:
git remote set-url origin https://github.com/data-love-co/REPO.git
```

### GitHub CLI (nice UX)

```bash
gh auth logout
gh auth login           # choose GitHub.com → HTTPS → paste PAT when asked
```

## 8. SSH alternative (if you prefer keys)

```bash
# create key (ed25519)
ssh-keygen -t ed25519 -C "jasmine@dataloveco.com"
# press Enter to accept default path (~/.ssh/id_ed25519), add a passphrase

# start agent & add key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# copy public key and add to GitHub → Settings → SSH and GPG keys
pbcopy < ~/.ssh/id_ed25519.pub

# switch remote to SSH
git remote set-url origin git@github.com:data-love-co/REPO.git
```

## 9. Large files & notebooks

### Git LFS for big binaries

```bash
# Git LFS for big binaries (models, media)
git lfs install
git lfs track "*.parquet"
git add .gitattributes
git add path/to/file.parquet
git commit -m "Track parquet with LFS"
git push
```

### Jupyter outputs

Keep repos lean by clearing outputs before commit (or use `nbstripout`):

```bash
pip install nbstripout
nbstripout --install
# now outputs are removed on commit automatically
```

## 10. One-liners you'll likely reuse

```bash
# repo tree (top 2 levels)
find . -maxdepth 2 -print | sed -e 's/[^-][^\/]*\//  /g' -e 's/\/$//;s/^[ ]*//'

# list big files (>100MB)
find . -type f -size +100M -print0 | xargs -0 ls -lh

# clean all __pycache__ and .ipynb_checkpoints
find . -type d -name "__pycache__" -prune -exec rm -rf {} +
find . -type d -name ".ipynb_checkpoints" -prune -exec rm -rf {} +

# quickly create a .gitignore
curl -L https://www.toptal.com/developers/gitignore/api/python,macos,visualstudiocode,jetbrains,anaconda > .gitignore
```

## 11. Minimal .gitignore for your workflow

```gitignore
# macOS
.DS_Store

# Python
__pycache__/
*.pyc
.venv/
.env

# Jupyter
.ipynb_checkpoints/

# Anaconda
anaconda3/
*.conda
*.ipynb_convert

# VS Code / JetBrains
.vscode/
.idea/

# Data / artifacts
data/
outputs/
*.parquet
*.csv.gz
*.zip
*.tgz
```

**Tip:** Keep **sample** data small (e.g., `data/sample/`) and commit it; keep real data ignored.

## 12. Rebase cheat (step-by-step)

```bash
# update main
git checkout main
git fetch origin
git pull --rebase origin main

# rebase your branch
git checkout feat/my-branch
git rebase main

# fix conflicts: edit files, then
git add <fixed-file>
git rebase --continue

# if stuck
git rebase --abort
```

## 13. Quick "first push" checklist (new machine/repo)

1. **Install Git:** `git --version` (installed?)
2. **Configure identity:**
   ```bash
   git config --global user.name "Jasmine Motupalli"
   git config --global user.email "jasmine@dataloveco.com"
   ```
3. **Set credentials:** PAT via Keychain or `gh auth login` or SSH
4. **Clone or init:**
   ```bash
   git clone ...
   # OR
   git init
   git remote add origin https://github.com/data-love-co/REPO.git
   ```
5. **Pull first (if repo has commits):**
   ```bash
   git pull --rebase origin main
   ```
6. **Work → commit → push:**
   ```bash
   git add -A
   git commit -m "Your message"
   git pull --rebase origin main  # always pull before push!
   git push
   ```

## Common Push Issues - Quick Troubleshooting

### Issue: "Updates were rejected"
**Solution:** Someone pushed before you. Pull first, then push:
```bash
git pull --rebase origin main
git push
```

### Issue: "Authentication failed"
**Solution:** Your PAT expired or credentials are wrong:
```bash
# Clear keychain and re-authenticate
printf "protocol=https\nhost=github.com\n\n" | git credential-osxkeychain erase
git push  # will prompt for credentials
```

### Issue: "Failed to push some refs"
**Solution:** You're behind the remote. Pull and merge/rebase:
```bash
git fetch origin
git pull --rebase origin main
# resolve any conflicts
git push
```

### Issue: "Repository not found" or 403 error
**Solution:** Check your remote URL and permissions:
```bash
git remote -v  # verify URL is correct
# Update if needed:
git remote set-url origin https://github.com/data-love-co/REPO.git
```