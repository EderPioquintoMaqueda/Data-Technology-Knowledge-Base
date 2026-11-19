---
tags:
  - git
  - version-control
  - beginner
  - ibm-engineering
  - obsidian-vault
aliases:
  - Git Beginner Basics
---

# Git Foundations: Commits, Branching, and Local Workflow

Related notes: [[LIN_01_Command_Line_Navigation_and_File_Management]], [[SQL-B04 Creating Tables - DDL, DML and Constraints]]

---

## 1. What is Git?
Git is a distributed version control system that tracks history, changes, and collaboration across codebases. In enterprise environments like IBM, Git (hosted on GitHub, GitLab, or Bitbucket) is the absolute standard for tracking data pipelines, machine learning models, and software applications.

---

## 2. Core Git Workflow and States

Git tracks files across three primary states:
1. **Working Directory:** Your local files where you write and edit code.
2. **Staging Area (`Index`):** A holding space where you prep snapshots of changes using `git add`.
3. **Repository (`HEAD`):** Permanent history snapshots stored in your local `.git` database via `git commit`.

### Essential Initialization and Commit Commands
```bash
# 1. Initialize a new Git repository in your project folder
git init

# 2. Check the status of your working directory and staging area
git status

# 3. Stage specific files or all changes
git add pipeline.py README.md
# or stage everything:
git add .

# 4. Commit staged changes with a descriptive message
git commit -m "feat: implement initial Pandas data cleaning pipeline"

# 5. View commit history log
git log --oneline
```

---

## 3. First-time Setup
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true      # Windows: convert line endings safely
git config --list
```

## 4. Seeing What Changed
```bash
git diff                 # working directory vs staging area
git diff --staged        # staging area vs last commit
git log --oneline --graph --all
git show <commit_hash>   # details of one commit
git restore file.py              # discard local changes in a file
git restore --staged file.py     # unstage (keep the changes)
```

## 5. Branches
A **branch** is a movable pointer to a commit. Work on features in isolation and keep `main` stable.
```bash
git branch                       # list branches
git switch -c feature/sales-report   # create + switch (older: git checkout -b)
# ...edit, add, commit...
git switch main
git merge feature/sales-report
git branch -d feature/sales-report   # delete merged branch
```

## 6. .gitignore for Data Projects
```gitignore
.venv/
__pycache__/
.ipynb_checkpoints/
.env
data/raw/
*.parquet
*.csv
.DS_Store
```
⚠️ Never commit secrets or large datasets. Version data with **DVC** or store it in object storage.

## 7. Good Commit Messages (Conventional Commits)
```
feat: add monthly revenue aggregation
fix: handle null emails in customer cleaning
docs: explain DataShop schema
refactor: split extract and transform modules
test: add cases for discount calculation
```
Rules: imperative mood ("add", not "added"), ≤ 50 characters in the title, one logical change per commit.

## 🧪 Practice
1. Create a repo for your SQL exercises, commit the practice database script.
2. Create a branch, change a query, merge it back.
3. Accidentally edit a file and recover it with `git restore`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| repository | /rɪˈpɑː.zə.tɔːr.i/ | ri-**PA**-zi-to-ri | repositorio |
| commit | /kəˈmɪt/ | ko-**MIT** | confirmación / confirmar |
| stage | /steɪdʒ/ | steich | preparar (para commit) |
| branch | /bræntʃ/ | branch | rama |
| merge | /mɝːdʒ/ | merch | fusionar |
| track | /træk/ | trak | rastrear / seguir |
| history | /ˈhɪs.tɚ.i/ | **JIS**-to-ri | historial |
| undo | /ʌnˈduː/ | an-**DU** | deshacer |

⬅️ [[00 - Tools Roadmap]] · ➡️ [[GIT_02_Intermediate_Merging_Conflicts_and_Remote_Repositories]]
