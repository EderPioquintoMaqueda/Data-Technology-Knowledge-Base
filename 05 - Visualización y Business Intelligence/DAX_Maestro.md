# 📊 DAX Maestro: Lógica de Negocio y Optimización de Modelos

DAX (Data Analysis Expressions) no es una herramienta de cálculo, es un lenguaje de **manipulación de contextos**. Quien domina el contexto de filtro, domina Power BI.

## 1. El Motor de DAX: Contexto de Filtro vs. Contexto de Fila

### A. Contexto de Fila (Row Context)
Ocurre en las **Columnas Calculadas**. La fórmula se ejecuta fila por fila. Es ineficiente porque consume RAM permanentemente.

### B. Contexto de Filtro (Filter Context)
Ocurre en las **Medidas**. La fórmula se calcula "al vuelo" basándose en los filtros aplicados en el reporte (Slicers, Visuals). Es el estándar profesional.

### C. Transición de Contexto
Cuando usamos la función `CALCULATE`, transformamos un contexto de fila en uno de filtro. Es la operación más potente de DAX.

---

## 2. Patrones de Cálculo Avanzados

### A. Inteligencia de Tiempo (Time Intelligence)
Para análisis de crecimiento, es obligatorio usar una tabla `Calendar` separada.

- **SPLY (Same Period Last Year)**:
  ```dax
  Ventas_SPLY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))
  ```
- **Cálculo de Acumulado (YTD)**:
  ```dax
  Ventas_YTD = TOTALYTD([Total Sales], 'Calendar'[Date])
  ```

### B. Iteradores (Funciones X)
Cuando necesitamos hacer un cálculo fila por fila antes de sumar (ej. `Precio * Cantidad` para cada producto y luego sumar todo).
- **Suma de productos**: `SUMX(Tabla, Tabla[Precio] * Tabla[Cantidad])`.
- **Diferencia**: `SUM` suma una columna; `SUMX` itera la tabla y luego suma.

---

## 3. Arquitectura de Modelado: Star Schema y Performance

### El Modelo en Estrella (Star Schema)
- **Fact Table**: Solo métricas y FKs.
- **Dimension Tables**: Atributos descriptivos.
- **Regla de Oro**: Evitar el "Many-to-Many" y los filtros bidireccionales, ya que degradan la performance y crean ambigüedades en los datos.

### Optimización del Motor VertiPaq
Power BI comprime los datos en memoria. Para optimizarlo:
1. **Reducir la Cardinalidad**: Evitar columnas con valores únicos masivos (ej. IDs de transacción) si no son necesarios.
2. **Evitar Columnas Calculadas**: Mover la lógica al Power Query (Lenguaje M) o al SQL original.

#powerbi #dax #business-intelligence #data-modeling #star-schema
