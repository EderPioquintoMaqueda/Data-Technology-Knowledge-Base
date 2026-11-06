# 🧩 Aprendizaje No Supervisado: Descubriendo Estructuras Latentes

El aprendizaje no supervisado es la herramienta para encontrar "la verdad oculta" en los datos cuando no tenemos etiquetas. En el sector profesional, se utiliza para la **Segmentación Estratégica** y la **Detección de Anomalías**.

## 1. Clustering Avanzado: Más allá de K-Means

K-Means es el inicio, pero tiene una debilidad: asume que los grupos son esféricos y de tamaño similar.

### A. DBSCAN (Density-Based Spatial Clustering)
En lugar de centros, busca zonas de "alta densidad".
- **Ventaja**: Puede encontrar clusters de cualquier forma (no solo círculos) y detecta automáticamente los **Outliers** como ruido.
- **Caso de Uso**: Detección de fraudes en transacciones bancarias donde el fraude no es un grupo compacto, sino una anomalía aislada.

### B. Clustering Jerárquico
Crea un árbol de grupos (Dendrograma).
- **Uso Pro**: Útil cuando no sabemos cuántos grupos existen ($K$ desconocido) y queremos ver la jerarquía de los datos.

---

## 2. Reducción de Dimensionalidad y Compresión de Información

El objetivo no es solo "hacer el gráfico más pequeño", sino eliminar la redundancia.

### A. PCA (Principal Component Analysis) - El Rigor Técnico
Como vimos en [[01 - Matemáticas|Álgebra Lineal]], el PCA proyecta los datos sobre los autovectores de la matriz de covarianza.
- **Análisis de Varianza**: Un profesional mira el `explained_variance_ratio_`. Si los primeros 3 componentes explican el 95% de la varianza, podemos eliminar las otras 97 variables sin perder información relevante.

### B. t-SNE y UMAP (Visualización No Lineal)
Para datos extremadamente complejos (como imágenes o texto), el PCA falla.
- **t-SNE / UMAP**: Conservan la estructura local de los datos. Son la herramienta estándar para visualizar los **Embeddings** de una IA.

---

## 3. Implementación Profesional: Segmentación de Clientes

```python
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler
import pandas as pd

# 1. Preprocesamiento (Súper Importante en Clustering)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(df[['ingresos', 'gastos', 'edad']])

# 2. Aplicar DBSCAN (Epsilon = distancia max, Min_samples = densidad min)
# No necesitamos decirle cuántos grupos quiere, él los encuentra.
dbscan = DBSCAN(eps=0.5, min_//samples=5)
clusters = dbscan.fit_predict(X_scaled)

# -1 indica que el punto es ruido (outlier)
df['segmento'] = clusters
print(f"Grupos detectados: {len(set(clusters)) - (1 if -1 in clusters else 0)}")
```

---

## 🎯 Reto de Análisis No Supervisado
**Escenario**: Tienes un dataset de sensores de una fábrica. Ves que el PCA reduce 100 variables a 2 componentes, pero el 90% de la varianza se mantiene.
1. ¿Qué significa esto para el negocio? (Respuesta: Que la mayoría de los sensores están midiendo lo mismo; hay una redundancia masiva).
2. ¿Cuál es el riesgo de usar K-Means si los grupos tienen formas alargadas y no circulares? (Respuesta: K-Means fallará al asignar puntos al centroide más cercano, cortando los grupos a la mitad).

#machine-learning #clustering #dbscan #pca #unsupervised-learning
