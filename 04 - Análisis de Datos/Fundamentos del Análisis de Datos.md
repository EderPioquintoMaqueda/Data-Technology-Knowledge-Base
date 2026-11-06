# 🔍 Fundamentos del Análisis de Datos: El Framework de Valor

El análisis de datos profesional no consiste en ejecutar comandos de Python, sino en aplicar un **sistema de pensamiento crítico** para convertir datos brutos en decisiones estratégicas. Para un perfil senior, el análisis es un proceso de reducción de incertidumbre.

## 1. El Framework de Análisis Profesional (The Value Chain)

Un análisis mediocre comienza con el código; un análisis profesional comienza con la **pregunta de negocio**.

### El Flujo de Trabajo Estructurado:
1. **Definición del Problema (Business Question)**: Traducir un problema de negocio a un problema de datos. 
   - *Mal*: "Quiero ver las ventas".
   - *Pro*: "Necesito identificar el segmento de clientes con la mayor tasa de deserción (*Churn*) en el último trimestre para diseñar una campaña de retención".
2. **Extracción y Curaduría (Data Sourcing)**: No solo extraer, sino validar la procedencia.
   - ¿El dato es confiable? ¿Tiene sesgos? ¿Cuál es la granularidad (diaria, mensual)?
3. **Limpieza y Transformación (Data Wrangling)**: Aplicar el rigor técnico visto en [[02 - Programación]].
4. **Análisis Exploratorio (EDA)**: Usar estadística para validar hipótesis.
5. **Síntesis y Comunicación (Insight Delivery)**: Traducir el "p-valor" o la "correlación" en una recomendación accionable para la gerencia.

---

## 2. Gestión de Calidad de Datos (Data Quality Engineering)

El principio fundamental es **"Garbage In, Garbage Out" (GIGO)**. Si los datos están sucios, la IA más avanzada dará resultados erróneos.

### Estrategias Profesionales de Limpieza:

#### A. Tratamiento de Valores Faltantes (Imputación)
No siempre se deben eliminar los nulos. La elección del método depende de la distribución:
- **Media ($\mu$)**: Solo si la distribución es Normal y no hay outliers.
- **Mediana**: El estándar profesional. Es robusta frente a valores extremos.
- **Moda**: Para variables categóricas.
- **Imputación Predictiva (KNN / MICE)**: Usar otros datos para predecir el valor faltante. Es el nivel más avanzado.

#### B. Control de Outliers (Valores Atípicos)
Un outlier puede ser un error de sensor o la señal de un fraude bancario.
- **Método IQR (Rango Intercuartílico)**: Ideal para distribuciones no normales.
  - $\text{Límite Superior} = Q3 + 1.5 \times (Q3 - Q1)$.
- **Z-Score**: Ideal para distribuciones Normales.
  - $\text{Outlier si } |Z| > 3$.

#### C. Normalización y Estandarización
Fundamental antes de alimentar cualquier modelo de ML (conectando con [[01 - Matemáticas|Álgebra Lineal]]).
- **Min-Max Scaling**: Mapea al rango $[0, 1]$.
- **StandardScaler**: Centra la media en 0 y la desviación en 1.

---

## 3. Matriz de Decisión de Limpieza

| Tipo de Error | Acción Recomendada | Herramienta / Técnica |
| :--- | :--- | :--- |
| Nulos en variable crítica | Imputación Predictiva o Eliminación | `IterativeImputer` (Scikit-Learn) |
| Outlier extremo (error) | Winsorization (Limitar al percentil 99) | `np.clip()` |
| Formato inconsistente | Normalización de Strings | `.str.strip().str.upper()` |
| Datos duplicados | Deduplicación basada en PK | `df.drop_duplicates(subset=['id'])` |

#data-analysis #data-quality #engineering #pipeline #insights
