---
tags:
  - git
  - version-control
  - pro
  - workflow
  - obsidian-vault
aliases:
  - Git Pro Rebase and Stashing
---

# Git Pro: Stashing, Rebasing, and Collaborative Workflows

Related notes: [[GIT_02_Intermediate_Merging_Conflicts_and_Remote_Repositories]], [[LIN_01_Command_Line_Navigation_and_File_Management]]

---

## 1. Temporarily Saving Work with `git stash`

Often, while working on a feature branch, you need to urgently switch to `main` to fix a production bug, but your current changes are incomplete and not ready to commit. `git stash` saves your uncommitted changes in a temporary stack.

### Essential Stash Commands
```bash
# 1. Save your current dirty working directory and staging area to the stash stack
git stash save "WIP: refactoring data ingestion pipeline"

# 2. List all saved stashes
git stash list

# 3. Apply the most recent stash and remove it from the stack
git stash pop

# 4. Apply a specific stash without removing it from the stack
git stash apply stash@{1}

# 5. Clear all stored stashes
git stash clear
```

---

## 2. Rebase — a linear history
`git rebase main` replays your branch's commits **on top of** the latest `main`.
```bash
git switch feature/x
git fetch origin
git rebase origin/main
# fix conflicts → git add . → git rebase --continue   (or --abort)
git push --force-with-lease      # needed after rewriting history
```
Interactive rebase to clean up before a PR:
```bash
git rebase -i HEAD~4     # pick / squash / reword / drop commits
```
🚨 **Golden rule:** never rebase commits that others already pulled (shared branches like `main`).

## 3. Undoing Things Safely
| Goal | Command |
|---|---|
| Fix the last commit message / add a forgotten file | `git commit --amend` |
| Undo a pushed commit (safe, new commit) | `git revert <hash>` |
| Move branch back, keep changes staged | `git reset --soft HEAD~1` |
| Move branch back, discard changes ⚠️ | `git reset --hard HEAD~1` |
| Recover "lost" commits | `git reflog` → `git switch -c rescue <hash>` |
| Apply one commit from another branch | `git cherry-pick <hash>` |
| Find which commit introduced a bug | `git bisect start/bad/good` |

## 4. Workflows
| Workflow | Branches | Best for |
|---|---|---|
| **GitHub Flow** | `main` + short-lived feature branches | most data/analytics teams |
| **Trunk-based** | everyone merges to `main` daily, feature flags | CI/CD-heavy teams |
| **GitFlow** | `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` | versioned releases |

## 5. Tags and Releases
```bash
git tag -a v1.2.0 -m "Monthly report pipeline v1.2"
git push origin v1.2.0
```

## 6. Git for Data Work
- **Notebooks:** clear outputs before committing (`nbstripout`) or use Jupytext.
- **Large files / datasets:** DVC (`dvc add data/raw.parquet`) or Git LFS.
- **CI:** GitHub Actions running `ruff`, `pytest`, and `sqlfluff lint` on every PR.
```yaml
# .github/workflows/ci.yml
name: ci
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync
      - run: uv run ruff check . && uv run pytest -q
```

## 🧪 Practice
1. Squash 4 messy commits into 1 with interactive rebase.
2. Delete a branch with unmerged work, then recover it with `reflog`.
3. Add a GitHub Actions workflow that runs your tests.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| rebase | /riːˈbeɪs/ | ri-**BÉIS** | rebasar (reaplicar commits) |
| stash | /stæʃ/ | stash | guardar temporalmente |
| squash | /skwɑːʃ/ | skuash | aplastar (combinar commits) |
| revert | /rɪˈvɝːt/ | ri-**VERT** | revertir |
| amend | /əˈmend/ | a-**MEND** | enmendar |
| rewrite history | /riːˈraɪt ˈhɪs.tɚ.i/ | ri-**RÁIT** **JIS**-to-ri | reescribir el historial |
| release | /rɪˈliːs/ | ri-**LIS** | versión publicada |
| hotfix | /ˈhɑːt.fɪks/ | **JAT**-fiks | corrección urgente |

⬅️ [[GIT_02_Intermediate_Merging_Conflicts_and_Remote_Repositories]] · Deep dive (ES): [[Git y GitHub]] · 🏠 [[00 - Tools Roadmap]]
