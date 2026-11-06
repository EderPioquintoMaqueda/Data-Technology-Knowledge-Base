# 📈 Business Intelligence y Power BI: Arquitectura de Datos y DAX Avanzado

El Business Intelligence (BI) profesional no es crear reportes, es construir un **Sistema de Información Estratégica**. Power BI es la herramienta, pero la inteligencia reside en el **Modelado de Datos** y la **Lógica de Negocio** implementada en DAX.

## 1. Arquitectura de Datos: El Star Schema (Esquema en Estrella)

Para que un dashboard sea escalable y rápido, debemos alejarnos de las tablas planas (estilo Excel) y adoptar el **Star Schema**.

### Componentes del Modelo:
- **Fact Tables (Tablas de Hechos)**: Contienen los eventos cuantitativos (Ventas, Lecturas de Sensor, Transacciones). Son tablas masivas, con millones de filas, diseñadas para ser agregadas.
- **Dimension Tables (Tablas de Dimensiones)**: Contienen la metadata descriptiva (Clientes, Productos, Calendario, Ubicación). Son tablas pequeñas, con valores únicos.
- **Relaciones**: Siempre se busca una relación de **1:N (Uno a Muchos)** desde la Dimensión hacia el Hecho.

**Análisis Profesional**: Un modelo en estrella minimiza la redundancia y optimiza el motor de almacenamiento en memoria (VertiPaq) de Power BI, permitiendo que el filtrado sea instantáneo.

---

## 2. Dominio de DAX (Data Analysis Expressions)

DAX no es Excel; es un lenguaje basado en el **Contexto de Filtro**.

### A. El Concepto de Contexto de Filtro (Filter Context)
Es la característica más potente y difícil de DAX. El valor de una medida cambia según los filtros aplicados en el dashboard (segmentadores, filas de una tabla, etc.).
- **CALCULATE**: Es la función más importante. Permite modificar el contexto de filtro actual para realizar cálculos complejos.

### B. Patrones DAX Profesionales

#### 1. Inteligencia de Tiempo (Time Intelligence)
Fundamental para analizar el crecimiento y la estacionalidad.
- **SPLY (Same Period Last Year)**:
  ```dax
  Sales LY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))
  ```
- **YTD (Year To Date)**:
  ```dax
  Sales YTD = TOTALYTD([Total Sales], 'Calendar'[Date])
  ```

#### 2. Ratio de Eficiencia y Porcentajes
Usar la función `DIVIDE` en lugar de `/` para evitar errores de división por cero.
```dax
Churn Rate = 
DIVIDE(
    [Lost Customers], 
    [Total Customers], 
    0
)
```

#### 3. Análisis de Pareto (80/20)
Cálculo de la suma acumulada para identificar los clientes que generan el 80% de los ingresos.
```dax
Cumulative Sales = 
CALCULATE(
    [Total Sales], 
    FILTER(
        ALLSELECTED('Customer'), 
        'Customer'[Sales] >= EARLIER('Customer'[Sales])
    )
)
```

---

## 3. Diseño de Dashboards Ejecutivos: Del Dato a la Decisión

Un dashboard senior no es una "colección de gráficos", es una **herramienta de diagnóstico**.

### El Flujo de Navegación:
1. **Visión Global (The "What")**: KPIs principales en la parte superior. (Ej: "Ventas totales bajaron 5%").
2. **Análisis de Tendencias (The "When")**: Gráficos de líneas temporales. (Ej: "La caída empezó en marzo").
3. **Análisis de Atributos (The "Who/Where")**: Desgloses por categoría o región. (Ej: " la caída es solo en la Región Norte").
4. **Detalle Granular (The "Exactly")**: Tabla de detalles con drill-through. (Ej: "Estos son los 10 clientes que dejaron de comprar").

---

## 🎯 Reto de Modelado de Datos
**Escenario**: Tienes una tabla de `Ventas` (10M filas) y una de `Productos` (500 filas).
1. ¿Cómo deben relacionarse? (Respuesta: Relación 1:N desde Productos a Ventas).
2. ¿Dónde crearías el cálculo del "Margen de Ganancia"? (Respuesta: Como una **Medida DAX**, no como columna calculada, para optimizar la memoria RAM).
3. ¿Por qué usarías una tabla `Calendario` separada en lugar de usar la fecha de la tabla de Ventas? (Respuesta: Para permitir análisis de inteligencia de tiempo consistentes y manejar huecos de fechas donde no hubo ventas).

#powerbi #dax #star-schema #bi-architecture #storytelling #data-modeling
