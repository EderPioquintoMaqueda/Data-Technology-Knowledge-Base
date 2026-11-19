# 📐 Álgebra Lineal II: Matrices, Operadores y Descomposiciones | Matrices, Operators & Decompositions

## 1. The Matrix as a Linear Operator | La Matriz como Operador Lineal

**ES:** En el nivel básico, una matriz es una tabla de datos. En el nivel experto, una matriz es un **operador lineal**. Multiplicar un vector $\mathbf{x}$ por una matriz $\mathbf{A}$ es ejecutar una transformación geométrica: el vector $\mathbf{x}$ es movido, rotado o escalado a una nueva posición en el espacio.

**EN:** At a basic level, a matrix is a data table. At an expert level, a matrix is a **linear operator**. Multiplying a vector $\mathbf{x}$ by a matrix $\mathbf{A}$ is executing a geometric transformation: the vector $\mathbf{x}$ is moved, rotated, or scaled to a new position in space.

### 🔬 Deep Dive: The Geometry of Transformation
- **Scaling:** Una matriz diagonal $\text{diag}(s_1, s_2, \dots, s_n)$ estira o encoge el espacio en cada eje.
- **Rotation:** Matrices ortogonales ($\mathbf{Q}^T\mathbf{Q} = \mathbf{I}$) rotan el espacio sin cambiar las distancias ni los ángulos.
- **Projection:** Proyectar un vector de $\mathbb{R}^{1000}$ a $\mathbb{R}^{10}$ es básicamente lo que hace un encoder de un LLM para comprimir información.

---

## 2. Determinants, Rank & Information Collapse | Determinantes, Rango y Colapso de Información

El **determinante** $\det(\mathbf{A})$ no es solo un número, es el **factor de escala del volumen**.

- **$\det(\mathbf{A}) = 2$:** La transformación duplica el área/volumen del espacio original.
- **$\det(\mathbf{A}) = 0$:** El espacio colapsa. Una dimensión desaparece. La matriz es **singular** (no invertible).
- **Rank (Rango):** Es la dimensión del espacio generado por las columnas de la matriz. 
    - **Full Rank:** No hay información redundante.
    - **Low Rank:** Hay correlación lineal entre las variables. En IA, buscamos representaciones de **bajo rango** (Low-Rank Adaptation o **LoRA**) para entrenar modelos gigantes usando muy pocos parámetros.

---

## 3. The "Holy Trinity" of Matrix Decompositions | La Santísima Trinidad de las Descomposiciones

Este es el corazón del análisis de datos avanzado. Descomponer una matriz es como "factorizar" un número para ver sus componentes primos.

### A. Eigen-Decomposition (Autovalores y Autovectores)
Si $\mathbf{A}\mathbf{v} = \lambda \mathbf{v}$, entonces $\mathbf{v}$ es un **autovector** y $\lambda$ es un **autovalor**.
- **Significado:** El autovector $\mathbf{v}$ es una dirección que **no cambia de orientación** durante la transformación; solo se escala por $\lambda$.
- **Aplicación:** PCA (Principal Component Analysis). Los autovectores con los $\lambda$ más grandes son las direcciones de máxima varianza de tus datos.

### B. SVD (Singular Value Decomposition) $\rightarrow$ El arma definitiva
Cualquier matriz $\mathbf{A}$ (incluso las no cuadradas) puede descomponerse en $\mathbf{A} = \mathbf{U \Sigma V}^T$.
- $\mathbf{U}$: Autovectores del espacio de salida.
- $\mathbf{\Sigma}$: Valores singulares (importancia de cada dimensión).
- $\mathbf{V}^T$: Autovectores del espacio de entrada.
- **Uso Experto:** Compresión de imágenes, eliminación de ruido y motores de recomendación (Latent Semantic Analysis).

### C. QR Decomposition
$\mathbf{A} = \mathbf{QR}$ donde $\mathbf{Q}$ es ortogonal y $\mathbf{R}$ es triangular superior.
- **Uso:** Es la forma numéricamente estable de resolver mínimos cuadrados ($\text{least squares}$) y calcular autovalores.

---

## 4. From Matrices to Tensors | De Matrices a Tensores

Un tensor es la generalización de los escalares, vectores y matrices.
- **Escalar:** Tensor de orden 0.
- **Vector:** Tensor de orden 1.
- **Matriz:** Tensor de orden 2.
- **Tensors ($\mathbb{R}^{d_1 \times d_2 \times \dots \times d_n}$):** Tensor de orden $n$.

### 🧠 Memory Architecture (Stride & Shape)
Las librerías como PyTorch no guardan tensores como "cubos" en la memoria, sino como un **bloque lineal de datos (1D array)**. 
- **Shape:** Las dimensiones del tensor.
- **Stride:** Cuántos elementos saltar en la memoria lineal para llegar al siguiente elemento de una dimensión.
- **Expert Note:** Cambiar el `shape` mediante `view()` o `reshape()` es casi instantáneo porque no mueve los datos, solo cambia la interpretación del *stride*.

---

## 💻 Professional Implementation (Numerical Stability)

A nivel experto, nunca calculamos la inversa de una matriz $\mathbf{A}^{-1}$ directamente porque es computacionalmente caro y numéricamente inestable.

```python
import numpy as np

A = np.array([[1.0, 2.0], [3.0, 4.0]])
b = np.array([5.0, 11.0])

# ❌ MAL: Usar la inversa (Inestable y lento)
# x = np.linalg.inv(A).dot(b)

# ✅ BIEN: Usar solvers basados en descomposición (LU/QR)
x = np.linalg.solve(A, b) 

# 🚀 EXPERT: SVD para matrices mal condicionadas (Pseudo-inversa de Moore-Penrose)
# Útil cuando la matriz es casi singular (det approx 0)
pinv_A = np.linalg.pinv(A)
x_stable = pinv_A.dot(b)

print(f"Solución estable: {x_stable}")
```

### ⚡ Computational Complexity (Big O)
- **Multiplicación de matrices:** $O(n^3)$ estándar. 
- **Algoritmo de Strassen:** $\approx O(n^{2.807})$.
- **GPU Acceleration:** Las GPUs no son más "inteligentes", sino que realizan miles de multiplicaciones y sumas simultáneamente (**SIMD - Single Instruction, Multiple Data**), reduciendo el tiempo real de ejecución de horas a segundos.

---

## 🔗 Connection to LLMs: The Attention Matrix

En un Transformer, la operación central es:
$$\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\left(\frac{\mathbf{QK}^T}{\sqrt{d_k}}\right)\mathbf{V}$$
1. $\mathbf{QK}^T$: Un producto punto masivo que crea una **matriz de afinidad** (quién atiende a quién).
2. $\sqrt{d_k}$: Escalamiento para evitar que los valores del softmax sean demasiado extremos (evitando el *vanishing gradient*).
3. Multiplicación por $\mathbf{V}$: Una suma ponderada de los vectores de valor basada en la afinidad calculada.

#math #linear-algebra #SVD #tensors #numpy #deep-learning #llm #performance
