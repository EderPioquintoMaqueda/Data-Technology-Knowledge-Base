# 🔍 Fundamentos del Análisis de Datos: Gobernanza, Arquitectura y Valor | Data Analysis Fundamentals: Governance, Architecture & Value

**ES:** Un analista junior limpia datos; un analista senior diseña la **estrategia de calidad y el flujo de la información**. El análisis profesional no es un acto aislado, es una tubería (*pipeline*) donde la confianza en el dato es el activo más valioso.

**EN:** A junior analyst cleans data; a senior analyst designs the **quality strategy and information flow**. Professional analysis is not an isolated act; it is a pipeline where data trust is the most valuable asset.

---

## 1. The Medallion Architecture | Arquitectura de Medallones (Bronze, Silver, Gold)

Para evitar el caos en la gestión de datos, la industria utiliza el estándar de la **Arquitectura Medallón**. Esto asegura que el dato sea trazable y auditable.

### 🥉 Bronze Layer (Raw Data)
- **Estado:** Datos brutos, tal como vienen de la fuente (APIs, CSVs, DBs).
- **Regla:** **Nunca se modifica**. Se guarda el historial completo. Si cometemos un error en el análisis, podemos volver aquí y procesar todo de nuevo.

### 🥈 Silver Layer (Cleansed & Conformed)
- **Estado:** Datos limpios, tipados y validados.
- **Proceso:** Se eliminan duplicados, se estandarizan formatos (ej. fechas a ISO 8601) y se resuelven nulos.
- **Objetivo:** Es la "fuente de verdad" para los analistas.

### 🥇 Gold Layer (Business Ready / Aggregated)
- **Estado:** Datos agregados y optimizados para el negocio.
- **Proceso:** Cálculo de KPIs, joins finales y tablas resumen.
- **Objetivo:** Alimentar los dashboards de BI y los modelos de ML.

---

## 2. Data Lineage & Governance | Linaje y Gobernanza de Datos

**Data Lineage** es la capacidad de rastrear un dato desde el KPI final hasta la fuente original.

### ¿Por qué es crítico el Linaje?
Si un CEO ve que la "Tasa de Churn" subió un 10%, la primera pregunta es: **"¿Este dato es correcto?"**. Sin linaje, el analista responde "creo que sí". Con linaje, el analista muestra:
`Fuente SQL $\rightarrow$ Transformación Python (Filtro X) $\rightarrow$ Agregación Gold $\rightarrow$ Dashboard`.

### Data Profiling Profesional
Antes de limpiar, un experto hace un **Perfilado de Datos**. No mira los datos, mira las **metadatos de la calidad**:
- **Completeness:** ¿Qué % de la columna es nulo?
- **Uniqueness:** ¿Cuántos valores únicos hay respecto al total?
- **Consistency:** ¿El formato es siempre el mismo (ej. todos los teléfonos tienen 10 dígitos)?
- **Timeliness:** ¿Cuándo fue la última actualización del dato?

---

## 3. Expert Data Cleaning: Beyond the Basics | Limpieza de Datos Avanzada

Ya conocemos la media y la mediana. A nivel experto, aplicamos técnicas de **estabilidad numérica**.

### A. Winsorization (Limitación de Outliers)
En lugar de borrar los outliers (lo que puede sesgar la muestra), la **Winsorización** consiste en limitar los valores extremos a un percentil específico (ej. el percentil 5 y 95).
- **Uso:** Ideal en finanzas para evitar que una sola transacción millonaria arruine el promedio de un segmento.

### B. Imputación Iterativa (MICE)
Cuando los datos faltantes no son aleatorios, la media es un error. Usamos **MICE (Multivariate Imputation by Chained Equations)**.
- **Lógica:** El algoritmo modela cada variable con nulos como una función de las otras variables. Es una "mini-IA" para rellenar huecos.

### C. Feature Scaling: Standard vs Robust
- **StandardScaler:** $\frac{x - \mu}{\sigma}$. Malo si hay outliers extremos.
- **RobustScaler:** $\frac{x - \text{mediana}}{\text{IQR}}$. El estándar profesional cuando los datos están "sucios", ya que la mediana y el IQR no se dejan engañar por los valores extremos.

---

## 🛠️ Professional Workflow: The Quality Checklist

Antes de entregar cualquier análisis, un experto pasa por este checklist de calidad:

1. [ ] **Source Validation:** ¿La fuente de datos es la oficial o una copia desactualizada?
2. [ ] **Grain Check:** ¿La granularidad es la correcta? (Ej: ¿Estoy sumando ventas diarias cuando el KPI pide promedio mensual?).
3. [ ] **Outlier Impact:** ¿Si quito los 5 valores más altos, el resultado cambia drásticamente?
4. [ ] **Logic Test:** ¿Hay valores imposibles? (Ej: Edad = -1, Salario = 0 en un cargo senior).
5. [ ] **Lineage Doc:** ¿Tengo documentado el paso a paso de la transformación?

#data-analysis #data-governance #medallion-architecture #data-quality #insights #engineering
