# 📏 Métricas y KPIs: Estrategia de Medición, Valor de Negocio y Análisis de Cohortes | Strategic Measurement & Cohort Analysis

**ES:** En la industria, la capacidad de calcular un número es trivial; la capacidad de definir la **métrica correcta** es lo que define a un analista senior. Un KPI no es solo una fórmula, es la representación numérica de un objetivo estratégico que debe ser **accionable**.

**EN:** In the industry, the ability to calculate a number is trivial; the ability to define the **correct metric** is what defines a senior analyst. A KPI is not just a formula, it is the numerical representation of a strategic objective that must be **actionable**.

---

## 1. The Measurement Hierarchy | La Jerarquía de la Medición

Para evitar el ruido y el "engaño" de los datos, clasificamos las medidas en tres niveles técnicos:

### A. Vanity Metrics (Métricas de Vanidad)
Números que crecen linealmente y hacen sentir bien al equipo, pero no indican salud del negocio.
- **Ejemplos:** "Total de usuarios registrados", "Visitas totales a la web", "Número de seguidores".
- **El peligro:** Puedes tener 1 millón de usuarios registrados, pero si solo el 1% usa la app, el negocio está muriendo.

### B. Performance Metrics (Métricas Operativas)
Miden la eficiencia de un proceso técnico o humano. Son diagnósticas.
- **Ejemplos:** "Tasa de error de la API", "Tiempo de carga de la página", "Costo de Adquisición (CAC)".
- **Uso:** Nos dicen *dónde* está el problema, pero no necesariamente el impacto en la rentabilidad.

### C. North Star Metrics (KPIs Estratégicos)
La métrica única que refleja el valor real entregado al cliente.
- **Ejemplos:** "Active Users per Day (DAU)" para Facebook, "Noches reservadas" para Airbnb.
- **Propiedad:** Si la North Star sube, el valor percibido por el cliente aumenta y el ingreso sigue orgánicamente.

---

## 2. Advanced Analysis: Cohort Analysis | Análisis de Cohortes

El análisis de promedios es el enemigo del análisis experto. El promedio oculta la realidad. El **Análisis de Cohortes** es la herramienta para revelar la verdad.

### ¿Qué es una Cohorte?
Una cohorte es un grupo de usuarios que comparten una característica en un tiempo determinado (ej. "Usuarios que se registraron en Enero").

### El análisis de Retención (Retention Matrix)
En lugar de decir "tenemos un 20% de retención", un experto analiza la **Matriz de Retención**:
- **Eje Y:** Mes de registro (Cohorte).
- **Eje X:** Meses transcurridos desde el registro.
- **Valor:** % de usuarios activos.

**Análisis Forense de Cohortes:**
- Si la retención de la cohorte de "Marzo" es mayor que la de "Enero", el experto pregunta: *¿Qué cambiamos en el producto en Marzo que hizo que los usuarios se quedaran más?*

---

## 3. Predictive Value Metrics: LTV and CAC | Métricas de Valor Predictivo

Un negocio es sostenible solo si el valor del cliente es mayor que el costo de adquirirlo.

### A. LTV (Customer Lifetime Value)
El valor total que un cliente aporta durante toda su relación con la empresa.
$$\text{LTV} = \text{Valor promedio de compra} \times \text{Frecuencia de compra} \times \text{Vida útil del cliente}$$

### B. La Regla de Oro de la Sostenibilidad (LTV/CAC Ratio)
Un analista senior vigila la relación entre LTV y CAC:
- **LTV/CAC < 1:** El negocio pierde dinero por cada cliente.
- **LTV/CAC $\approx 1$:** El negocio sobrevive pero no crece.
- **LTV/CAC $\approx 3$:** El "Sweet Spot" de crecimiento saludable.
- **LTV/CAC $> 5$:** Estás gastando muy poco en adquisición; podrías crecer mucho más rápido.

---

## 💻 Professional Implementation: Cohort Analysis in Python

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

def calculate_cohort_matrix(df):
    # df debe tener: ['user_id', 'order_date']
    
    # 1. Convertir fecha a primer mes de actividad (Cohorte)
    df['order_month'] = df['order_date'].dt.to_period('M')
    df['cohort'] = df.groupby('user_id')['order_month'].transform('min')
    
    # 2. Calcular el índice del mes (Mes 0, Mes 1, etc.)
    df['cohort_index'] = (df['order_month'] - df['cohort']).apply(lambda x: x.n)
    
    # 3. Agrupar por cohorte y mes índice
    cohort_data = df.groupby(['cohort', 'cohort_index'])['user_id'].nunique().reset_index()
    
    # 4. Pivotar la tabla
    cohort_pivot = cohort_data.pivot(index='cohort', columns='cohort_index', values='user_id')
    
    # 5. Convertir a porcentaje de retención
    cohort_sizes = cohort_pivot.iloc[:, 0]
    retention = cohort_pivot.divide(cohort_sizes, axis=0)
    
    return retention

# Visualización Profesional
plt.figure(figsize=(12, 8))
sns.heatmap(retention, annot=True, fmt='.0%', cmap='YlGnBu')
plt.title("Matriz de Retención de Usuarios (Cohort Analysis)")
plt.ylabel("Cohorte de Registro")
plt.xlabel("Meses desde el Registro")
plt.show()
```

---

## 🎯 Reto de Análisis Estratégico
**Escenario:** Tu North Star Metric (Usuarios Activos) está subiendo, pero el LTV está bajando.
1. **¿Qué está pasando?** (Respuesta: Estás adquiriendo muchos usuarios nuevos, pero son de "baja calidad" o no están encontrando valor a largo plazo).
2. **¿Qué métrica mirarías para confirmar?** (Respuesta: La tasa de retención de las cohortes más recientes vs las antiguas).
3. **¿Cuál sería la recomendación al CEO?** (Respuesta: Dejar de optimizar el embudo de adquisición y empezar a optimizar el *Onboarding* y la experiencia de usuario para aumentar la retención).

#metrics #kpi #cohort-analysis #ltv #cac #business-strategy #data-driven
