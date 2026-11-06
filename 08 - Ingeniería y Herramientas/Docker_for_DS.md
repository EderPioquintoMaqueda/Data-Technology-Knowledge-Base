# 🐳 Docker para Data Science: Portabilidad y Escalabilidad

El problema más común en la industria es: *"En mi computadora funciona, pero en el servidor no"*. Esto ocurre por diferencias en las versiones de Python, librerías o el sistema operativo. **Docker** soluciona esto creando un "contenedor" que incluye todo lo necesario para que el código corra exactamente igual en cualquier lugar.

## 1. Conceptos Fundamentales

### Imagen vs. Contenedor
- **Imagen**: Es la "receta" o plantilla. Contiene el SO, Python, las librerías (`requirements.txt`) y el código. Es inmutable.
- **Contenedor**: Es la instancia viva de la imagen. Es el proceso ejecutándose en el servidor.

---

## 2. Creación de un Dockerfile Profesional para ML

Un `Dockerfile` mal escrito crea imágenes de 5GB que tardan minutos en cargar. Un profesional optimiza la imagen.

### Ejemplo de Dockerfile Optimizado:
```dockerfile
# 1. Usar una imagen base ligera (Slim) para reducir el tamaño
FROM python:3.9-slim

# 2. Establecer el directorio de trabajo
WORKDIR /app

# 3. Instalar dependencias del sistema necesarias (compiladores C para NumPy/Pandas)
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# 4. Copiar solo el requirements primero para aprovechar el cache de Docker
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copiar el resto del código
COPY . .

# 6. Exponer el puerto de la API (FastAPI)
EXPOSE 8000

# 7. Comando de ejecución con Uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 3. Orquestación y Despliegue

### Docker Compose
Cuando tu sistema necesita más que solo la API (ej. la API + una base de datos PostgreSQL + una Vector DB), usas `docker-compose.yml` para levantar todo el ecosistema con un solo comando: `docker-compose up`.

### Flujo de Despliegue Profesional:
$\text{Código} \to \text{Git Push} \to \text{CI/CD Pipeline} \to \text{Build Docker Image} \to \text{Push to Registry (DockerHub/AWS ECR)} \to \text{Deploy to Cloud (K8s/ECS)}$.

---

## 🎯 Resumen de Ventajas para el Ingeniero de Datos
1. **Aislamiento**: Puedes tener un proyecto con Python 3.7 y otro con 3.11 en la misma máquina sin conflictos.
2. **Escalabilidad**: Si tu API de IA tiene mucho tráfico, puedes levantar 10 contenedores idénticos detrás de un balanceador de carga.
3. **Consistencia**: El modelo que entrenaste en tu PC es exactamente el mismo que se ejecuta en el servidor de producción.

#docker #mlops #deployment #cloud-computing #devops
