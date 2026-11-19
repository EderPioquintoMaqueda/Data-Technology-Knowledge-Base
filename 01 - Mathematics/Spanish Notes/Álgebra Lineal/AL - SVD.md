# 🧬 AL - SVD: Descomposición en Valores Singulares (Análisis Profesional)

La **SVD (Singular Value Decomposition)** es la culminación del álgebra lineal aplicada. A diferencia de la descomposición en autovalores ($\mathbf{A} = \mathbf{V}\mathbf{\Lambda}\mathbf{V}^{-1}$), que requiere que la matriz sea cuadrada y diagonalizable, la SVD es universal: **cualquier matriz $\mathbf{A} \in \mathbb{R}^{m \times n}$ puede ser descompuesta.**

## 1. El Teorema Fundamental de SVD

Cualquier matriz $\mathbf{A}$ puede factorizarse en:
$$\mathbf{A} = \mathbf{U} \mathbf{\Sigma} \mathbf{V}^\top$$

### Desglose Técnico:
- $\mathbf{U} \in \mathbb{R}^{m \times m}$: Matriz ortogonal ($\mathbf{UU}^\top = \mathbf{I}$). Sus columnas son los **vectores singulares izquierdos**. Representan la base ortogonal del espacio de salida (espacio de las filas).
- $\mathbf{\Sigma} \in \mathbb{R}^{m \times n}$: Matriz rectangular diagonal. Contiene los **valores singulares** $\sigma_1 \ge \sigma_2 \ge \dots \ge \sigma_k \ge 0$. Estos valores cuantifican la "energía" o importancia de cada componente.
- $\mathbf{V}^\top \in \mathbb{R}^{n \times n}$: Matriz ortogonal. Sus columnas (filas de $\mathbf{V}^\top$) son los **vectores singulares derechos**. Representan la base ortogonal del espacio de entrada (espacio de las columnas).

---

## 2. Interpretación en Ciencia de Datos: Compresión y Filtrado

Desde una perspectiva profesional, la SVD es una herramienta de **filtrado de ruido**. 

### La Aproximación de Bajo Rango (Eckart-Young-Mirsky Theorem)
Si queremos la mejor aproximación de $\mathbf{A}$ usando una matriz de rango $k < \text{rank}(\mathbf{A})$, simplemente tomamos los $k$ valores singulares más grandes de $\mathbf{\Sigma}$ y ponemos el resto en cero.
$$\mathbf{A}_k = \sum_{i=1}^{k} \sigma_i \mathbf{u}_i \mathbf{v}_i^\top$$
Esto elimina las dimensiones que contienen poca varianza, que generalmente corresponden al **ruido blanco** del dataset.

### 🏦 Caso de Uso Profesional: LSA (Latent Semantic Analysis)
En el análisis de documentos o perfiles bancarios:
- $\mathbf{A}$ es una matriz $\text{Documentos} \times \text{Términos}$.
- $\mathbf{U}$ nos dice qué documentos están relacionados con qué "conceptos latentes".
- $\mathbf{V}$ nos dice qué palabras definen esos "conceptos latentes".
- $\mathbf{\Sigma}$ nos dice qué tan fuerte es cada concepto en el corpus global.

---

## 💻 Implementación de Alta Performance (Python)

Para datasets masivos, no usamos `np.linalg.svd` (que es costoso), sino **SVD Truncada** o **Randomized SVD**.

```python
import numpy as np
from sklearn.decomposition import TruncatedSVD

# Matriz de alta dimensionalidad (ej. 10,000 clientes x 500 variables)
A = np.random.randn(10000, 500)

# SVD Profesional: Solo calculamos los 50 componentes más importantes
# Esto reduce drásticamente la memoria y el tiempo de cómputo
svd = TruncatedSVD(n_components=50)
A_reduced = svd.fit_transform(A)

print(f"Original: {A.shape} -> Reducido: {A_reduced.shape}")
print(f"Varianza explicada total: {np.sum(svd.explained_variance_ratio_):.2%}")
```

---

## 🎯 Análisis Crítico para Entrevistas Técnicas
**Pregunta**: ¿Cuál es la relación entre SVD y PCA?
**Respuesta**: El PCA es un caso particular de SVD. Si centramos los datos (restamos la media) y calculamos la SVD de la matriz de datos $\mathbf{X}$, los vectores singulares derechos $\mathbf{V}$ son exactamente los autovectores de la matriz de covarianza $\mathbf{C} = \frac{1}{n-1}\mathbf{X}^\top\mathbf{X}$.

#matematicas #algebra-lineal #svd #dimensionality-reduction #lsa #numpy #sklearn
