# 🐳 Docker y Contenerización | Docker & Containerization

## ¿Qué es? | What is it?
**ES:** Docker es una plataforma que permite empaquetar una aplicación y todas sus dependencias (librerías, configuración, sistema operativo) en un "contenedor". Esto resuelve el problema clásico de: *"En mi máquina funciona, pero en la tuya no"*.

**EN:** Docker is a platform that allows packaging an application and all its dependencies (libraries, configuration, OS) into a "container". This solves the classic problem of: *"It works on my machine, but not on yours"*.

---

## 🏗️ Conceptos Fundamentales | Core Concepts

### 1. Imagen (Image)
Es un archivo estático y de solo lectura que contiene la "receta" para crear un contenedor. Incluye el sistema operativo base, el código y las librerías.
- *Ejemplo:* Una imagen de `python:3.9-slim` con Pandas y Scikit-Learn ya instalados.

### 2. Contenedor (Container)
Es la instancia ejecutable de una imagen. Es un entorno aislado y ligero que corre el código.

### 3. Dockerfile
Un archivo de texto con las instrucciones paso a paso para construir una imagen.

---

## 🛠️ Ejemplo de un Dockerfile para Data Science | Technical Implementation

Imagina que quieres desplegar un modelo de ML que creaste en la carpeta `06`.

#### 💻 Código del `Dockerfile`:
```dockerfile
# 1. Usar una imagen base de Python ligera
FROM python:3.9-slim

# 2. Establecer el directorio de trabajo dentro del contenedor
WORKDIR /app

# 3. Copiar el archivo de dependencias y el código
COPY requirements.txt .
COPY model.py .

# 4. Instalar las librerías necesarias
RUN pip install --no-cache-dir -r requirements.txt

# 5. Comando para ejecutar la aplicación al iniciar el contenedor
CMD ["python", "model.py"]
```

#### 💻 Comandos Básicos de Terminal:
| Acción | Comando | Descripción |
| :--- | :--- | :--- |
| **Construir** | `docker build -t mi-modelo-ml .` | Crea la imagen a partir del Dockerfile. |
| **Ejecutar** | `docker run mi-modelo-ml` | Inicia un contenedor basado en la imagen. |
| **Listar** | `docker ps` | Muestra los contenedores activos. |
| **Detener** | `docker stop <id>` | Detiene un contenedor específico. |

---

## Aplicación Real en Datos | Real-world Data Application

**Caso: Despliegue de un API de Predicciones**
1. Creas un modelo de ML en la carpeta `06`.
2. Lo envuelves en una API con **FastAPI** (en la carpeta `02`).
3. Creas un **Docker Image** con todo el entorno.
4. Subes la imagen a la nube (AWS/Azure).
5. Ahora, cualquier aplicación en el mundo puede enviar datos a tu contenedor y recibir una predicción, sin importar si el servidor usa Linux o Windows.

---

## Conceptos Relacionados | Related Concepts
- [[08 - Ingeniería y Herramientas|Git y GitHub]] $\longleftrightarrow$ Guardas el `Dockerfile` en Git para que otros puedan recrear tu entorno.
- [[02 - Programación]] $\longleftrightarrow$ Docker elimina la necesidad de pelear con los `virtualenvs` manuales en cada máquina.

---
#docker #containers #devops #deployment
