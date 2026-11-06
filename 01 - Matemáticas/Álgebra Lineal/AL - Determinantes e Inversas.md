# ⚖️ AL - Determinantes e Inversas: Estabilidad y Soluciones

En la ciencia de datos, no todas las matrices son "útiles". Algunas matrices contienen información redundante o contradictoria que puede hacer que nuestros modelos matemáticos colapsen. El determinante y la inversa son las herramientas para detectar estos problemas.

## 1. El Determinante ($\det(\mathbf{A})$)
El determinante es un número escalar que nos dice cuánto se "estira" o "encoge" el espacio después de una transformación lineal.

- **Interpretación Geométrica**: 
    - Si $\det(\mathbf{A}) = 0$, el espacio se colapsa en una dimensión menor (una línea o un punto). La matriz es **Singular**.
    - Si $\det(\mathbf{A}) \neq 0$, la transformación es reversible.

### 🏦 Caso Bancario: El Colapso de la Información
Imagina que estás construyendo un modelo de riesgo y usas dos variables: $\text{Sueldo en USD}$ y $\text{Sueldo en Euros}$. 
Como una es solo la otra multiplicada por el tipo de cambio, la matriz de datos resultante tendrá un **Determinante = 0**. 
**Consecuencia**: El modelo no puede "distinguir" entre las dos variables, lo que provoca errores matemáticos al intentar calcular los pesos del modelo (división por cero).

---

## 2. La Matriz Inversa ($\mathbf{A}^{-1}$)
La inversa es la matriz que "deshace" la acción de la matriz original. 
$$\mathbf{A} \cdot \mathbf{A}^{-1} = \mathbf{I}$$
Donde $\mathbf{I}$ es la **Matriz Identidad** (el "1" de las matrices).

- **Condición Existencial**: Una matriz $\mathbf{A}$ tiene inversa **si y solo si** $\det(\mathbf{A}) \neq 0$.

### 🏦 Caso Bancario: Recuperación de Parámetros
En los modelos de Regresión Lineal (como predecir la probabilidad de default), la fórmula para hallar los coeficientes óptimos $\mathbf{w}$ es:
$$\mathbf{w} = (\mathbf{X}^\top \mathbf{X})^{-1} \mathbf{X}^\top \mathbf{y}$$
Aquí, el término $(\mathbf{X}^\top \mathbf{X})^{-1}$ requiere que la matriz sea invertible. Si tienes variables redundantes (determinante 0), el banco no puede calcular los pesos del modelo y el sistema falla.

---

## 💻 Implementación en Python (NumPy)

```python
import numpy as np

# Matriz A: Estable (Determinante != 0)
A = np.array([
    [1, 2],
    [3, 4]
])

# Matriz B: Singular/Colapsada (Determinante == 0)
# La segunda fila es el doble de la primera
B = np.array([
    [1, 2],
    [2, 4]
])

# 1. Cálculo del Determinante
det_A = np.linalg.det(A)
det_B = np.linalg.det(B)

print(f"Determinante A: {det_A:.2f}") # Salida: -2.00
print(f"Determinante B: {det_B:.2f}") # Salida: 0.00 (¡Colapsada!)

# 2. Cálculo de la Inversa
try:
    inv_A = np.linalg.inv(A)
    print("Inversa de A calculada con éxito.")
except np.linalg.LinAlgError:
    print("A no es invertible.")

try:
    inv_B = np.linalg.inv(B)
except np.linalg.LinAlgError:
    print("Error: B es una matriz singular y NO tiene inversa.")
```

#matematicas #algebra-lineal #banking #stability #numpy
