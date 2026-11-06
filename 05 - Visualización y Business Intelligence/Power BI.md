# 📈 Business Intelligence y Power BI: Arquitectura de Datos Empresarial | Enterprise BI Architecture & Power BI

**ES:** Para un usuario básico, Power BI es una herramienta para hacer reportes. Para un arquitecto de datos, Power BI es un motor de compresión columnar basado en el motor **VertiPaq**. La diferencia entre un reporte que tarda 30 segundos en cargar y uno que carga en 1 segundo no es la computadora, sino la **arquitectura del modelo de datos**.

**EN:** For a basic user, Power BI is a tool for making reports. For a data architect, Power BI is a columnar compression engine based on the **VertiPaq** engine. The difference between a report that takes 30 seconds to load and one that loads in 1 second is not the computer, but the **data model architecture**.

---

## 1. The Star Schema: The Gold Standard of BI | El Esquema en Estrella: El Estándar de Oro

El error más común es intentar usar "tablas planas" (estilo Excel) en Power BI. Esto provoca redundancia, errores de cálculo y lentitud extrema. El profesional utiliza el **Star Schema**.

### A. Fact Tables (Tablas de Hechos)
Son el centro de la estrella. Contienen los eventos cuantitativos y las claves foráneas (FK).
- **Características:** Millones de filas, columnas numéricas, alta volatilidad.
- **Ejemplo:** `Fact_Ventas` (ID_Producto, ID_Cliente, ID_Fecha, Monto, Cantidad).

### B. Dimension Tables (Tablas de Dimensiones)
Son las puntas de la estrella. Contienen los atributos descriptivos.
- **Características:** Pocas filas, valores únicos en la PK, baja volatilidad.
- **Ejemplo:** `Dim_Productos` (ID_Producto, Nombre, Categoría, Marca, Color).

### C. The Power of the Relationship (1:N)
La relación debe ser siempre de **1 (Dimensión) a Muchos (Hechos)**.
- **Filtrado Unidireccional:** El filtro fluye desde la Dimensión $\rightarrow$ hacia el Hecho.
- **Súper Prohibido:** Relaciones Muchos-a-Muchos (*Many-to-Many*) y Filtros Bidireccionales (*Both*). Estos crean ambigüedades en los datos y destruyen la performance del motor VertiPaq.

---

## 2. Storage Modes: Managing Big Data | Modos de Almacenamiento: Gestión de Big Data

Un experto elige el modo de almacenamiento basándose en el volumen de datos y la necesidad de frescura (*latency*).

| Modo | Cómo funciona | Ventaja | Desventaja | Caso de Uso |
| :--- | :--- | :--- | :--- | :--- |
| **Import** | Carga los datos en la RAM y los comprime. | Velocidad extrema (máxima performance). | Límite de RAM; requiere refresco programado. | 90% de los casos; dashboards ejecutivos. |
| **DirectQuery** | No carga datos; hace una consulta SQL al servidor en tiempo real. | Datos siempre frescos; maneja Petabytes. | Lento; depende de la velocidad del servidor SQL. | Monitoreo en tiempo real; datasets masivos. |
| **Composite** | Mezcla Import y DirectQuery en el mismo modelo. | Lo mejor de los dos mundos. | Arquitectura compleja de gestionar. | Tablas maestras en Import y transacciones en DirectQuery. |

---

## 3. Row-Level Security (RLS) | Seguridad a Nivel de Fila

En entornos corporativos, no puedes enviar el mismo reporte a todos. El **RLS** permite definir reglas de visibilidad.

- **Dynamic RLS:** Se utiliza la función `USERPRINCIPALNAME()` para capturar el email del usuario logueado.
- **Lógica:** Se crea un filtro en la tabla de Dimensiones: `[Email_Vendedor] = USERPRINCIPALNAME()`.
- **Resultado:** El vendedor de la Zona Norte solo verá los datos de la Zona Norte, aunque el reporte sea el mismo para toda la empresa.

---

## 4. The VertiPaq Engine: Columnar Compression | El Motor VertiPaq: Compresión Columnar

Para optimizar la RAM, Power BI no guarda los datos fila por fila, sino **columna por columna**.

### Técnicas de Compresión Interna:
1. **Value Encoding:** Si una columna tiene pocos valores únicos, Power BI crea un diccionario y guarda solo el índice (Entero) en lugar del string.
2. **Hash Encoding:** Optimiza la búsqueda de valores únicos.
3. **Run-Length Encoding (RLE):** Si hay miles de filas con el mismo valor seguido, Power BI guarda: "Valor X repetido 5000 veces".

**Consejo de Experto:** Para maximizar VertiPaq, **reduce la cardinalidad**. Elimina columnas de IDs únicos (como el ID de transacción) si no son estrictamente necesarias para el análisis; esto reducirá el tamaño del modelo de GBs a MBs.

---

## 🎯 Reto de Arquitectura de BI
**Escenario:** Tienes que diseñar un reporte para una cadena de 500 tiendas. Tienes una tabla de `Ventas` (500M filas) y una de `Tiendas` (500 filas). El CEO quiere ver los datos en tiempo real.
1. **¿Qué modo de almacenamiento usarías?** (Respuesta: DirectQuery, debido al volumen y la necesidad de tiempo real).
2. **¿Cómo estructurarías el modelo?** (Respuesta: Star Schema: `Dim_Tiendas` $\rightarrow$ `Fact_Ventas` con relación 1:N).
3. **¿Cómo asegurarías que cada Gerente de Tienda solo vea sus datos?** (Respuesta: Implementando Dynamic RLS usando el email del gerente).

#powerbi #bi-architecture #star-schema #vertipaq #rls #big-data #enterprise-bi
