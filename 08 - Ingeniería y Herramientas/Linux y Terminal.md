# 🐧 Linux y la Terminal: Administración de Sistemas para IA | Linux & The Command Line: System Administration for AI

**ES:** Para un profesional de datos, la terminal no es solo una herramienta para lanzar scripts; es el **sistema operativo de la infraestructura de IA**. La gran mayoría de los modelos de ML se entrenan en servidores Linux remotos. Dominar la CLI (Command Line Interface) es la diferencia entre depender de una interfaz gráfica lenta y controlar el hardware al máximo nivel de eficiencia.

**EN:** For a data professional, the terminal is not just a tool for launching scripts; it is the **operating system of the AI infrastructure**. Most ML models are trained on remote Linux servers. Mastering the CLI (Command Line Interface) is the difference between depending on a slow graphical interface and controlling the hardware at the maximum level of efficiency.

---

## 1. Advanced Bash Scripting: Automating Data Pipelines | Bash Scripting Avanzado: Automatizando Pipelines de Datos

Un experto no ejecuta comandos uno por uno; crea **scripts robustos** que manejan errores y automatizan flujos complejos.

### A. Estructuras de Control Profesionales
Un script senior no solo corre comandos, valida el entorno:
- **Manejo de Errores:** Uso de `set -e` para que el script se deture inmediatamente si un comando falla, evitando que un error en la extracción de datos corrompa la base de datos.
- **Loops y Condicionales:** Procesar múltiples archivos de datos en una carpeta automáticamente.

### B. El Poder del Pipe y el Redireccionamiento
El concepto de "Tuberías" permite encadenar herramientas pequeñas para lograr resultados masivos:
- `grep "ERROR" log.txt | wc -l` $\rightarrow$ Cuenta cuántos errores hay en un log.
- `cat data.csv | awk -F',' '{print $1}' | sort | uniq -c` $\rightarrow$ Cuenta la frecuencia de valores únicos en la primera columna de un CSV.

---

## 2. Remote Infrastructure & Security | Infraestructura Remota y Seguridad

El entrenamiento de modelos ocurre en servidores con GPUs (AWS, Azure, GCP). Acceder a ellos requiere seguridad y eficiencia.

### A. SSH (Secure Shell) & Key Management
Olvidar las contraseñas y usar **Llaves SSH (Public/Private Keys)**:
- **ssh-keygen:** Genera el par de llaves.
- **ssh-copy-id:** Envía la llave pública al servidor para entrar sin contraseña.
- **Túneles SSH (Port Forwarding):** Permite acceder a un puerto del servidor remoto (ej. el puerto 8888 de un Jupyter Notebook) como si estuviera corriendo en tu propia laptop.

### B. Background Processes & Persistence
Cuando entrenas un modelo que tarda 10 horas, no puedes mantener la terminal abierta.
- **nohup:** `nohup python train.py &` $\rightarrow$ Ejecuta el script en segundo plano y no lo mata si cierras la conexión.
- **Tmux / Screen:** Crea sesiones persistentes en el servidor. Puedes desconectarte, apagar tu laptop, volver al día siguiente y encontrar la terminal exactamente donde la dejaste.

---

## 3. Hardware Monitoring: Diagnosing Bottlenecks | Monitorización de Hardware: Diagnosticando Cuellos de Botella

Entrenar una IA es costoso. Un experto sabe exactamente dónde está el cuello de botella del sistema.

### A. GPU Monitoring (`nvidia-smi`)
La herramienta fundamental para cualquier ingeniero de IA. Permite monitorear:
- **GPU Utilization:** ¿Está la GPU trabajando al 100% o el CPU es el cuello de botella?
- **VRAM Usage:** ¿Se quedó el modelo sin memoria (*Out of Memory - OOM*)?
- **Power & Temperature:** Evitar el *Thermal Throttling* (cuando la GPU baja la velocidad para no quemarse).

### B. System Resources (`htop`, `iotop`, `df`)
- **`htop`:** Visualización en tiempo real de la carga de cada núcleo de la CPU y la RAM.
- **`iotop`:** Detecta si el entrenamiento es lento porque el disco duro no puede leer los datos lo suficientemente rápido.
- **`df -h`:** Verifica el espacio en disco antes de descargar un dataset de 500GB.

---

## 💻 Professional Automation Example: The Training Wrapper

```bash
#!/bin/bash
# script_entrenamiento.sh - Wrapper profesional para entrenamiento de IA

# 1. Variables de entorno
MODEL_NAME="transformer_v1"
LOG_FILE="logs/train_$(date +%Y%m%d).log"
GPU_ID=0

echo "Iniciando entrenamiento de $MODEL_NAME en GPU $GPU_ID..."

# 2. Verificar si la GPU está disponible
if ! nvidia-smi | grep -q "NVIDIA"; then
    echo "ERROR: No se detectó GPU NVIDIA. Abortando."
    exit 1
fi

# 3. Ejecutar entrenamiento en segundo plano con persistencia
# Guardamos la salida estándar y el error en el archivo de log
export CUDA_VISIBLE_DEVICES=$GPU_ID
nohup python train.py --model $MODEL_NAME > $LOG_FILE 2>&1 &

echo "Entrenamiento lanzado en segundo plano. PID: $!"
echo "Puedes monitorear el progreso con: tail -f $LOG_FILE"
```

#linux #terminal #bash #devops #gpu-monitoring #ssh #system-admin
