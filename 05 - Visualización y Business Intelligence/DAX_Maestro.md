# 📊 DAX Maestro: El Arte de la Manipulación de Contextos | DAX Master: The Art of Context Manipulation

**ES:** DAX (Data Analysis Expressions) no es una extensión de Excel; es un lenguaje funcional diseñado para operar sobre modelos dimensionales. El secreto para dominar DAX no es aprender funciones, sino entender los **Contextos**. Quien domina la transición de contexto, domina la inteligencia de negocios.

**EN:** DAX (Data Analysis Expressions) is not an Excel extension; it is a functional language designed to operate on dimensional models. The secret to mastering DAX is not learning functions, but understanding **Contexts**. Those who master context transition master business intelligence.

---

## 1. The Core Architecture: Filter vs Row Context | Arquitectura Base: Contexto de Filtro vs Contexto de Fila

En DAX, el valor de una medida no es estático; es el resultado de la interacción entre dos contextos.

### A. Row Context (Contexto de Fila) $\rightarrow$ "El Iterador"
Ocurre cuando DAX mira una sola fila a la vez.
- **Dónde sucede:** En las **Columnas Calculadas** y en las funciones iteradoras (**SUMX, AVERAGEX, FILTER**).
- **Limitación:** El contexto de fila **NO filtra** la tabla. Si tienes una columna calculada `[Precio] * [Cantidad]`, DAX solo ve la fila actual, no sabe quién es el cliente o la fecha.

### B. Filter Context (Contexto de Filtro) $\rightarrow$ "El Embudo"
Es el conjunto de filtros aplicados al modelo en un momento dado.
- **Dónde sucede:** En las **Medidas**.
- **Cómo se define:** Por los segmentadores (Slicers), los filtros de página o las filas/columnas de una matriz.
- **Lógica:** El motor de DAX filtra las tablas de dimensiones $\rightarrow$ el filtro viaja por la relación $\rightarrow$ se filtran las tablas de hechos $\rightarrow$ se calcula el resultado.

---

## 2. Context Transition: The Magic of CALCULATE | Transición de Contexto: La Magia de CALCULATE

La **Transición de Contexto** es el proceso de convertir un Contexto de Fila en un Contexto de Filtro. Esto ocurre automáticamente cuando usamos la función `CALCULATE()`.

### ¿Por qué es vital?
Si estás dentro de un `SUMX` (Contexto de Fila) y quieres calcular el porcentaje de ventas de ese producto respecto al total de la categoría, necesitas "romper" la fila actual y mirar la categoría completa.

**La fórmula maestra:**
$$\text{Medida} = \text{CALCULATE}( \text{Expresión}, \text{Filtros} )$$

- **Uso Profesional:** `CALCULATE([Total Sales], ALL('Product'))` $\rightarrow$ Elimina todos los filtros de la tabla producto para calcular el Gran Total, permitiendo crear ratios de porcentaje.

---

## 3. Advanced Patterns: Virtual Tables & Intelligence | Patrones Avanzados: Tablas Virtuales e Inteligencia

### A. Virtual Tables (The Power User Approach)
A veces necesitas crear una tabla que no existe en el modelo para hacer un cálculo complejo. Usamos funciones que devuelven tablas:
- **`SUMMARIZE`:** Agrupa datos sobre la marcha.
- **`ADDCOLUMNS`:** Añade cálculos a una tabla virtual.
- **`FILTER`:** Crea una sub-tabla basada en una condición.

**Ejemplo Experto:** Calcular la venta promedio solo de los días que tuvieron ventas superiores a \$1,000.
`AVERAGEX( FILTER( 'Sales', 'Sales'[Amount] > 1000 ), 'Sales'[Amount] )`

### B. Time Intelligence (The Business Heartbeat)
Para análisis de crecimiento, el uso de una tabla `Calendar` es obligatorio.
- **YTD (Year To Date):** `TOTALYTD([Sales], 'Calendar'[Date])`
- **SPLY (Same Period Last Year):** `CALCULATE([Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))`
- **Growth %:** `DIVIDE([Sales] - [Sales SPLY], [Sales SPLY])`

---

## 4. Performance Tuning: VertiPaq Optimization | Optimización de Performance

Un experto no solo escribe DAX que funcione, sino DAX que sea rápido.

### Estrategias de Optimización:
1. **Medidas sobre Columnas:** Nunca crees columnas calculadas si puedes usar una medida. Las columnas consumen RAM fija; las medidas se calculan al vuelo.
2. **Evitar `FILTER(ALL(Table), ...)`:** Es más eficiente usar `KEEPFILTERS` o filtrar columnas específicas en lugar de tablas completas.
3. **Uso de Variables (`VAR`):** Las variables almacenan el resultado de un cálculo una sola vez, evitando que DAX recalcule la misma expresión varias veces en la misma medida.

```dax
// ✅ PATRÓN PROFESIONAL CON VARIABLES
Sales Growth = 
VAR CurrentSales = [Total Sales]
VAR PastSales = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))
RETURN
    DIVIDE(CurrentS_Sales - PastSales, PastSales, 0)
```

---

## 🎯 Reto de Lógica DAX
**Escenario:** Tienes un dashboard con un segmentador de "Categoría de Producto". Quieres crear una medida que muestre las ventas de la categoría seleccionada, pero que al lado muestre la suma de **todas las demás categorías** (excluyendo la seleccionada).
1. **¿Qué función usarías para romper el filtro de la categoría seleccionada?** (Respuesta: `ALL` o `ALLEXCEPT`).
2. **¿Cómo filtrarías para excluir la selección actual?** (Respuesta: Usando `FILTER` con una condición de desigualdad $\neq$ o usando `KEEPFILTERS` con un filtro invertido).

#powerbi #dax #context-transition #bi-optimization #data-modeling #enterprise-bi
