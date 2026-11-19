# 🐳 Docker y MLOps: Contenerización y el Ciclo de Vida de la IA | Docker & MLOps: Containerization & AI Lifecycle

**ES:** En el nivel básico, Docker es una herramienta para "evitar errores de instalación". En el nivel experto, Docker es la unidad fundamental de despliegue de la IA. La **Contenerización** permite que la infraestructura sea tratada como código (Infrastructure as Code), asegurando que el modelo se comporte exactamente igual en la laptop del científico que en el cluster de GPUs de la nube.

**EN:** At a basic level, Docker is a tool to "avoid installation errors". At an expert level, Docker is the fundamental unit of AI deployment. **Containerization** allows infrastructure to be treated as code (Infrastructure as Code), ensuring that the model behaves exactly the same on the scientist's laptop as it does in a cloud GPU cluster.

---

## 1. Advanced Image Optimization | Optimización de Imágenes para Producción

Una imagen de ML mal construida puede pesar 10GB, lo que hace que el despliegue sea lento y costoso. El experto utiliza **Multi-stage Builds**.

### A. Multi-stage Builds (Construcción Multietapa)
Consiste en usar una imagen pesada para compilar y una imagen ultra-ligera para ejecutar.

```dockerfile
# --- ETAPA 1: Builder (Pesada) ---
FROM python:3.11-slim AS builder
WORKDIR /build
COPY requirements.txt .
# Instalamos dependencias en una carpeta temporal
RUN pip install --user --no-cache-dir -r requirements.txt

# --- ETAPA 2: Runner (Ligera) ---
FROM python:3.11-slim
WORKDIR /app
# Solo copiamos las librerías ya instaladas desde la etapa builder
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.//local/bin:$PATH
CMD ["python", "main.py"]
```
- **Impacto:** Reducción del tamaño de la imagen de GBs a MBs, eliminando compiladores y archivos temporales.

### B. GPU-Acceleration with NVIDIA Docker
Para que la IA use la GPU dentro de un contenedor, no basta con instalar drivers en la máquina; el contenedor necesita el **NVIDIA Container Toolkit**.
- **Imagen Base:** Se debe usar `nvidia/cuda` en lugar de `python:slim`.
- **Ejecución:** Se debe usar el flag `--gpus all` al correr el contenedor.

---

## 2. Orchestration with Docker Compose | Orquestación con Docker Compose

En la vida real, una IA no vive sola; necesita una base de datos, una cola de mensajes y una API. **Docker Compose** permite gestionar este "ecosistema" con un solo archivo `.yml`.

### Ejemplo de Arquitectura de ML:
```yaml
version: '3.8'
services:
  api:
    build: ./api
    ports: ["8000:8000"]
    depends_on: ["db", "redis"]
    environment:
      - REDIS_HOST=redis
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
  redis:
    image: redis:alpine
```
- **Ventaja:** Levantas todo el entorno de producción con `docker-compose up -d`, eliminando la configuración manual de servicios.

---

## 3. MLOps: The Professional Lifecycle | El Ciclo de Vida de MLOps

MLOps es la aplicación de DevOps al Machine Learning. El objetivo es cerrar la brecha entre el entrenamiento y la producción.

### El Pipeline de MLOps:
1. **Continuous Integration (CI):** El código se sube a Git $\rightarrow$ Se ejecuten tests $\rightarrow$ Se construye la Imagen de Docker.
2. **Continuous Delivery (CD):** La imagen se sube a un Registry (DockerHub/AWS ECR) $\rightarrow$ Se despliega en el cluster (Kubernetes).
3. **Continuous Monitoring (CM):** Monitorización del modelo en vivo.
    - **Data Drift:** Cuando los datos del mundo real cambian y el modelo deja de ser preciso.
    - **Concept Drift:** Cuando la relación entre la entrada y la salida cambia (ej. comportamiento de compra post-pandemia).
4. **Retraining Loop:** Si el monitoreo detecta Drift $\rightarrow$ Se dispara automáticamente un nuevo entrenamiento $\rightarrow$ Se despliega la nueva versión.

---

## 💻 Professional Implementation: The MLOps Wrapper

Un experto no corre `python main.py`. Empaqueta el modelo en un contenedor y lo despliega mediante un orquestador.

```bash
# 1. Construir la imagen optimizada
docker build -t ml-service:v1 .

# 2. Ejecutar con acceso a la GPU y puerto expuesto
docker run --gpus all -p 8000:8000 ml-service:v1
```

#docker #containers #devops #deployment #mlops #kubernetes #gpu-computing
