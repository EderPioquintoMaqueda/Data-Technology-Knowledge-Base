---
tags:
  - git
  - version-control
  - intermediate
  - collaboration
  - obsidian-vault
aliases:
  - Git Merging and Remote Repositories
---

# Git Intermediate: Remote Repositories, Merging, and Conflict Resolution

Related notes: [[GIT_01_Beginner_Basics_Commit_and_Branch]], [[LIN_01_Command_Line_Navigation_and_File_Management]]

---

## 1. Working with Remote Repositories (GitHub / GitLab / IBM Enterprise Git)

Local repositories exist solely on your machine. To collaborate with teams or back up code to enterprise servers, you link your local repository to a remote server.

### Essential Remote Commands
```bash
# 1. Link your local repository to a remote repository URL
git remote add origin [https://github.com/eder-maqueda/data-analysis-vault.git](https://github.com/eder-maqueda/data-analysis-vault.git)

# 2. Verify remote connections
git remote -v

# 3. Push your local commits to the remote main branch
git push -u origin main

# 4. Fetch and download changes from the remote repository without merging yet
git fetch origin

# 5. Download and merge remote changes into your current local branch
git pull origin main
```

---

## 2. Authentication
- **HTTPS + Personal Access Token**, or the GitHub CLI: `gh auth login`.
- **SSH keys:** `ssh-keygen -t ed25519 -C "you@example.com"` → add the public key (`~/.ssh/id_ed25519.pub`) to GitHub → `git remote set-url origin git@github.com:user/repo.git`.

## 3. Merge Strategies
| Situation | Result |
|---|---|
| `main` did not move since you branched | **fast-forward** (no merge commit) |
| both branches have new commits | **merge commit** with two parents |
| `git merge --squash feature` | all feature commits combined into one |

## 4. Resolving Conflicts
A conflict happens when two branches change the **same lines**.
```text
<<<<<<< HEAD
SELECT country, SUM(total) FROM orders GROUP BY country;
=======
SELECT country, SUM(total) AS revenue FROM orders GROUP BY country ORDER BY revenue DESC;
>>>>>>> feature/revenue-sort
```
1. Open the file, keep the correct version (or combine both), delete the markers.
2. `git add file.sql`
3. `git commit` (or `git merge --continue`).
4. To give up: `git merge --abort`.
VS Code shows **Accept Current / Incoming / Both** buttons.

## 5. Pull Requests (GitHub Flow)
```
main ──●──────────────●── (merge PR)
        \            /
         ●──●──●────●   feature/x  → push → open PR → review → CI checks → merge
```
```bash
git switch -c feature/add-cohort-query
git push -u origin feature/add-cohort-query
gh pr create --title "Add cohort retention query" --body "Closes #12"
```
A good PR: small, one purpose, clear description, screenshots/results for data changes, tests passing.

## 6. Keeping your Branch Up to Date
```bash
git fetch origin
git merge origin/main          # or: git rebase origin/main  (see GIT_03)
```

## 🧪 Practice
1. Create a conflict on purpose in two branches and resolve it.
2. Push a branch and open a PR with `gh pr create`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| remote | /rɪˈmoʊt/ | ri-**MOUT** | remoto |
| push / pull | /pʊʃ/ /pʊl/ | push / pul | subir / bajar (cambios) |
| fetch | /fetʃ/ | fech | traer (sin fusionar) |
| conflict | /ˈkɑːn.flɪkt/ | **KAN**-flikt | conflicto |
| pull request | /pʊl rɪˈkwest/ | pul ri-**KUEST** | solicitud de integración |
| review | /rɪˈvjuː/ | ri-**VIU** | revisión |
| fast-forward | /ˌfæstˈfɔːr.wɚd/ | fast-**FOR**-uerd | avance rápido |

⬅️ [[GIT_01_Beginner_Basics_Commit_and_Branch]] · ➡️ [[GIT_03_Pro_Rebase_Stashing_and_Collaborative_Workflows]]
