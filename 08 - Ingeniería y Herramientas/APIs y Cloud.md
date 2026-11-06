# ☁️ APIs y Cloud Computing: Backend Engineering & Scalable Architecture | APIs & Cloud Computing: Backend Engineering & Scalable Architecture

**ES:** En la industria, un modelo de IA que vive en un notebook de Jupyter no tiene valor. El valor real aparece cuando el modelo se convierte en un **servicio accesible** mediante una API y se despliega en una arquitectura de nube escalable. La ingeniería de backend es lo que permite que la IA pase de ser un experimento a ser un producto.

**EN:** In the industry, an AI model that lives in a Jupyter notebook has no value. Real value appears when the model becomes an **accessible service** via an API and is deployed in a scalable cloud architecture. Backend engineering is what allows AI to move from an experiment to a product.

---

## 1. The Modern API Stack: FastAPI & Pydantic | El Stack Moderno de APIs

Para desplegar modelos de ML, el estándar actual es **FastAPI**. A diferencia de Flask o Django, FastAPI está construido sobre **ASGI (Asynchronous Server Gateway Interface)**, lo que le permite manejar miles de peticiones simultáneas sin bloquear el servidor.

### A. Asincronismo (`async` / `await`)
En IA, las inferencias pueden tardar segundos. Si el servidor es síncrono, un solo usuario puede bloquear a todos los demás. FastAPI usa `asyncio` para procesar otras peticiones mientras espera que la GPU termine la inferencia.

### B. Validación con Pydantic
FastAPI utiliza **Pydantic** para definir la forma de los datos.
- **Tipado Estricto:** Si la API espera un `float` y recibe un `string`, FastAPI devuelve un error 422 automáticamente antes de que el dato llegue al modelo, evitando crashes en producción.

### C. REST vs gRPC vs GraphQL
| Tecnología | Estilo | Uso Ideal | Ventaja Técnica |
| :--- | :--- | :--- | :--- |
| **REST** | Recurso-céntrico | APIs Públicas, Web | Universal, simple, basado en JSON. |
| **gRPC** | Procedural | Comunicación Interna (Microservicios) | Usa **Protocol Buffers** (binario), es órdenes de magnitud más rápido que JSON. |
| **GraphQL** | Grafo-céntrico | Frontends Complejos | El cliente pide exactamente los campos que necesita; evita el *over-fetching*. |

---

## 2. Cloud Architecture for AI | Arquitectura de Nube para IA

El despliegue de IA requiere una gestión inteligente de los recursos (CPU vs GPU).

### A. Serverless vs Kubernetes (K8s)
- **Serverless (AWS Lambda / Azure Functions):**
    - **Cuándo:** Procesos ligeros, disparados por eventos (ej. procesar una imagen cuando se sube a S3).
    - **Pro:** Cero mantenimiento de servidores, pago por ejecución.
    - **Contra:** *Cold Start* (latencia en la primera llamada) y límites de RAM/Tiempo.
- **Kubernetes (K8s):**
    - **Cuándo:** Modelos masivos (LLMs) que requieren GPUs dedicadas y escalado automático.
    - **Pro:** Control total sobre el hardware, orquestación de contenedores.
    - **Contra:** Curva de aprendizaje altísima y costo de mantenimiento.

### B. CI/CD Pipelines for ML (MLOps)
Un despliegue profesional no es un `git push` manual. Sigue un flujo automatizado:
$\text{Commit} \rightarrow \text{Unit Tests} \rightarrow \text{Model Validation} \rightarrow \text{Docker Build} \rightarrow \text{Deploy to Staging} \rightarrow \text{Canary Deploy} \rightarrow \text{Production}$.

---

## 💻 Professional Implementation: FastAPI for ML Deployment

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import torch
import numpy as np

# 1. Definimos la estructura de los datos con Pydantic
class PredictionRequest(BaseModel):
    features: list[float]
    model_version: str = "v1"

app = FastAPI(title="ML Production API")

# Simulamos un modelo cargado en memoria
model = lambda x: np.sum(x) * 0.5 # Representación simplificada

@app.post("/predict")
async def predict(request: PredictionRequest):
    try:
        # Convertir la lista de Pydantic a tensor de PyTorch
        data = torch.tensor(request.features).float()
        
        # Inferencia asíncrona (simulada)
        prediction = model(data.numpy())
        
        return {
            "prediction": float(prediction),
            "status": "success",
            "version": request.model_version
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# Para ejecutar: uvicorn main:app --reload
```

---

## 🎯 Reto de Arquitectura Cloud
**Escenario:** Tienes un modelo de detección de objetos que procesa videos en tiempo real. Tienes picos de tráfico: 10 usuarios a las 3 AM y 10,000 usuarios a las 3 PM.
1. **¿Qué modo de despliegue usarías?** (Respuesta: Kubernetes con **HPA (Horizontal Pod Autoscaler)** para escalar el número de contenedores según la carga de la GPU).
2. **¿Cómo evitarías que la API colapse durante el pico de las 3 PM?** (Respuesta: Implementando una **Cola de Mensajes** como RabbitMQ o Apache Kafka para procesar las solicitudes de forma asíncrona).
3. **¿Dónde guardarías los videos procesados?** (Respuesta: Object Storage como AWS S3 o Azure Blob Storage, ya que es la opción más barata y escalable para datos no estructurados).

#cloud #api #fastapi #pydantic #kubernetes #mlops #backend-engineering
