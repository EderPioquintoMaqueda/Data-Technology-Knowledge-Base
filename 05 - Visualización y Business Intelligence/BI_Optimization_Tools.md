# 🚀 Optimización de Power BI: Tabular Editor y DAX Studio

Cuando un reporte de Power BI se vuelve lento, el problema no es la cantidad de datos, sino la eficiencia de las medidas DAX y la arquitectura del modelo. Los profesionales utilizan herramientas externas para "abrir el capó" del motor VertiPaq.

## 1. Tabular Editor: El IDE de la Metadata

Tabular Editor no es una herramienta de visualización, es un editor de la estructura del modelo. Permite hacer cambios que en Power BI Desktop tomarían horas.

### Capacidades Profesionales:
- **Scripting en C#**: Puedes crear scripts para generar 100 medidas similares automáticamente.
- **Calculation Groups**: Permite crear una sola medida que cambie su comportamiento (ej. cambiar entre "Ventas Actuales", "Ventas Año Pasado" y "Crecimiento %") mediante un solo segmentador.
- **BPA (Best Practice Analyzer)**: Analiza tu modelo y te dice exactamente qué reglas de diseño estás rompiendo (ej. "Tienes columnas calculadas que deberían ser medidas").

---

## 2. DAX Studio: El Microscopio de la Performance

DAX Studio es la herramienta fundamental para optimizar el tiempo de respuesta de los reportes.

### Análisis de Performance:
1. **Server Timings**: Permite ver cuánto tiempo pasa la consulta en la **Fórmula Engine** (CPU) y cuánto en el **Storage Engine** (Disco/RAM).
2. **Optimización de FórmCulas**: Si una medida es lenta, DAX Studio te muestra el plan de ejecución.
   - **Señales de Alerta**: Si ves demasiadas "CallbackDataID", significa que DAX está abandonando el motor rápido y volviendo al lento, lo que destruye la performance.

---

## 3. Estrategia de Optimización de Reportes

Para reducir el tiempo de carga de un dashboard de 10 segundos a 1 segundo:

| Problema | Solución Técnica | Herramienta |
| :--- | :--- | :--- |
| Medidas DAX lentas | Sustituir `FILTER` por `KEEPFILTERS` o optimizar la granularidad. | DAX Studio |
| Modelo muy pesado | Eliminar columnas de alta cardinalidad (IDs únicos innecesarios). | Tabular Editor |
| Carga de datos lenta | Implementar **Incremental Refresh** (solo cargar datos nuevos). | Power BI Desktop |
| Cálculos redundantes | Mover la lógica de DAX hacia el SQL (Push-down). | SQL / Power Query |

#powerbi #dax-studio #tabular-editor #performance-tuning #bi-engineering
