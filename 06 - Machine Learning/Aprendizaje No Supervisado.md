# 🧩 Aprendizaje No Supervisado: Descubriendo Estructuras Latentes y Anomalías | Unsupervised Learning: Latent Structure & Anomaly Detection

**ES:** El aprendizaje no supervisado es la herramienta para encontrar la "verdad oculta" en los datos cuando no tenemos etiquetas. En el sector profesional, no se trata de "hacer grupos", sino de realizar una **Segmentación Estratégica** y una **Detección de Anomalías** basada en la topología de los datos.

**EN:** Unsupervised learning is the tool for finding the "hidden truth" in data when labels are unavailable. In the professional sector, it's not about "making groups", but about **Strategic Segmentation** and **Anomaly Detection** based on data topology.

---

## 1. Advanced Clustering: Beyond the Spheres | Clustering Avanzado: Más allá de las Esferas

K-Means es el punto de partida, pero tiene una debilidad matemática: asume que los clusters son convexos (esféricos) y que todos tienen un tamaño similar. Un experto elige el algoritmo según la **forma de los datos**.

### A. DBSCAN (Density-Based Spatial Clustering of Applications with Noise)
En lugar de buscar centros, DBSCAN busca zonas de "alta densidad" de puntos.
- **Lógica:** Un punto es parte de un cluster si tiene al menos `min_samples` vecinos dentro de un radio `eps`.
- **Ventajas Críticas:**
    1. **Formas Arbitrarias:** Puede detectar clusters en forma de media luna, anillos o líneas.
    2. **Detección Automática de Ruido:** Los puntos que no pertenecen a ningún cluster denso se marcan como `-1` (Outliers).
- **Uso Pro:** Detección de fraudes bancarios donde el comportamiento normal es denso y el fraude es un punto aislado.

### B. Hierarchical Clustering (HCA)
Crea un árbol de grupos (Dendrograma) uniendo los puntos más cercanos iterativamente.
- **Criterios de Linkage:** La forma de medir la distancia entre clusters (Single, Complete, Ward).
- **Uso Pro:** Análisis de taxonomías y filogenia de datos donde queremos entender la jerarquía de los segmentos.

---

## 2. Dimensionality Reduction: Compressing Information | Reducción de Dimensionalidad: Comprimiendo Información

Reducir dimensiones no es solo "hacer el gráfico más pequeño", es eliminar la redundancia y extraer las **características principales**.

### A. PCA (Principal Component Analysis) - El Rigor Lineal
Como vimos en [[01 - Matemáticas|Álgebra Lineal]], el PCA proyecta los datos sobre los autovectores de la matriz de covarianza.
- **Explained Variance Ratio:** Un profesional no elige el número de componentes al azar. Mira la varianza acumulada. Si los primeros 3 componentes explican el 90% de la varianza, el resto es ruido.
- **Limitación:** El PCA es lineal. Si la estructura de los datos es una espiral, el PCA la aplastará.

### B. t-SNE y UMAP (Manifold Learning)
Para datos complejos (como Embeddings de texto o imágenes), usamos algoritmos no lineales que preservan la estructura local.
- **t-SNE (t-distributed Stochastic Neighbor Embedding):** Excelente para visualización, pero lento y no preserva las distancias globales.
- **UMAP (Uniform Manifold Approximation and Projection):** El estándar actual. Es más rápido que t-SNE y preserva mejor la estructura global y local.
- **Uso Pro:** Visualizar la "nube de palabras" de un LLM para ver cómo se agrupan los conceptos semánticamente.

---

## 3. Anomaly Detection: The Art of the Rare | Detección de Anomalías: El Arte de lo Raro

En el mundo real, el "outlier" es a menudo el dato más valioso (fraude, falla de motor, ataque cibernético).

### A. Isolation Forest (Bosques de Aislamiento)
En lugar de medir la distancia al centro, este algoritmo intenta "aislar" cada punto mediante particiones aleatorias.
- **Lógica:** Los outliers son más fáciles de aislar (están en las periferias), por lo que requieren menos divisiones para quedar solos en una hoja del árbol.
- **Ventaja:** Escala la red la dimensionalidad y es extremadamente rápido.

### B. Local Outlier Factor (LOF)
Compara la densidad local de un punto con la de sus vecinos.
- **Lógica:** Un punto es un outlier si su densidad es significativamente menor que la de sus vecinos, incluso si está cerca de un cluster.

---

## 💻 Professional Implementation: Unsupervised Pipeline

```python
import numpy as np
import pandas as pd
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import IsolationForest
from umap import UMAP # pip install umap-learn

# 1. Preprocesamiento Obligatorio (Scaling)
# El clustering se basa en distancias; sin scaling, la variable con números más grandes domina.
scaler = StandardScaler()
X_scaled = scaler.fit_transform(df_numeric)

# 2. Detección de Anomalías con Isolation Forest
iso = IsolationForest(contamination=0.01, random_state=42)
df['anomaly'] = iso.fit_predict(X_scaled) # -1 = anomalía

# 3. Clustering de Densidad con DBSCAN
dbscan = DBSCAN(eps=0.3, min_samples=10)
df['cluster'] = dbscan.fit_predict(X_scaled)

# 4. Visualización de Alta Dimensión con UMAP
reducer = UMAP(n_neighbors=15, min_dist=0.1, n_components=2, random_state=42)
embedding = reducer.fit_transform(X_scaled)

# Visualizar: scatter plot de 'embedding' coloreado por 'cluster'
```

---

## 🎯 Reto de Análisis No Supervisado
**Escenario:** Tienes embeddings de 1536 dimensiones de comentarios de clientes. Quieres saber si existen grupos de quejas comunes.
1. **¿Por qué no usar K-Means?** (Respuesta: En 1536D, el concepto de "centro" y "distancia euclidiana" pierde sentido debido a la maldición de la dimensionalidad).
2. **¿Cuál sería el flujo experto?** (Respuesta: UMAP para reducir a 2D $\rightarrow$ Visualizar clusters $\rightarrow$ DBSCAN para etiquetar los grupos y detectar ruido).
3. **¿Qué significaría un punto marcado como -1 por DBSCAN?** (Respuesta: Que ese comentario es único y no se parece a ninguna de las quejas comunes; podría ser un problema nuevo y crítico).

#machine-learning #unsupervised #dbscan #umap #pca #anomaly-detection #isolation-forest
