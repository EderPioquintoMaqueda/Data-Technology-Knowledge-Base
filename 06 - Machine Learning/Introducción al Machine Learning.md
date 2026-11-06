# 🤖 Introducción al Machine Learning | Introduction to Machine Learning

## ¿Qué es? | What is it?
**ES:** El Machine Learning (ML) es una rama de la Inteligencia Artificial que permite que las computadoras aprendan patrones a partir de los datos sin ser programadas explícitamente para cada tarea. En lugar de escribir reglas fijas (`if/else`), creamos un algoritmo que encuentra la regla matemática por sí mismo.

**EN:** Machine Learning (ML) is a branch of Artificial Intelligence that allows computers to learn patterns from data without being explicitly programmed for every task. Instead of writing fixed rules (`if/else`), we create an algorithm that finds the mathematical rule on its own.

---

## ⚙️ El Cambio de Paradigma | The Paradigm Shift

| Programación Tradicional | Machine Learning |
| :--- | :--- |
| **Entrada:** Datos + Reglas $\rightarrow$ **Salida:** Resultado | **Entrada:** Datos + Resultados $\rightarrow$ **Salida:** Regla (Modelo) |
| El humano define la lógica. | El algoritmo descubre la lógica. |
| Ejemplo: `if temperatura > 30: alerta()` | Ejemplo: Analiza 10,000 fallas $\rightarrow$ "Aprende" que la temperatura es la causa. |

---

## 🏫 Tipos de Aprendizaje | Types of Learning

El ML se divide principalmente según la naturaleza de los datos y el objetivo:

### 1. Aprendizaje Supervisado (Supervised Learning)
El modelo aprende con datos "etiquetados". Sabemos la respuesta correcta.
- **Regresión:** Predecir un valor numérico continuo (Ej: Precio de una casa, Temperatura).
- **Clasificación:** Predecir una categoría (Ej: ¿Es fraude o no?, ¿Es perro o gato?).

### 2. Aprendizaje No Supervisado (Unsupervised Learning)
El modelo busca patrones ocultos en datos "sin etiquetas". No hay respuesta correcta predefinida.
- **Clustering:** Agrupar datos similares (Ej: Segmentación de clientes).
- **Reducción de Dimensionalidad:** Simplificar datos complejos (Ej: PCA).

### 3. Aprendizaje por Refuerzo (Reinforcement Learning)
El modelo aprende mediante ensayo y error, recibiendo "recompensas" o "castigos".
- **Ejemplo:** Una IA aprendiendo a jugar ajedrez o un brazo robótico aprendiendo a sujetar un objeto.

---

## 🛠️ El Stack Técnico de ML en Python | The ML Technical Stack

Para implementar ML, la librería fundamental es **Scikit-Learn**.

#### 💻 Flujo Básico de un Modelo (The ML Pipeline):
```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

# 1. Preparación de datos (X = Características, y = Objetivo)
X = df[['feature1', 'feature2']] 
y = df['target']

# 2. División: Entrenamiento (80%) y Prueba (20%)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 3. Definición y Entrenamiento del Modelo
model = LinearRegression()
model.fit(X_train, y_train) # Aquí ocurre la "magia" del aprendizaje

# 4. Predicción y Evaluación
predictions = model.predict(X_test)
error = mean_squared_error(y_test, predictions)
print(f"Error cuadrático medio: {error:.2f}")
```

---

## Conceptos Relacionados | Related Concepts
- [[01 - Matemáticas|Cálculo y Optimización]] $\longleftrightarrow$ El entrenamiento de un modelo es básicamente optimizar una función de error.
- [[01 - Matemáticas|Álgebra Lineal]] $\longleftrightarrow$ Los datos se pasan al modelo como matrices de características.
- [[04 - Análisis de Datos|Análisis Exploratorio (EDA)]] $\rightarrow$ Un mal EDA lleva a un mal modelo ("Garbage In, Garbage Out").

---
#machine-learning #ai #scikit-learn #predictive-analytics
