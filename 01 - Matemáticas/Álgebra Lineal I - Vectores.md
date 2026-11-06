# 📐 Álgebra Lineal I: Vectores, Espacios y la Geometría de la Información | Vectors, Spaces & Information Geometry

## 1. The Vector: From Geometry to Semantics | El Vector: De la Geometría a la Semántica

**ES:** En el nivel básico, un vector es una flecha en el espacio. En el nivel experto, un vector es un **punto en un espacio de características (Feature Space)**. Cada dimensión del vector representa una propiedad del dato. En los LLMs, esto se conoce como **Embeddings**.

**EN:** At a basic level, a vector is an arrow in space. At an expert level, a vector is a **point in a feature space**. Each dimension of the vector represents a property of the data. In LLMs, this is known as **Embeddings**.

### 🔬 Deep Dive: The Embedding Space
Cuando un modelo como GPT-4 representa la palabra "Apple", no usa un ID, sino un vector en $\mathbb{R}^{d}$ (donde $d$ puede ser 1536). 
- **Semantic Proximity:** Palabras con significados similares están "cerca" en este espacio.
- **Vector Arithmetic:** El famoso ejemplo $\text{vec}(\text{"King"}) - \text{vec}(\text{"Man"}) + \text{vec}(\text{"Woman"}) \approx \text{vec}(\text{"Queen"})$ demuestra que el espacio vectorial captura relaciones semánticas lineales.

---

## 2. Metric Spaces & Norms | Espacios Métricos y Normas

Para medir la "distancia" o "similitud" entre dos vectores, necesitamos una **Norma ($\|\cdot\|$)**. No todas las distancias son iguales.

### A. $\ell_2$ Norm (Euclidean Norm) | Norma Euclidiana
Es la distancia "en línea recta".
$$\|\mathbf{x}\|_2 = \sqrt{\sum_{i=1}^n x_i^2}$$
- **Uso:** Distancias físicas, K-Means clustering.
- **Problema:** Muy sensible a valores atípicos (outliers) debido al cuadrado.

### B. $\ell_1$ Norm (Manhattan Norm) | Norma Manhattan
La suma de las diferencias absolutas.
$$\|\mathbf{x}\|_1 = \sum_{i=1}^n |x_i|$$
- **Uso:** Regresión LASSO (promueve la **sparsity**, es decir, fuerza a que muchos pesos sean exactamente 0).

### C. Cosine Similarity | Similitud del Coseno
En alta dimensionalidad, la magnitud del vector importa menos que su **dirección**.
$$\text{sim}(\mathbf{a}, \mathbf{b}) = \frac{\mathbf{a} \cdot \mathbf{b}}{\|\mathbf{a}\|_2 \|\mathbf{b}\|_2} = \cos(\theta)$$
- **Uso:** RAG (Retrieval Augmented Generation). Compara la pregunta del usuario con fragmentos de documentos. Si $\cos(\theta) \approx 1$, el contenido es semánticamente similar.

---

## 3. Linear Combinations, Span & Basis | Combinaciones Lineales, Span y Base

Este es el concepto que separa al programador del ingeniero de IA.

- **Linear Combination:** $\mathbf{v} = c_1\mathbf{v}_1 + c_2\mathbf{v}_2 + \dots + c_n\mathbf{v}_n$. Es la base de cómo las capas de una red neuronal mezclan información.
- **Span (Espacio Generado):** El conjunto de todos los vectores que puedes alcanzar combinando linealmente un grupo de vectores.
- **Basis (Base):** El conjunto mínimo de vectores que "generan" todo el espacio. 
    - **Expert Note:** Cuando hacemos PCA (Principal Component Analysis), lo que hacemos es buscar una **nueva base** donde la primera dimensión tenga la máxima varianza de los datos.

---

## 4. The Curse of Dimensionality | La Maldición de la Dimensionalidad

**ES:** A medida que aumentamos las dimensiones ($d \to \infty$), ocurren fenómenos contraintuitivos:
1. **Concentración de Distancias:** La distancia entre el punto más cercano y el más lejano se vuelve casi la misma. La noción de "cercanía" desaparece.
2. **Empty Space:** Los datos se vuelven extremadamente dispersos. Necesitarías una cantidad exponencial de datos para llenar el espacio.
3. **Solución:** **Dimensionality Reduction** (SVD, PCA, Autoencoders) para proyectar los datos en un "manifold" (variedad) de menor dimensión.

---

## 💻 Professional Implementation (NumPy & Performance)

Para un experto, `np.dot` es solo la superficie. La eficiencia real viene de la **Vectorización**.

```python
import numpy as np

# Generamos 10,000 embeddings aleatorios de 768 dimensiones
embeddings = np.random.randn(10000, 768).astype('float32')
query_vector = np.random.randn(768).astype('float32')

# ❌ MAL: Bucle for (Lento, Python overhead)
# for e in embeddings: 
#     dist = np.linalg.norm(e - query_vector)

# ✅ BIEN: Vectorización (Llamada a BLAS/LAPACK en C)
# Calculamos la similitud del coseno de la query contra TODOS los embeddings a la vez
norms = np.linalg.norm(embeddings, axis=1)
query_norm = np.linalg.norm(query_vector)

similarities = np.dot(embeddings, query_vector) / (norms * query_norm)

# Top 5 resultados más similares
top_5_idx = np.argsort(similarities)[-5:][::-1]
print(f"Top indices: {top_5_idx}")
```

### ⚡ Performance Tips for Experts:
1. **Float Precision:** Usa `float32` en lugar de `float64` para reducir el uso de memoria a la mitad y ganar velocidad en GPU.
2. **Broadcasting:** NumPy permite operar matrices y vectores de diferentes tamaños siguiendo reglas estrictas. Dominar el broadcasting evita el uso de bucles.
3. **Contiguous Memory:** `np.ascontiguousarray()` puede acelerar las operaciones al asegurar que los datos estén juntos en la memoria RAM.

---

## 🔗 Connection to LLMs & Modern AI

En la arquitectura de **Attention (Transformer)**, el vector de entrada $\mathbf{x}$ se multiplica por tres matrices ($\mathbf{W}_Q, \mathbf{W}_K, \mathbf{W}_V$) para crear tres vectores nuevos: **Query, Key y Value**.
La "atención" es básicamente un cálculo de **similitud del coseno (producto punto)** entre la Query y todas las Keys del contexto para decidir qué información es relevante.

#math #linear-algebra #embeddings #high-dimensional-geometry #numpy #performance #llm
