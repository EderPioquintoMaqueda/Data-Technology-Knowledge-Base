# 📏 Métricas y KPIs: Estrategia de Medición y Valor de Negocio

En la industria, la capacidad de calcular un número es trivial; la capacidad de definir la **métrica correcta** es lo que define a un analista senior. Un KPI no es solo una fórmula, es la representación numérica de un objetivo estratégico.

## 1. La Jerarquía de la Medición

Para evitar el ruido en la toma de decisiones, clasificamos las medidas en tres niveles:

### A. Métricas de Vanidad (Vanity Metrics)
Números que se ven bien en una presentación pero no impulsan ninguna acción.
- **Ejemplos**: "Número total de usuarios registrados", "Número de descargas de la app".
- **Riesgo**: Pueden dar una falsa sensación de éxito mientras el negocio colapsa.

### B. Métricas Operativas (Performance Metrics)
Miden la eficiencia de un proceso específico.
- **Ejemplos**: "Tiempo promedio de respuesta del servidor", "Tasa de error en la carga de datos".
- **Uso**: Son diagnósticas; nos dicen *dónde* está el problema, pero no necesariamente *cómo* afecta al negocio.

### C. KPIs Estratégicos (North Star Metrics)
Métricas que están directamente ligadas al valor entregado al cliente y al ingreso de la empresa.
- **Ejemplo**: "LTV (Lifetime Value) del Cliente" o "Tasa de Churn Mensual".
- **Propiedad**: Si el KPI sube, el negocio crece. Si baja, la estrategia debe cambiar.

---

## 2. Ingeniería de KPIs: El Método SMART y la Acción

Un KPI profesional debe ser **Accionable**. Si el KPI cambia y tú no sabes qué palanca mover para corregirlo, entonces no es un KPI, es solo una métrica.

### El Proceso de Definición:
1. **Objetivo**: "Aumentar la rentabilidad de la cartera de créditos".
2. **Métrica Base**: "Tasa de Default".
3. **KPI SMART**: "Reducir la tasa de default de créditos hipotecarios del 5% al 3% en los próximos 6 meses mediante el ajuste del scoring de riesgo".

---

## 3. Implementación Técnica y Automatización

Un KPI profesional no se calcula en un Excel manual; se automatiza en un pipeline de datos.

### El Flujo de Cálculo:
$\text{SQL (Extracción)} \to \text{Python (Lógica de Negocio)} \to \text{BI (Visualización)}$.

#### Ejemplo: Cálculo de Churn Rate (Tasa de Abandono)
$$\text{Churn Rate} = \frac{\text{Clientes perdidos en el periodo}}{\text{Total de clientes al inicio del periodo}} \times 100$$

```python
import pandas as pd

def calculate_churn(df_start, df_end):
    """
    Calcula la tasa de churn comparando dos snapshots de clientes.
    df_start: Clientes activos al inicio del mes.
    df_end: Clientes activos al final del mes.
    """
    start_count = len(df_start)
    # Clientes que estaban al inicio pero ya no están al final
    lost_customers = df_start[~df_start['id'].isin(df_end['id'])]
    churn_rate = (len(lost_customers) / start_count) * 100
    
    return churn_rate

# Uso profesional: Automatizado mediante una tarea programada (Airflow)
# churn = calculate_churn(df_january, df_february)
```

---

## 🎯 Reto de Análisis Estratégico
**Escenario**: Tu jefe te dice: "El número de visitas a la web ha subido un 200%, ¡estamos teniendo un éxito increíble!".
1. ¿Es esta una Métrica de Vanidad o un KPI? (Respuesta: Métrica de Vanidad).
2. ¿Qué KPI propondrías para validar si ese tráfico realmente es un éxito? (Respuesta: Tasa de Conversión $\text{Visitas} \to \text{Ventas}$).
3. ¿Por qué la segunda es superior? (Respuesta: Porque mide el valor real y la eficiencia del embudo, no solo la curiosidad del usuario).

#metrics #kpi #business-intelligence #strategy #data-driven
