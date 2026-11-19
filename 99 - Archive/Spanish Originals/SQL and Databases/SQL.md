# 🐚 SQL Profesional: Optimización, Analítica y Performance

SQL no es solo un lenguaje de consulta; es la herramienta de manipulación de conjuntos más potente de la industria. Para un perfil profesional, el objetivo no es "hacer que la query funcione", sino "hacer que la query sea eficiente".

## 1. SQL Analítico Avanzado (The Power User)

Para análisis de datos complejos, el SQL básico es insuficiente. Dominamos estas tres herramientas:

### A. CTEs (Common Table Expressions)
Usando la cláusula `WITH`, creamos tablas temporales legibles.
- **Ventaja**: Transforman una query "monstruo" en una serie de pasos lógicos.
- **Uso Pro**: Ideales para cálculos recursivos o para limpiar datos antes de un JOIN final.

### B. Window Functions (Funciones de Ventana)
Permiten hacer cálculos sobre un grupo de filas sin colapsar el resultado en una sola fila (como hace `GROUP BY`).
- **Operadores**: `RANK()`, `DENSE_RANK()`, `ROW_NUMBER()`, `LEAD()`, `LAG()`.
- **Caso de Uso**: Calcular el "crecimiento mensual de ventas" comparando la fila actual con la anterior (`LAG`).

### C. Subconsultas Correlacionadas
Consultas que dependen de la query exterior. Útiles para filtros complejos, aunque deben usarse con cuidado por su impacto en la performance.

---

## 2. Optimización y Performance (Tuning)

En producción, una query mal escrita puede bloquear una base de datos entera.

### El Plan de Ejecución (`EXPLAIN ANALYZE`)
La herramienta más importante del profesional. Nos dice cómo la base de datos planea ejecutar la query:
- **Seq Scan (Sequential Scan)**: La base de datos lee toda la tabla. **Malo** en tablas grandes.
- **Index Scan**: La base de datos usa un índice para ir directo al dato. **Ideal**.
- **Nested Loop vs Hash Join**: Cómo la DB combina tablas. El Hash Join es generalmente más eficiente para grandes volúmenes.

### Estrategias de Optimización:
1. **SARGability**: Evitar funciones sobre las columnas en el `WHERE`. 
   - ❌ `WHERE YEAR(fecha) = 2023` (Anula el índice).
   - ✅ `WHERE fecha >= '2023-01-01' AND fecha < '2024-01-01'` (Usa el índice).
2. **Selective SELECT**: Nunca uses `SELECT *`. Trae solo las columnas necesarias para reducir la carga de I/O y memoria.
3. **JOIN Order**: Filtrar la tabla más grande primero antes de unirla con las pequeñas.

---

## 3. Implementación Técnica: Ejemplo de Análisis de Retención

Calculamos la retención de clientes mes a mes usando Window Functions:

```sql
WITH monthly_activity AS (
    SELECT 
        customer_id, 
        DATE_TRUNC('month', transaction_date) as active_month,
        ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY transaction_date) as visit_number
    FROM transactions
)
SELECT 
    active_month, 
    COUNT(DISTINCT customer_id) as retained_customers
FROM monthly_activity
WHERE visit_number > 1
GROUP BY active_month
ORDER BY active_month;
```

---

## 🎯 Reto de Optimización
**Escenario**: Tienes una tabla `Transacciones` con 100 millones de filas. La query `SELECT * FROM Transacciones WHERE cliente_id = 500` tarda 10 segundos.
1. ¿Cuál es la causa probable? (Respuesta: Seq Scan por falta de índice).
2. ¿Cómo lo solucionas? (Respuesta: `CREATE INDEX idx_cliente ON Transacciones(cliente_id);`).
3. ¿Cuál es el costo de crear ese índice? (Respuesta: Ralentiza las inserciones/updates ya que el índice debe actualizarse).

#sql-pro #query-optimization #window-functions #postgres #database-tuning
