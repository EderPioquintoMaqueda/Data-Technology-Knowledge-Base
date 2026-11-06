# 💎 AL - Eigenvalues y Eigenvectors: El Análisis de la Varianza

El estudio de los autovalores ($\lambda$) y autovectores ($\mathbf{v}$) es la base de cualquier técnica de reducción de dimensionalidad. En un entorno profesional, no los vemos como simples soluciones a un polinomio característico, sino como los **ejes de simetría de la información**.

## 1. Rigor Matemático: El Problema del Autovalor

Sea $\mathbf{A} \in \mathbb{R}^{n \times n}$ una matriz cuadrada. Un vector no nulo $\mathbf{v}$ es un autovector de $\mathbf{A}$ si:
$$\mathbf{A}\mathbf{v} = \lambda \mathbf{v}$$
Esto implica que la transformación lineal $\mathbf{A}$ solo **escala** el vector $\mathbf{v}$ por un factor $\lambda$.

### El Polinomio Característico
Para hallar $\lambda$, resolvemos el determinante de la matriz desplazada:
$$\det(\mathbf{A} - \lambda \mathbf{I}) = 0$$
Las raíces de este polinomio son los autovalores. En datasets reales, esto se resuelve mediante algoritmos iterativos (como el algoritmo de QR), ya que resolver determinantes de $1000 \times 1000$ es computacionalmente inviable.

---

## 2. Aplicación Profesional: PCA (Principal Component Analysis)

El PCA es la aplicación fundamental de este concepto para combatir la **maldición de la dimensionalidad**.

### El Flujo de Trabajo Técnico:
1. **Estandarización**: $\mathbf{X}_{std} = \frac{\mathbf{X} - \mu}{\sigma}$. Crucial porque los autovalores son sensibles a la escala.
2. **Matriz de Covarianza**: $\mathbf{C} = \frac{1}{n-1} \mathbf{X}^\top \mathbf{X}$. Esta matriz captura cómo cada variable co-varía con todas las demás.
3. **Descomposición Espectral**: Hallamos los autovalores y autovectores de $\mathbf{C}$.
   - El autovector $\mathbf{v}_1$ asociado al $\lambda_{max}$ es la dirección de **máxima varianza**.
   - $\sum \lambda_i$ es la varianza total del sistema.
4. **Proyección**: Proyectamos los datos originales sobre los $k$ primeros autovectores.

### 🏦 Caso Bancario: Segmentación de Clientes
Si tienes 100 indicadores financieros, el PCA puede revelar que el 80% de la varianza depende de solo 3 componentes:
- **Componente 1**: Solvencia General (mezcla de ingresos, activos y score).
- **Componente 2**: Apetito de Riesgo (mezcla de inversiones en crypto, acciones y derivados).
- **Componente 3**: Estabilidad Temporal (mezcla de antigüedad y recurrencia).

---

## 💻 Implementación Profesional con Scikit-Learn

En la industria, no implementamos el polinomio característico, sino que usamos implementaciones optimizadas de SVD para hacer PCA.

```python
import numpy as np
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

# Dataset simulado: 1000 clientes, 50 variables
X = np.random.randn(1000, 50)

# 1. Estandarización (Indispensable)
X_scaled = StandardScaler().fit_transform(X)

# 2. PCA configurado para mantener el 95% de la varianza
pca = PCA(n_components=0.95) 
X_pca = pca.fit_transform(X_scaled)

print(f"Variables originales: 50")
print(f"Variables necesarias para el 95% de la info: {pca.n_components_}")
print(f"Varianza explicada por componente: \n{pca.explained_variance_ratio_}")
```

---

## 🎯 Análisis Crítico: ¿Cuándo NO usar PCA?
1. **Relaciones No Lineales**: PCA asume que la estructura de los datos es lineal. Si los datos forman una "S" o un círculo, PCA fallará. Solución: **Kernel PCA** o **t-SNE/UMAP**.
2. **Outliers**: Los autovalores son extremadamente sensibles a valores extremos, ya que la varianza se calcula al cuadrado.

#matematicas #algebra-lineal #pca #eigenvalues #dimensionality-reduction #sklearn
