# 🔬 SciPy y Statsmodels: El Rigor de la Estadística Profesional

Saber programar un modelo de ML no es suficiente; un profesional debe saber si el resultado es **estadísticamente significativo**. Aquí es donde entran `SciPy` y `Statsmodels`.

## 1. SciPy: El Arsenal del Cálculo Numérico

SciPy es la librería base para cualquier operación matemática avanzada que no sea álgebra lineal pura (NumPy).

### Módulos Críticos:
- **`scipy.stats`**: Distribuciones de probabilidad y tests estadísticos.
- **`scipy.optimize`**: Minimización de funciones y búsqueda de raíces (la base del entrenamiento de modelos).
- **`scipy.interpolate`**: Creación de curvas suaves a partir de puntos dispersos.
- **`scipy.signal`**: Procesamiento de señales (filtros, transformadas de Fourier), vital para sensores de maquinaria.

---

## 2. Statsmodels: Modelado Estadístico y Econometría

Mientras que Scikit-Learn se enfoca en la **predicción**, Statsmodels se enfoca en la **inferencia** (entender la relación entre variables).

### El Análisis de Regresión Profesional
En lugar de solo obtener la predicción, `statsmodels` te entrega un **Resumen Estadístico** completo:

```python
import statsmodels.api as sm

# Añadir la constante (intercepto) que Scikit-Learn hace automático
X = sm.add_constant(X)
model = sm.OLS(y, X).fit()

print(model.summary()) # Aquí está el oro: p-valores, R-squared, t-stats
```

### Cómo leer el Summary de un Profesional:
1. **R-squared**: ¿Qué porcentaje de la varianza de $y$ es explicada por el modelo? (Cercano a 1 es mejor).
2. **P-value ($\text{P} > |t|$)**: Si es $< 0.05$, la variable es estadísticamente significativa. Si es $> 0.05$, la variable es "ruido" y debería ser eliminada.
3. **Prob (F-statistic)**: ¿El modelo en su conjunto es mejor que simplemente usar la media?

---

## 3. Test de Hipótesis en la Vida Real

Un analista senior no dice "creo que esto subió", dice "la diferencia es estadísticamente significativa con un p-valor de 0.01".

### Casos de Uso:
- **T-Test**: Comparar si la media de dos grupos es diferente (Ej: ¿El nuevo sensor es más preciso que el viejo?).
- **ANOVA**: Comparar la media de 3 o más grupos.
- **Test de Normalidad (Shapiro-Wilk)**: Verificar si los datos siguen una distribución normal antes de elegir entre un test paramétrico o no paramétrico.

```python
from scipy import stats

# Ejemplo: T-test para comparar dos grupos de rendimiento
grupo_a = [10, 12, 11, 13, 12]
grupo_b = [15, 14, 17, 13, 16]

t_stat, p_val = stats.ttest_ind(grupo_a, grupo_b)
if p_val < 0.05:
    print("Hay una diferencia significativa")
```

#statistics #scipy #statsmodels #data-science #hypothesis-testing
