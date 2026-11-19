# 🛠️ Power Query y Lenguaje M: El Motor de ETL Profesional | Power Query & M Language: Professional ETL Engine

**ES:** Power Query es la puerta de entrada de los datos. El 80% del éxito de un dashboard depende de que los datos lleguen limpios, estructurados y optimizados. Mientras que la interfaz de usuario es intuitiva, el verdadero poder reside en el **Lenguaje M**, un lenguaje funcional que permite transformaciones que serían imposibles con simples clics.

**EN:** Power Query is the gateway for data. 80% of a dashboard's success depends on the data arriving clean, structured, and optimized. While the UI is intuitive, the true power lies in the **M Language**, a functional language that allows transformations that would be impossible with simple clicks.

---

## 1. The M Language Architecture | Arquitectura del Lenguaje M

El lenguaje M es un lenguaje **case-sensitive** y **funcional**. A diferencia de SQL, que es declarativo, M es una serie de pasos secuenciales donde cada paso es una variable que hace referencia al paso anterior.

### Anatomía de un Script M:
```powerquery
let
    Origen = Sql.Database("servidor", "db"),
    Filtrado = Table.SelectRows(Origen, each [Ventas] > 100),
    Transformado = Table.TransformColumnTypes(Filtrado, {{"Fecha", type date}})
in
    Transformado
```
**Análisis Técnico:** El bloque `let` define las variables y el bloque `in` define cuál es la salida final. Este flujo permite hacer "debug" paso a paso en la interfaz de Power Query.

---

## 2. The Holy Grail: Query Folding | El Santo Grial: Query Folding

**Query Folding** es la capacidad de Power Query de traducir sus pasos de transformación a **SQL nativo** y enviarlos directamente al servidor de la base de datos.

### ¿Por qué es la diferencia entre un experto y un novato?
- **Con Folding:** El servidor SQL filtra 100 millones de filas y solo envía 1,000 resultados a Power BI. **Resultado:** Carga en segundos.
- **Sin Folding:** Power BI descarga las 100 millones de filas a la RAM de tu computadora y luego las filtra. **Resultado:** El sistema colapsa o tarda horas.

### Reglas para Mantener el Folding:
1. **Filtros Primero:** Realiza los `Table.SelectRows` y `Table.SelectColumns` inmediatamente después de la extracción.
2. **Evitar Funciones Complejas Temprano:** No uses funciones personalizadas o transformaciones de texto complejas al inicio, ya que esto "rompe" el folding y obliga a Power BI a descargar los datos.
3. **Verificación:** Clic derecho en un paso $\rightarrow$ "View Native Query". Si la opción está deshabilitada, el folding se ha roto en ese paso.

---

## 3. Advanced Transformations: Beyond the UI | Transformaciones Avanzadas

Un profesional de datos domina tres operaciones críticas que no siempre son obvias en la interfaz:

### A. Unpivot (El Normalizador)
Convierte datos "estilo Excel" (columnas para cada mes) en datos "estilo Base de Datos" (una columna de Mes y una de Valor).
- **Uso:** Es obligatorio para que el Star Schema y DAX funcionen correctamente.

### B. Custom Functions (Automatización de ETL)
Puedes convertir una query en una **función** que reciba parámetros.
- **Ejemplo:** Crear una función `fn_LimpiarVentas(archivo)` y aplicarla a una carpeta entera de 100 CSVs mediante `Table.AddColumn`.

### C. Conditional Columns & Logic
Uso de la estructura `if ... then ... else ...` para crear categorías de negocio complejas antes de que el dato llegue al modelo.

---

## 4. Memory & Performance Optimization | Optimización de Memoria y Performance

### Estrategias de Carga:
1. **Import Mode:** Los datos se comprimen en el motor VertiPaq. Ideal para la mayoría de los casos.
2. **Incremental Refresh:** En lugar de recargar 10 años de datos cada día, Power BI solo recarga los datos del último día y mantiene el resto archivado.
3. **Schema Drift:** Manejar cambios en las columnas de la fuente mediante la selección explícita de columnas (`Table.SelectColumns`) para evitar que el reporte se rompa si la fuente cambia.

---

## 🎯 Reto de Ingeniería de ETL
**Escenario:** Tienes que conectar Power BI a una base de datos con 1 billón de filas. Necesitas filtrar por "Año 2023" y "País = México".
1. **¿Qué pasa si haces el filtro en DAX?** (Respuesta: El reporte fallará porque intentará cargar el billón de filas en la RAM).
2. **¿Qué pasa si haces el filtro en Power Query pero rompes el Query Folding?** (Respuesta: Seguirá siendo lento porque la descarga ocurre antes del filtro).
3. **¿Cuál es la solución experta?** (Respuesta: Asegurar que los filtros de Año y País ocurran al inicio de la query en Power Query para que se traduzcan a un `WHERE` de SQL y el servidor solo envíe los datos necesarios).

#powerbi #powerquery #language-m #etl #data-engineering #query-folding #big-data
