# 🛠️ Power Query y Lenguaje M: El Motor de ETL Profesional

Power Query es la puerta de entrada de los datos. El 80% del éxito de un dashboard depende de que los datos lleguen limpios y estructurados. El lenguaje **M** es el motor detrás de cada paso de transformación.

## 1. El Flujo de ETL Profesional

Un proceso de ETL (Extract, Transform, Load) eficiente sigue este orden:
1. **Extracción**: Conexión a la fuente (SQL, API, Web).
2. **Filtrado Temprano**: Eliminar filas y columnas innecesarias *antes* de hacer Joins.
3. **Transformación**: Limpieza de tipos, normalización y pivoteo.
4. **Carga**: Carga al modelo de datos (preferiblemente como "Import" para velocidad).

---

## 2. Dominio del Lenguaje M

El lenguaje M es un lenguaje funcional. Cada paso en Power Query es una variable que hace referencia a la anterior.

### Funciones Críticas de M:
- **`Table.Pivot` / `Table.Unpivot`**: El Unpivot es la herramienta más potente para normalizar datos "estilo Excel" y convertirlos en formato tabular para BI.
- **`Table.Group`**: Agregaciones previas a la carga para reducir la cantidad de filas en el modelo.
- **`Text.Select` / `Text.Combine`**: Limpieza avanzada de strings.

---

## 3. Optimización de Carga: Query Folding

El **Query Folding** es el "Santo Grial" de Power Query. Es la capacidad de Power Query de traducir sus pasos de transformación a **SQL nativo** y enviarlos al servidor.

### ¿Por qué es vital?
- **Con Folding**: El servidor de SQL filtra los datos y Power BI solo recibe el resultado final (Súper rápido).
- **Sin Folding**: Power BI descarga la tabla completa y filtra los datos en la RAM de tu PC (Lento y costoso).

**Cómo mantener el Folding**:
- Realiza los filtros y Joins al principio del proceso.
- Evita usar funciones complejas de M que no tengan equivalente en SQL al inicio de la query.

#powerbi #powerquery #language-m #etl #data-engineering
