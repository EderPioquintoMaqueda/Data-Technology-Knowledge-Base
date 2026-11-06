# 🗄️ Fundamentos de Datos: Arquitectura y Modelado Profesional

En el entorno profesional, la gestión de datos no es simplemente "guardar información", es el diseño de un **activo estratégico**. Un error en el modelado inicial puede degradar el rendimiento de una IA o hacer que un reporte financiero sea incorrecto.

## 1. Taxonomía Avanzada de Datos

Más allá de la división básica, un Ingeniero de Datos clasifica la información según su ciclo de vida y estructura:

### A. Datos Estructurados (Tabulares)
Siguen un esquema rígido. Son el dominio del **Álgebra Relacional**.
- **Uso Pro**: Almacenados en RDBMS (PostgreSQL, Oracle). Optimizados para transacciones rápidas (ACID).

### B. Datos Semi-estructurados (Flexible Schema)
Datos que llevan su propia descripción (self-describing).
- **Formatos**: JSON, Parquet, Avro.
- **Uso Pro**: Parquet es el estándar en Big Data (Spark/Hadoop) porque es un formato **columnar**, lo que permite leer solo las columnas necesarias, reduciendo la E/S de disco en un 90%.

### C. Datos No Estructurados (Unstructured)
El "océano" de datos.
- **Uso Pro**: Se procesan mediante **Embeddings** (transformando imágenes/textos en vectores) para poder ser analizados matemáticamente en el espacio vectorial (conectando con [[01 - Matemáticas|Álgebra Lineal]]).

---

## 2. El Modelo Relacional y la Normalización

El objetivo de un modelo profesional es eliminar la **redundancia** y evitar las **anomalías de actualización**.

### Las Formas Normales (NF) - Resumen Ejecutivo:
1. **1NF (Primera Forma Normal)**: Cada celda debe contener un solo valor atómico (no listas dentro de celdas).
2. **2NF (Segunda Forma Normal)**: Debe estar en 1NF y todas las columnas no-clave deben depender de la PK completa.
3. **3NF (Tercera Forma Normal)**: Debe estar en 2NF y no debe haber dependencias transitivas (una columna no puede depender de otra columna que no sea la PK).

**Análisis Profesional**: En Data Science, a veces aplicamos la **Desnormalización** intencional. Creamos tablas redundantes ("Tablas Hecho" o "Flat Tables") para acelerar las consultas de lectura en dashboards de Power BI, sacrificando espacio en disco por velocidad de respuesta.

---

## 3. Integridad y Restricciones (Constraints)

Un modelo profesional es aquel que "se protege a sí mismo" mediante restricciones:
- **Entity Integrity**: Asegurada por la `PRIMARY KEY`.
- **Referential Integrity**: Asegurada por la `FOREIGN KEY` con reglas de `ON DELETE CASCADE` o `SET NULL`.
- **Domain Integrity**: Asegurada por `CHECK constraints` (Ej: `CHECK (edad >= 18)`).

---

## 🔗 Conexiones con el Sector de IA
- **Data Lake $\to$ Data Warehouse**: Los datos no estructurados entran al Lake (S3/Azure Blob), se limpian y se transforman en datos estructurados en el Warehouse (Snowflake/BigQuery) para ser consumidos por modelos de ML.
- **Feature Store**: Un repositorio de variables precalculadas (estratégicamente modeladas) que los modelos de IA consumen para evitar recalcular la misma variable millones de veces.

#data-architecture #relational-model #normalization #big-data #data-engineering
