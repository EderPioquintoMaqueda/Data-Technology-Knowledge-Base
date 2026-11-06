# 📈 Análisis Exploratorio de Datos (EDA): Perfilado Estadístico y Patrones Complejos | Advanced EDA: Statistical Profiling & Complex Patterns

**ES:** Para un analista junior, el EDA es "hacer gráficas para ver qué pasa". Para un experto, el EDA es una **investigación forense**. El objetivo es extraer la estructura latente de los datos y validar supuestos matemáticos antes de que un modelo de ML tome decisiones basadas en ruido.

**EN:** For a junior analyst, EDA is "making plots to see what happens". For an expert, EDA is a **forensic investigation**. The goal is to extract the latent structure of the data and validate mathematical assumptions before an ML model makes decisions based on noise.

---

## 1. Univariate Analysis: The Geometry of a Single Variable | Análisis Univariado: La Geometría de una Variable

No basta con saber que hay un sesgo; hay que entender la **implicación matemática** de ese sesgo.

### A. Skewness (Sesgo) y Transformaciones de Datos
El sesgo afecta la capacidad de los modelos lineales para converger.
- **Positive Skew (Sesgo a la Derecha):** $\text{Media} > \text{Mediana}$. Común en ingresos o precios de viviendas.
    - **Solución Experta:** Aplicar $\log(x)$, $\sqrt{x}$ o la **Transformación Box-Cox**. Esto "comprime" la cola derecha y hace que la distribución se acerque a una Normal, mejorando la performance de la regresión.
- **Negative Skew (Sesgo a la Izquierda):** $\text{Media} < \text{Mediana}$.

### B. Kurtosis y el Riesgo de "Colas Pesadas" (Fat Tails)
La curtosis mide la probabilidad de eventos extremos.
- **Leptocúrtica (Kurtosis Alta):** Distribución muy picuda con colas largas. Indica que los outliers son frecuentes y significativos.
- **Aplicación en Riesgo:** En banca, una curtosis alta en la cartera de créditos significa que hay una probabilidad mayor de "default masivo" (evento de Cisne Negro) que lo que predeciría una distribución Normal.

---

## 2. Bivariate & Multivariate Analysis: Uncovering Latent Relations | Análisis Bivariado y Multivariado: Descubriendo Relaciones Latentes

### A. Más allá de Pearson: Correlaciones No Lineales
La correlación de Pearson solo mide líneas rectas. Un experto usa:
- **Spearman's Rank Correlation:** Mide relaciones **monotónicas** (si uno sube, el otro sube, aunque no sea lineal). Es la herramienta correcta para datos con outliers.
- **Kendall's Tau:** Ideal para datasets muy pequeños o variables ordinales.

### B. El Problema de la Multicolinealidad y el VIF
Cuando dos variables están muy correlacionadas, el modelo de ML sufre de **Inestabilidad de Pesos**.
- **VIF (Variance Inflation Factor):** Un experto calcula el VIF para cada variable.
    - $\text{VIF} > 5 \text{ o } 10 \implies$ Multicolinealidad severa.
    - **Acción:** Eliminar la variable redundante o combinar ambas mediante PCA.

### C. Dimensionality Reduction for EDA (PCA)
Cuando tienes 50 variables, no puedes hacer 50 histogramas.
- **PCA (Principal Component Analysis):** Proyectamos las 50 dimensiones en las 2 o 3 direcciones de máxima varianza.
- **Uso en EDA:** Visualizar si los datos forman "clusters" naturales antes de aplicar cualquier algoritmo de clustering.

---

## 3. Advanced Outlier Detection: ML-Based Methods | Detección Avanzada de Outliers: Métodos basados en ML

El Z-Score y el IQR fallan cuando los outliers no son extremos en una sola dimensión, sino que son "extraños" en la combinación de dimensiones.

### A. Isolation Forest (Bosques de Aislamiento)
En lugar de medir la distancia al centro, este algoritmo intenta "aislar" cada punto.
- **Lógica:** Los outliers son más fáciles de aislar (requieren menos divisiones del espacio) que los puntos normales.
- **Ventaja:** Detecta outliers multivariados que el Z-Score ignoraría.

### B. Local Outlier Factor (LOF)
Mide la densidad local de un punto comparada con la de sus vecinos.
- **Lógica:** Si un punto tiene una densidad mucho menor que sus vecinos, es un outlier local.

---

## 💻 Professional EDA Pipeline (The Expert's Code)

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats
from sklearn.decomposition import PCA
from sklearn.ensemble import IsolationForest

def expert_eda_pipeline(df, target_col):
    # 1. Análisis de Distribución y Transformación sugerida
    skew = df[target_col].skew()
    print(f"Skewness: {skew:.2f}")
    if abs(skew) > 0.75:
        print("Sugerencia: Aplicar transformación Logarítmica para normalizar.")

    # 2. Detección de Outliers Multivariados (Isolation Forest)
    iso = IsolationForest(contamination=0.05, random_state=42)
    numeric_cols = df.select_dtypes(include=np.number)
    outliers = iso.fit_predict(numeric_cols)
    df['is_outlier'] = outliers # -1 es outlier, 1 es normal
    print(f"Outliers detectados: {len(df[df['is_outlier'] == -1])}")

    # 3. Reducción de Dimensionalidad para Visualización (PCA)
    pca = PCA(n_components=2)
    components = pca.fit_transform(numeric_cols.fillna(0))
    
    plt.figure(figsize=(10, 6))
    sns.scatterplot(x=components[:,0], y=components[:,1], hue=df[target_col])
    plt.title("Proyección PCA de los datos (Estructura Latente)")
    plt.show()

# Uso: expert_eda_pipeline(df, 'target')
```

---

## 🎯 Reto de Análisis Forense
**Escenario:** Tienes una correlación de Pearson de 0.1 entre el "Gasto en Publicidad" y las "Ventas", pero la correlación de Spearman es de 0.8.
1. **¿Qué significa esto?** (Respuesta: La relación no es una línea recta, pero es fuertemente monotónica. Hay un crecimiento, pero probablemente saturado o exponencial).
2. **¿Qué gráfico usarías para confirmar?** (Respuesta: Scatter plot con una línea de suavizado LOWESS).
3. **¿Qué pasaría si usas una Regresión Lineal simple aquí?** (Respuesta: El modelo tendría un error alto porque intentaría ajustar una línea recta a una curva).

#eda #statistics #pca #isolation-forest #data-science #python-pro #multivariate-analysis
