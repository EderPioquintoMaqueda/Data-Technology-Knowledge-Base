# 🔄 AL - Transformaciones Lineales: Proyecciones y Espacios de Datos

Una transformación lineal es el mecanismo mediante el cual movemos datos de un espacio de representación a otro. En el análisis de datos profesional, no vemos las transformaciones como "cambios de forma", sino como **cambios de base** para optimizar la extracción de información.

## 1. Rigor Matemático

Una función $T: \mathbb{R}^n \to \mathbb{R}^m$ es una transformación lineal si para todo $\mathbf{u}, \mathbf{v} \in \mathbb{R}^n$ y escalar $c$:
1. $T(\mathbf{u} + \mathbf{v}) = T(\mathbf{u}) + T(\mathbf{v})$
2. $T(c\mathbf{v}) = cT(\mathbf{v})$

Toda transformación lineal puede representarse como $\mathbf{y} = \mathbf{A}\mathbf{x}$, donde $\mathbf{A}$ es la matriz de transformación.

---

## 2. El Concepto de Cambio de Base (Change of Basis)

Este es el concepto más importante para un Ingeniero de Datos. Los datos suelen venir en la **Base Estándar** (columnas de la DB), pero esa base no siempre es la más eficiente para el análisis.

### ¿Por qué cambiar la base?
Imagina que tus variables son "Ingresos" y "Gasto". Están muy correlacionadas. Cambiar la base mediante PCA significa crear nuevas variables (Componentes Principales) que sean **ortogonales** (no correlacionadas entre sí). 
- **Base Original**: Correlacionada, redundante.
- **Nueva Base (PCA)**: Independiente, compacta.

---

## 3. Proyecciones Ortogonales

La proyección es la base de la reducción de dimensionalidad y el ajuste de modelos lineales. Proyectar un vector $\mathbf{x}$ sobre un subespacio $W$ significa encontrar el vector $\mathbf{p} \in W$ que esté más cerca de $\mathbf{x}$.

$$\mathbf{p} = \text{proj}_W(\mathbf{x}) = \frac{\mathbf{x} \cdot \mathbf{w}}{\|\mathbf{w}\|^2} \mathbf{w}$$

### 🏦 Aplicación: Regresión Lineal como Proyección
Desde el álgebra lineal, la Regresión Lineal no es "ajustar una línea", sino **proyectar el vector de respuestas $\mathbf{y}$ sobre el espacio generado por las columnas de la matriz de características $\mathbf{X}$**. 
El error (residuo) es simplemente la parte del vector $\mathbf{y}$ que es ortogonal al espacio de $\mathbf{X}$.

---

## 🛠️ Implementación Técnica: Normalización y Escalado

Para que las transformaciones lineales funcionen, los datos deben estar en la misma escala.

### Standard Scaler (Z-Score)
$$\mathbf{z} = \frac{\mathbf{x} - \mu}{\sigma}$$
Mueve los datos al centro (media 0) y los escala por la desviación estándar. Esto es una transformación lineal (traslación y escalado).

### Min-Max Scaler
$$\mathbf{x}_{norm} = \frac{\mathbf{x} - \min(\mathbf{x})}{\max(\mathbf{x}) - \min(\mathbf{x})}$$
Mapea los datos al rango $[0, 1]$. Útil para redes neuronales que usan funciones de activación como Sigmoide.

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler
import numpy as np

data = np.array([[1000, 25], [5000, 35], [10000, 45]]) # [Sueldo, Edad]

# Transformación Profesional
scaler_z = StandardScaler()
data_z = scaler_z.fit_transform(data)

scaler_mm = MinMaxScaler()
data_mm = scaler_mm.fit_transform(data)

print(f"Z-Score (Media 0, Var 1):\n{data_z}")
print(f"Min-Max (Rango 0-1):\n{data_mm}")
```

---

## 🎯 Resumen de Conexiones Profesionales
- **SVD** $\rightarrow$ Encuentra la mejor base para comprimir la matriz.
- **Eigen-decomposition** $\rightarrow$ Encuentra los ejes de máxima varianza de la covarianza.
- **Transformaciones** $\rightarrow$ Permiten pasar de la base de la base de datos $\to$ Base de PCA $\to$ Base de Predicción.

#matematicas #algebra-lineal #linear-transformations #pca #data-scaling #sklearn
