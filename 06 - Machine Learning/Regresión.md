# 📉 Aprendizaje Supervisado: Regresión | Supervised Learning: Regression

## ¿Qué es? | What is it?
**ES:** La regresión es un tipo de aprendizaje supervisado donde el objetivo es predecir un valor numérico continuo. Se busca encontrar la función matemática que mejor describa la relación entre una o más variables de entrada ($X$) y una variable de salida ($y$).

**EN:** Regression is a type of supervised learning where the goal is to predict a continuous numerical value. The aim is to find the mathematical function that best describes the relationship between one or more input variables ($X$) and an output variable ($y$).

---

## 📐 Regresión Lineal Simple | Simple Linear Regression

Es el modelo más básico: una línea recta que intenta pasar lo más cerca posible de todos los puntos de datos.

### 📝 Formulación Matemática:
La ecuación de la recta es:
$$y = \beta_0 + \beta_1 x + \epsilon$$
Donde:
- $y$: Valor predicho (Variable dependiente).
- $x$: Valor de entrada (Variable independiente).
- $\beta_0$: Intercepto (donde la línea toca el eje Y).
- $\beta_1$: Pendiente (cuánto cambia $y$ por cada unidad que aumenta $x$).
- $\epsilon$: Error o residuo (la diferencia entre el valor real y la predicción).

### 📉 Función de Costo: MSE (Mean Squared Error)
Para saber si la línea es "buena", minimizamos el error cuadrático medio:
$$MSE = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$$
*El objetivo del modelo es encontrar el $\beta_0$ y $\beta_1$ que hagan que el MSE sea lo más pequeño posible.*

---

## 🛠️ Implementación Técnica con Scikit-Learn | Technical Implementation

#### 💻 Código en Python:
```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score, mean_absolute_error

# 1. Datos sintéticos: Horas de estudio vs Calificación
X = np.array([[1], [2], [3], [4], [5]]) # Horas (debe ser 2D)
y = np.array([45, 55, 70, 80, 92])       # Nota

# 2. Entrenar el modelo
model = LinearRegression()
model.fit(X, y)

# 3. Obtener los coeficientes
intercepto = model.intercept_
pendiente = model.coef_[0]
print(f"Ecuación: y = {intercepto:.2f} + {pendiente:.2f}x")

# 4. Predicción
nueva_hora = np.array([[3.5]])
prediccion = model.predict(nueva_hora)
print(f"Para 3.5 horas, la nota predicha es: {prediccion[0]:.2f}")

# 5. Evaluación: R² (Coeficiente de Determinación)
# Indica qué porcentaje de la variación de y es explicada por x
r2 = r2_score(y, model.predict(X))
print(f"Precisión del modelo (R²): {r2:.2f}")
```

---

## Aplicación Real en Ingeniería | Real-world Engineering Application

**Caso: Desgaste de Herramientas de Corte**
- **$X$**: Tiempo de uso de la herramienta (horas).
- **$y$**: Grosor del filo (mm).
- **Objetivo**: Predecir exactamente en qué hora el grosor llegará a un límite crítico para programar el cambio de la herramienta antes de que falle la pieza.

---

## Conceptos Relacionados | Related Concepts
- [[01 - Matemáticas|Cálculo y Optimización]] $\longleftrightarrow$ El modelo usa el "Descenso del Gradiente" para minimizar el MSE.
- [[04 - Análisis de Datos|Análisis Exploratorio (EDA)]] $\rightarrow$ Antes de hacer regresión, usamos un Scatter Plot para ver si la relación es realmente lineal.

---
#machine-learning #regression #scikit-learn #predictive
