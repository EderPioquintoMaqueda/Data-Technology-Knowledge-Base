# ⚡ FastAPI: Creando APIs Profesionales para Modelos de IA

FastAPI es el estándar moderno para desplegar modelos de Machine Learning. A diferencia de Flask, FastAPI es asíncrono (basado en `asyncio`), extremadamente rápido y utiliza `Pydantic` para validar que los datos que entran a tu modelo sean correctos.

## 1. Arquitectura de una API de ML

Para que un modelo sea productivo, no basta con un script; necesitas un servidor que reciba datos, los procese y devuelva una respuesta en formato JSON.

### Flujo de la Petición:
$\text{Cliente (Web/App)} \to \text{Request JSON} \to \text{FastAPI (Validación)} \to \text{Modelo de IA} \to \text{Respuesta JSON}$

---

## 2. Implementación Técnica: API de Predicción

Aquí tienes el código profesional para exponer un modelo de clasificación ya entrenado.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import joblib
import numpy as np

# 1. Definir la estructura de los datos de entrada con Pydantic
class PredictionInput(BaseModel):
    vibracion: float
    temperatura: float
    presion: float

# 2. Inicializar la API y cargar el modelo en memoria (una sola vez)
app = FastAPI(title="AI Predictive Maintenance API")
model = joblib.load("model_random_forest.pkl")

@app.get("/")
def home():
    return {"status": "API Online", "model": "Random Forest v1.2"}

@app.post("/predict")
async def predict(data: PredictionInput):
    try:
        # Convertir la entrada de Pydantic a array de NumPy
        features = np.array([[data.vibracion, data.temperatura, data.presion]])
        
        # Realizar la predicción
 la_prediccion = model.predict(features)[0]
        la_probabilidad = model.predict_proba(features)[0].max()
        
        return {
            "prediction": "Falla" if la_prediccion == 1 else "Sano",
            "confidence": f"{la_probabilidad:.2%}"
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

---

## 3. Ventajas Profesionales de FastAPI

### A. Documentación Automática (Swagger)
FastAPI genera automáticamente una interfaz web en `/docs` donde cualquier desarrollador puede probar tu modelo sin escribir una sola línea de código.

### B. Validación de Tipos
Si el usuario envía un texto en lugar de un número en `vibracion`, FastAPI devuelve automáticamente un error `422 Unprocessable Entity`, evitando que el modelo colapse.

### C. Asincronismo (`async def`)
Permite que el servidor maneje miles de peticiones simultáneas sin bloquearse, algo vital cuando el modelo de IA tarda algunos milisegundos en procesar.

#fastapi #python-api #mlops #deployment #pydantic
