# 📊 Visualización Interactiva: Plotly y Bokeh

Para un analista senior, un gráfico estático es solo el inicio. La verdadera potencia surge cuando el usuario puede interactuar con los datos: hacer zoom, filtrar categorías en tiempo real y explorar valores exactos mediante hovers.

## 1. Plotly: El Estándar de la Interactividad

Plotly es la librería líder para crear visualizaciones que se sienten como aplicaciones web. Se basa en la librería JavaScript `Plotly.js`.

### Ventajas Técnicas:
- **Interactividad Nativa**: Zoom, pan y hover integrados sin escribir JS.
- **Plotly Express**: Una API de alto nivel que permite crear gráficos complejos en una sola línea de código.
- **Dash**: Permite convertir tus gráficos de Plotly en dashboards web completos y desplegables.

#### 💻 Implementación: Gráfico de Dispersión Interactivo
```python
import plotly.express as px

# Visualización de correlación con hover detallado
fig = px.scatter(df, x="ingresos", y="gasto", 
                 color="categoria", 
                 hover_data=["id_cliente", "region"],
                 title="Análisis de Consumo por Segmento")

fig.update_layout(template="plotly_dark") # Estética profesional
fig.show()
```

---

## 2. Bokeh: Control Total y Grandes Datasets

Mientras que Plotly es excelente para reportes rápidos, **Bokeh** está diseñado para flujos de datos masivos y personalizaciones profundas.

### Diferencia Clave:
- **Plotly**: Crea el gráfico y lo entrega.
- **Bokeh**: Crea una conexión entre Python y un servidor Bokeh, permitiendo actualizaciones de datos en tiempo real sin recargar la página.

**Uso Pro**: Crear monitores de sensores industriales que se actualizan cada segundo.

---

## 3. Guía de Selección: ¿Cuándo usar qué?

| Objetivo | Herramienta | Razón |
| :--- | :--- | :--- |
| **EDA Rápido** | Seaborn / Matplotlib | Velocidad de prototipado. |
| **Reporte Ejecutivo** | Power BI / Tableau | Gobernanza y distribución masiva. |
| **Exploración Interactiva** | Plotly | Facilidad de interactividad y calidad visual. |
| **Monitorización Real-time** | Bokeh / Grafana | Optimización de flujo de datos continuo. |

#dataviz #plotly #bokeh #interactive-data #python-viz
