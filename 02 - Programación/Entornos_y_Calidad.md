# 🛠️ Entornos y Calidad: El Ecosistema del Desarrollador de Datos

Escribir código que funcione es fácil; escribir código que sea mantenible por un equipo y ejecutable en cualquier servidor es lo que define a un profesional.

## 1. Gestión de Entornos Virtuales: Adiós al "En mi PC funciona"

Instalar todo en la base de Python es un error crítico. Los entornos virtuales aíslan las dependencias de cada proyecto.

### Herramientas de Gestión:
- **Venv / Conda**: Los estándares básicos. Conda es superior para ciencia de datos porque gestiona dependencias no-Python (como librerías de C++ necesarias para XGBoost).
- **Poetry / Pipenv**: Herramientas modernas que gestionan el archivo `pyproject.toml` y el `lock file`, asegurando que cada miembro del equipo tenga exactamente las mismas versiones de las librerías.

**Comando Maestro (Poetry)**: `poetry add pandas` $\to$ Instala la librería y actualiza el lock file automáticamente.

---

## 2. Calidad de Código y Estilo (Linting & Formatting)

El código de datos suele ser sucio. Un profesional usa herramientas automáticas para mantener la legibilidad.

### El Stack de Calidad:
1. **Ruff**: El linter más rápido del mundo (escrito en Rust). Detecta errores lógicos, variables no usadas y código redundante.
2. **Black**: El formateador "sin compromisos". Reescribe tu código para que siga el estándar PEP 8 exactamente.
3. **Mypy**: Para el tipado estático. Verifica que no estés pasando un `string` a una función que espera un `float`.

---

## 3. Versionamiento de Datos (DVC - Data Version Control)

Git es excelente para el código, pero horrible para los datos (no puede manejar archivos de 2GB). **DVC** es el "Git para Datos".

### Cómo funciona DVC:
1. El dataset se guarda en la nube (S3, Azure Blob, Google Drive).
2. DVC crea un archivo pequeño `.dvc` (un hash) que se sube a Git.
3. Cuando un compañero clona el repo, hace `dvc pull` y el sistema descarga la versión exacta del dataset asociada a ese commit de código.

#python-env #poetry #ruff #dvc #software-engineering #clean-code
