# 🐧 Linux y la Terminal | Linux & The Command Line

## ¿Qué es? | What is it?
**ES:** Linux es la familia de sistemas operativos que domina el mundo de los servidores, la nube y la ciencia de datos. La Terminal (CLI - Command Line Interface) es la interfaz de texto que permite interactuar con el sistema operativo de forma mucho más rápida y potente que usando el ratón.

**EN:** Linux is the family of operating systems that dominates the world of servers, the cloud, and data science. The Terminal (CLI - Command Line Interface) es the text interface that allows interacting with the operating system much faster and more powerfully than using a mouse.

---

## 🛠️ Comandos Esenciales para Data Scientists | Essential Commands

Para moverte en un servidor de datos, debes dominar estos comandos básicos:

### 1. Navegación y Archivos (File System)
| Comando | Acción | Ejemplo |
| :--- | :--- | :--- |
| `ls` | Listar archivos | `ls -la` (Ver todo, incluyendo ocultos) |
| `cd` | Cambiar directorio | `cd /home/user/projects` |
| `pwd` | Ruta actual | `pwd` (Print Working Directory) |
| `mkdir` | Crear carpeta | `mkdir my_project` |
| `rm` | Borrar | `rm -rf folder` (Borrar carpeta y contenido) |
| `cp` / `mv` | Copiar / Mover | `mv old_name.py new_name.py` |

### 2. Gestión de Sistema y Permisos
- `sudo`: Ejecutar un comando con permisos de administrador (Super User Do).
- `chmod`: Cambiar permisos de un archivo (Ej: `chmod +x script.sh` para hacerlo ejecutable).
- `top` / `htop`: Ver el uso de CPU y RAM en tiempo real (Crucial al entrenar modelos de ML).

### 3. Manipulación de Texto (The Power Tools)
- `grep`: Buscar texto dentro de archivos. (Ej: `grep "Error" log.txt`).
- `cat` / `tail`: Ver contenido de archivos. `tail -f log.txt` permite ver el log en tiempo real mientras el programa corre.
- `nano` / `vim`: Editores de texto directamente en la terminal.

---

## 🚀 Automatización con Bash Scripts | Bash Automation

Un script de Bash es un archivo de texto que contiene una secuencia de comandos que la terminal ejecuta automáticamente.

#### 💻 Ejemplo: Script de Backup de Base de Datos
```bash
#!/bin/bash

# Definir variables
DB_NAME="my_database"
BACKUP_DIR="/backups/db"
DATE=$(date +%Y-%m-%d)

echo "Iniciando backup de la base de datos $DB_NAME..."

# Crear carpeta si no existe
mkdir -p $BACKUP_DIR

# Ejecutar comando de exportación de PostgreSQL
pg_dump $DB_NAME > $BACKUP_DIR/backup_$DATE.sql

echo "Backup completado exitosamente en $BACKUP_DIR"
```

---

## Aplicación Real en Datos | Real-world Data Application

**Caso: Entrenamiento de Modelos en Servidores Remotos**
Un Data Scientist no entrena modelos pesados en su laptop, sino en un servidor con GPUs en la nube:
1. Se conecta vía **SSH** (Secure Shell) al servidor.
2. Navega la estructura de archivos con `cd` y `ls`.
3. Activa un entorno virtual de Python.
4. Ejecuta el entrenamiento en segundo plano usando `nohup python train.py &` para que el proceso no se detenga si se cierra la conexión.
5. Monitorea la temperatura de la GPU y el uso de memoria con `nvidia-smi` y `htop`.

---

## Conceptos Relacionados | Related Concepts
- [[08 - Ingeniería y Herramientas|Docker]] $\longleftrightarrow$ La mayoría de las imágenes de Docker están basadas en distribuciones de Linux (como Ubuntu o Alpine).
- [[08 - Ingeniería y Herramientas|APIs y Cloud]] $\rightarrow$ Las instancias de AWS o Azure son, en su gran mayoría, servidores Linux.

---
#linux #terminal #bash #devops #cli
