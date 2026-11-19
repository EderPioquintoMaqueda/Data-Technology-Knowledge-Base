# 🐘 Postgres Internals: Optimización de Almacenamiento y Concurrencia

Dominar PostgreSQL significa entender qué pasa debajo del motor. No se trata de escribir SQL, sino de configurar el motor para que el SQL vuele.

## 1. Estrategias de Indexación Profesional

No todos los índices son iguales. Elegir el incorrecto puede ralentizar las inserciones y no ayudar en las consultas.

| Tipo de Índice | Funcionamiento Técnico | Caso de Uso Ideal |
| :--- | :--- | :--- |
| **B-Tree** | Árbol balanceado de búsqueda. | Consultas de rango (`>`, `<`) e igualdad. El estándar. |
| **Hash** | Tabla hash de valores únicos. | Únicamente igualdad (`=`). Más rápido que B-Tree para IDs únicos. |
| **GIN (Generalized Inverted Index)** | Índice invertido de elementos. | Búsqueda dentro de arrays, JSONB o texto completo. |
| **BRIN (Block Range Index)** | Almacena min/max por bloque de disco. | Tablas masivas ordenadas por tiempo (Time-series). Ahorra 99% de espacio. |

### Índices Parciales y Cubrientes
- **Índices Parciales**: `CREATE INDEX ... WHERE status = 'ACTIVE'`. Solo indexa los datos relevantes, ahorrando espacio y tiempo.
- **Covering Indexes (`INCLUDE`)**: Permite que la query obtenga la respuesta directamente del índice sin tener que ir a la tabla (Index Only Scan).

---

## 2. Gestión de la Concurrencia y Bloqueos (Locks)

En sistemas bancarios, la consistencia es ley. Postgres usa **MVCC (Multi-Version Concurrency Control)**.

### Cómo funciona MVCC:
Cuando actualizas una fila, Postgres no borra la versión vieja; crea una versión nueva. Esto permite que alguien lea el dato mientras otro lo escribe sin bloquearse mutuamente.

### Niveles de Aislamiento (Isolation Levels):
1. **Read Committed (Default)**: Evita leer datos no confirmados.
2. **Repeatable Read**: Garantiza que si lees un dato dos veces en la misma transacción, sea el mismo.
3. **Serializable**: El nivel más estricto. Simula que las transacciones se ejecutan una tras otra. Evita los "Phantom Reads".

---

## 3. Optimización de Escritura Masiva

Para cargar millones de filas sin colapsar el servidor:
- **`COPY` en lugar de `INSERT`**: El comando `COPY` es la forma más rápida de cargar datos en Postgres.
- **Desactivar Índices**: Borra los índices antes de una carga masiva y recréalos al final.
- **Unlogged Tables**: Para datos temporales que no necesitan recuperación ante fallos, usa `UNLOGGED` para eliminar el costo del registro de transacciones (WAL).

#postgresql-internals #database-tuning #mvcc #indexing #backend-engineering
