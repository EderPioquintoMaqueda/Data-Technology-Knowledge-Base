# 📈 Análisis Exploratorio de Datos (EDA): Rigor Estadístico y Patrones

El EDA no es "hacer gráficos", es la fase de **validación de hipótesis**. Un analista senior utiliza el EDA para decidir qué modelo de Machine Learning es viable y para detectar sesgos que podrían arruinar la predicción.

## 1. Análisis Univariado: La Forma del Dato

Antes de buscar relaciones, debemos entender la naturaleza de cada variable.

### A. Análisis de Distribución y Sesgo (Skewness)
La forma de la distribución determina qué métricas usar:
- **Simétrica (Normal)**: $\text{Media} \approx \text{Mediana}$. Podemos usar la Media y Desviación Estándar.
- **Sesgada a la Derecha (Positive Skew)**: La cola está a la derecha. La Media $>$ Mediana. **Acción**: Usar la Mediana o aplicar una transformación logarítmica ($\log(x)$) para normalizar el dato.
- **Sesgada a la Izquierda (Negative Skew)**: La cola está a la izquierda. La Media $<$ Mediana.

### B. Curtosis (Kurtosis)
Mide el "picado" de la distribución. Una curtosis alta indica que la mayoría de los datos están muy concentrados, pero hay outliers extremos (colas pesadas). Esto es crítico en el **Riesgo Bancario** (Eventos de "Cisne Negro").

---

## 2. Análisis Bivariado y Multivariado: Relaciones y Dependencias

### A. Correlación vs. Causalidad
El error más grave en datos es asumir que la correlación implica causalidad.
- **Pearson ($r$)**: Mide la relación **lineal**. Sensible a outliers.
- **Spearman ($\rho$)**: Mide la relación **monotónica** (si uno sube, el otro también, aunque no sea una línea recta). Es robusto a outliers.

### B. Matriz de Correlación y Multicolinealidad
En modelos de IA, si dos variables están muy correlacionadas ($r > 0.9$), ocurre la **Multicolinealidad**.
- **Impacto**: El modelo se vuelve inestable y los pesos ($\mathbf{w}$) pierden sentido físico.
- **Solución**: Eliminar una de las variables o usar **PCA** (conectando con [[01 - Matemáticas|Álgebra Lineal]]).

---

## 3. Implementación Profesional de EDA en Python

Un pipeline de EDA senior sigue este flujo:

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats

def professional_eda(df, target_col):
    # 1. Análisis de Distribución del Target
    plt.figure(figsize=(10, 4))
    sns.histplot(df[target_col], kde=True)
    plt.title(f"Distribución de {target_col}")
    plt.show()
    
    # 2. Detección de Outliers mediante Z-Score
    z_scores = np.abs(stats.zscore(df.select_dtypes(include=np.number)))
    outliers = (z_scores > 3).sum()
    print(f"Outliers detectados por columna:\n{outliers}")
    
    # 3. Matriz de Correlación Filtrada
    corr = df.corr(numeric_only=True)
    # Solo mostramos correlaciones fuertes (> 0.5 o < -0.5)
    strong_corr = corr[(corr > 0.5) | (corr < -0.5)]
    sns.heatmap(strong_corr, annot=True, cmap='RdBu', center=0)
    plt.show()

# Uso
# professional_eda(df, 'ventas_mensuales')
```

---

## 🎯 Reto de Análisis Técnico
**Escenario**: Tienes un dataset de salarios. La media es \$5,000 pero la mediana es \$2,500.
1. ¿Qué tipo de sesgo tiene la distribución? (Respuesta: Sesgo Positivo / Derecha).
2. ¿Qué significa esto en el mundo real? (Respuesta: Hay unos pocos salarios extremadamente altos que inflan la media).
3. ¿Qué métrica de tendencia central es la más representativa aquí? (Respuesta: La Mediana).

#eda #statistics #data-science #python #outliers #correlation
