# 🚀 SQL Avanzado: El Arte de la Extracción Eficiente

Para un Ingeniero de Datos, escribir una query que "funcione" es el mínimo. El objetivo profesional es escribir queries que sean **SARGables** (Search ARGumentable), es decir, que permitan al motor de la base de datos utilizar los índices de la manera más eficiente posible.

## 1. Window Functions: Análisis sin Colapso de Filas

A diferencia de `GROUP BY`, que reduce el número de filas, las Window Functions permiten realizar cálculos agregados manteniendo el detalle de cada registro.

### Sintaxis Maestra: `FUNCTION() OVER (PARTITION BY ... ORDER BY ...)`

#### A. Ranking y Numeración
- `ROW_NUMBER()`: Asigna un número único secuencial.
- `RANK()`: Asigna el mismo rango a valores idénticos, saltando el siguiente.
- `DENSE_RANK()`: Asigna el mismo rango pero sin saltos.

#### B. Navegación de Datos (Lead & Lag)
Permiten acceder a datos de la fila siguiente o anterior sin hacer un JOIN con la misma tabla.
- `LAG(col, n)`: Trae el valor de la fila $n$ posiciones atrás.
- `LEAD(col, n)`: Trae el valor de la fila $n$ posiciones adelante.

**Caso de Uso Profesional**: Calcular la tasa de crecimiento mensual comparando el valor actual con el del mes anterior.
```sql
SELECT 
    mes, 
    ventas, 
    LAG(ventas) OVER (ORDER BY mes) as ventas_mes_anterior,
    (ventas - LAG(ventas) OVER (ORDER BY mes)) / LAG(ventas) OVER (ORDER BY mes) as crecimiento
FROM ventas_mensuales;
```

---

## 2. CTEs (Common Table Expressions) y Recursividad

Las CTEs (`WITH`) transforman la lógica anidada en una secuencia lineal y legible.

### CTEs Recursivas: El Poder de la Jerarquía
Permiten procesar estructuras de árbol o grafos (como organigramas o rutas de logística).

**Ejemplo: Recorrer la jerarquía de mando de un banco**
```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Ancla: El CEO
    SELECT id, name, manager_id, 1 as level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    -- Parte Recursiva: Empleados que reportan al nivel anterior
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level;
```

---

## 3. Tuning y Performance: El Camino al Millonésimo Registro

Un profesional nunca lanza una query masiva sin antes revisar el **Plan de Ejecución**.

### Comando Maestro: `EXPLAIN ANALYZE`
Este comando ejecuta la query y desglosa el costo en milisegundos y el método de acceso.

#### Patrones de Performance:
1. **Seq Scan $\rightarrow$ Index Scan**: Si ves un `Seq Scan` en una tabla de millones de filas, necesitas un índice.
2. **Nested Loop $\rightarrow$ Hash Join**: Para tablas grandes, el `Hash Join` es la meta. Si ves muchos `Nested Loops`, revisa si faltan índices en las claves foráneas.

#### Reglas de Oro de SARGability:
- ❌ **MAL**: `WHERE YEAR(fecha) = 2023` $\rightarrow$ El motor debe calcular la función para cada fila, anulando el índice.
- ✅ **BIEN**: `WHERE fecha >= '2023-01-01' AND fecha < '2024-01-01'` $\rightarrow$ El motor va directo al rango del índice.

#sql-pro #performance-tuning #window-functions #postgres #data-engineering
