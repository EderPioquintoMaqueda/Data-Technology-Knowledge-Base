---
tags:
  - linux
  - system-administration
  - beginner
  - ibm-infrastructure
  - obsidian-vault
aliases:
  - Linux Command Line Navigation
---

# Linux Foundations: Command Line Navigation and File Management

Related notes: [[SQL-B04 Creating Tables - DDL, DML and Constraints]], [[pandas_series_dataframe_internals]]

---

## 1. The Linux Philosophy and Filesystem Hierarchy (FHS)

In Linux, **everything is a file** (including hardware devices, directories, and network sockets). The filesystem is structured as a single hierarchical tree rooted at `/` (root), unlike Windows which separates drives (`C:\`, `D:\`).

### Key Directory Paths
* `/` : Root directory (the top of the hierarchy).
* `/home` : User home directories (e.g., `/home/eder`).
* `/etc` : System-wide configuration files.
* `/var` : Variable data files (logs, databases like PostgreSQL data files).
* `/tmp` : Temporary files (cleared on reboot).

---

## 2. Essential Navigation and File Commands

### Terminal Commands Reference

```bash
# 1. Check current working directory
pwd

# 2. List files and directories (including hidden files with -a, and details with -l)
ls -la

# 3. Change directory
cd /var/log

# 4. Move up one directory level
cd ..

# 5. Create a new directory
mkdir -p projects/data_science_vault

# 6. Copy files or directories
cp source.txt backup.txt

# 7. Move or rename files
mv old_name.txt new_name.txt

# 8. Remove files or directories (-r for recursive, -f for force)
rm -rf temp_folder/
```

---

## 3. Reading and Searching Files
```bash
cat file.txt            # print the whole file
less big.log            # scroll (q to quit, / to search)
head -n 20 data.csv     # first 20 lines
tail -n 50 app.log      # last 50 lines
tail -f app.log         # follow a growing log live
wc -l data.csv          # count lines
grep -i "error" app.log              # search text (case-insensitive)
grep -rn "SELECT" src/               # recursive, with line numbers
find . -name "*.csv" -size +100M     # find big CSV files
du -sh data/*                        # size of each item
```

## 4. Pipes and Redirection — combining small tools
```bash
cat orders.csv | cut -d',' -f3 | sort | uniq -c | sort -rn | head    # top values of column 3
grep "ERROR" app.log > errors.txt     # > overwrite
echo "done" >> run.log                 # >> append
python etl.py 2> errors.log            # 2> redirect errors
python etl.py > out.log 2>&1           # both outputs to one file
```

## 5. Windows equivalents
| Linux / macOS / Git Bash / WSL | PowerShell |
|---|---|
| `ls` | `ls` / `Get-ChildItem` |
| `cat` | `cat` / `Get-Content` |
| `grep` | `Select-String` |
| `rm -r` | `Remove-Item -Recurse` |
| `export VAR=1` | `$env:VAR = "1"` |
💡 On Windows, install **WSL2** (Ubuntu) to get a real Linux shell — most data tooling assumes it.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| directory | /dəˈrek.tɚ.i/ | di-**REK**-to-ri | directorio |
| root | /ruːt/ | rut | raíz |
| hidden | /ˈhɪd.ən/ | **JI**-den | oculto |
| pipe | /paɪp/ | paip | tubería (canalización) |
| redirect | /ˌriː.dəˈrekt/ | ri-di-**REKT** | redirigir |
| recursive | /rɪˈkɝː.sɪv/ | ri-**KER**-siv | recursivo |

➡️ [[LIN_02_Permissions_Processes_and_System_Monitoring]] · 🏠 [[00 - Tools Roadmap]]
