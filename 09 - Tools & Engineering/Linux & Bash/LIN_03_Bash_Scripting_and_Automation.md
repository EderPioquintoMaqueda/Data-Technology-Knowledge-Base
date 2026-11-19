---
tags:
  - linux
  - system-administration
  - automation
  - bash-scripting
  - obsidian-vault
aliases:
  - Bash Scripting and Automation
---

# Linux Pro: Bash Scripting and Task Automation

Related notes: [[LIN_01_Command_Line_Navigation_and_File_Management]], [[LIN_02_Permissions_Processes_and_System_Monitoring]], [[SQL-B04 Creating Tables - DDL, DML and Constraints]]

---

## 1. Introduction to Bash Scripting

Bash (Bourne Again SHell) scripting allows you to automate repetitive server tasks, database backups, and Python pipeline deployments. A Bash script is simply a plain text file containing a sequence of shell commands.

### Creating and Executing a Script
```bash
# 1. Create a new script file
nano backup_pipeline.sh
```

---

## 2. A Robust Script Template
```bash
#!/usr/bin/env bash
# backup_pipeline.sh — dump PostgreSQL and run the ETL
set -euo pipefail            # stop on errors, undefined vars and pipe failures
IFS=$'\n\t'

DB_NAME="${DB_NAME:-datashop}"                 # default if not set
BACKUP_DIR="/var/backups/${DB_NAME}"
STAMP="$(date +%Y-%m-%d_%H%M)"
LOG="/var/log/pipeline_${STAMP}.log"

log() { echo "[$(date +'%F %T')] $*" | tee -a "$LOG"; }

mkdir -p "$BACKUP_DIR"
log "Starting backup of ${DB_NAME}"
pg_dump -Fc "$DB_NAME" > "${BACKUP_DIR}/${DB_NAME}_${STAMP}.dump"

log "Running ETL"
if python3 /opt/etl/main.py --date "$(date +%F)"; then
    log "ETL finished OK"
else
    log "ETL FAILED"; exit 1
fi

# keep only the last 7 backups
ls -1t "${BACKUP_DIR}"/*.dump | tail -n +8 | xargs -r rm --
log "Done"
```
```bash
chmod +x backup_pipeline.sh
./backup_pipeline.sh
```

## 3. Bash Building Blocks
```bash
name="Ana"; echo "Hello, $name"          # variables (no spaces around =)
files=$(ls *.csv)                         # command substitution
if [[ -f data.csv ]]; then echo "exists"; fi
if [[ $count -gt 10 ]]; then ...; fi      # -eq -ne -lt -le -gt -ge
for f in data/*.csv; do
    echo "Loading $f"
    psql -d datashop -c "\copy staging FROM '$f' CSV HEADER"
done
while read -r line; do echo "$line"; done < list.txt
case "$1" in
  start) echo "starting";;
  stop)  echo "stopping";;
  *)     echo "usage: $0 {start|stop}"; exit 1;;
esac
```
Arguments: `$0` script name · `$1..$9` args · `$#` count · `$@` all · `$?` last exit code (0 = success).

## 4. Scheduling with cron
```bash
crontab -e
# ┌ minute ┌ hour ┌ day of month ┌ month ┌ day of week
#   30      2      *              *       1-5     → 02:30, Monday–Friday
30 2 * * 1-5 /opt/etl/backup_pipeline.sh >> /var/log/cron_etl.log 2>&1
```
Use [crontab.guru](https://crontab.guru) to check expressions. For dependencies between tasks, retries and monitoring, move to an orchestrator → [[Apache Airflow Fundamentals]].

## 5. Text-processing power tools
```bash
awk -F',' '{sum += $3} END {print sum}' sales.csv      # sum column 3
sed -i 's/;/,/g' export.csv                            # replace ; with ,
cut -d',' -f1,3 data.csv                               # columns 1 and 3
sort -t',' -k3 -n data.csv                             # numeric sort by col 3
jq '.results[] | {id, name}' response.json             # JSON in the terminal
csvlook / csvstat (csvkit)                             # quick CSV inspection
```

## 🧪 Practice
1. Write a script that downloads a CSV with `curl`, counts rows and logs the result.
2. Schedule it daily with cron.
3. Make it fail on purpose and check the exit code with `echo $?`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| script | /skrɪpt/ | skript | guion / script |
| shebang | /ʃəˈbæŋ/ | sho-**BANG** | línea #! inicial |
| exit code | /ˈek.sɪt koʊd/ | **EK**-sit koud | código de salida |
| schedule | /ˈskedʒ.uːl/ | **SKE**-yul | programar / horario |
| substitution | /ˌsʌb.stɪˈtuː.ʃən/ | sab-sti-**TU**-shon | sustitución |
| backup | /ˈbæk.ʌp/ | **BAK**-ap | respaldo |

⬅️ [[LIN_02_Permissions_Processes_and_System_Monitoring]] · Deep dive (ES): [[Linux y Terminal]] · 🏠 [[00 - Tools Roadmap]]
