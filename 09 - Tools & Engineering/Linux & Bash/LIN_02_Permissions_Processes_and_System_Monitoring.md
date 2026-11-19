---
tags:
  - linux
  - system-administration
  - permissions
  - processes
  - obsidian-vault
aliases:
  - Linux Permissions and Processes
---

# Linux Intermediate: Permissions, Processes, and System Monitoring

Related notes: [[LIN_01_Command_Line_Navigation_and_File_Management]], [[SQL-B04 Creating Tables - DDL, DML and Constraints]]

---

## 1. Linux File Permissions (Access Control)

In enterprise environments like IBM servers, security is paramount. Every file and directory in Linux has strict access controls governing three entities:
1. **User (u):** The owner of the file.
2. **Group (g):** A collection of users who share access.
3. **Others (o):** Everyone else on the system.

### Permission Types
* **Read (`r`):** Permission to view file contents or list directory contents (numeric value: `4`).
* **Write (`w`):** Permission to modify or delete a file/directory (numeric value: `2`).
* **Execute (`x`):** Permission to run a script/program or enter a directory (numeric value: `1`).

### Checking and Modifying Permissions
```bash
# View detailed permissions and ownership of files in current directory
ls -l
# Example output: -rwxr-xr-- 1 eder sipdus 4096 Nov 11 16:00 script.sh
# Explanation: Owner has read/write/execute (7), Group has read/execute (5), Others have read-only (4) -> Mode 754

# Changing file permissions using chmod (numeric mode)
chmod 755 pipeline.py

# Changing file ownership using chown (user:group)
sudo chown eder:sipdus database_backup.sql
```

---

## 2. Processes
```bash
ps aux | grep python       # running processes
top                        # live view (htop is nicer)
kill 12345                 # ask a process to stop (SIGTERM)
kill -9 12345              # force stop (SIGKILL) — last resort
nohup python etl.py &      # run in background, survives logout
jobs; fg; bg               # manage background jobs
```
For long jobs over SSH use **tmux**: `tmux new -s etl` → detach `Ctrl+b d` → `tmux attach -t etl`.

## 3. System Monitoring
```bash
df -h                 # disk space per filesystem
free -h               # memory
uptime                # load average
nproc                 # CPU cores
ss -tulpn             # listening ports (is PostgreSQL on 5432?)
journalctl -u postgresql --since "1 hour ago"   # service logs
systemctl status postgresql
nvidia-smi            # GPU usage
```

## 4. Users and sudo
```bash
whoami; id
sudo apt update && sudo apt install -y postgresql-client
sudo adduser analyst
sudo usermod -aG docker $USER      # use docker without sudo (log out/in)
```

## 5. Environment Variables
```bash
export DB_URL="postgresql://user:pass@localhost/shop"
echo $DB_URL
env | grep DB_
# Persist for your user: add the export line to ~/.bashrc
```

## 6. Remote Work with SSH
```bash
ssh analyst@10.0.0.5
scp report.csv analyst@10.0.0.5:/data/
rsync -avz data/ analyst@10.0.0.5:/data/      # efficient sync
ssh -L 5433:localhost:5432 analyst@db-server  # tunnel: local 5433 → remote PostgreSQL
```

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| permission | /pɚˈmɪʃ.ən/ | per-**MI**-shon | permiso |
| owner | /ˈoʊ.nɚ/ | **OU**-ner | propietario |
| process | /ˈprɑː.ses/ | **PRA**-ses | proceso |
| background | /ˈbæk.ɡraʊnd/ | **BAK**-graund | segundo plano |
| disk space | /dɪsk speɪs/ | disk speis | espacio en disco |
| port | /pɔːrt/ | port | puerto |
| tunnel | /ˈtʌn.əl/ | **TA**-nel | túnel |

⬅️ [[LIN_01_Command_Line_Navigation_and_File_Management]] · ➡️ [[LIN_03_Bash_Scripting_and_Automation]]
