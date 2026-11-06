# ⚙️ Funciones: Diseño de Software para Data Pipelines

En el desarrollo profesional de software para datos, una función no es solo un "bloque de código", es una unidad de transformación de datos. El objetivo es crear funciones que sean **atómicas**, **testeables** y **eficientes**.

## 1. Diseño de Funciones Profesionales

### A. Pureza y Determinismo
Una función es **pura** si no depende de estados externos y no los modifica.
- **Importancia**: En el entrenamiento de modelos de IA, las funciones de transformación de datos deben ser puras para garantizar que el preprocesamiento sea idéntico en el entrenamiento y en la inferencia (producción).

### B. Single Responsibility Principle (SRP)
Cada función debe hacer **una sola cosa**. 
- **Mal**: `def procesar_datos():` (Limpia, filtra, calcula y guarda).
- **Pro**: `def limpiar_nulos():`, `def calcular_kpi():`, `def guardar_resultado():`.

---

## 2. Herramientas Avanzadas de Python para Datos

### A. Funciones Lambda y el método `.apply()` de Pandas
Para transformaciones rápidas en columnas de datos, usamos funciones anónimas ($\lambda$).

```python
import pandas as pd

df = pd.DataFrame({'precio': [100, 200, 300]})

# Aplicar un impuesto del 15% a toda la columna de forma vectorizada
df['precio_final'] = df['precio'].apply(lambda x: x * 1.15)
```

### B. Decoradores (@decorator)
Los decoradores permiten añadir funcionalidad a una función sin modificar su código interno. Son extremadamente útiles para el **Logging** y la **Medición de Performance**.

```python
import time

def medir_tiempo(func):
    def wrapper(*args, **kwargs):
        inicio = time.time()
        resultado = func(*args, **kwargs)
        print(f"La función {func.__name__} tardó {time.time()-inicio:.4f}s")
        return resultado
    return wrapper

@medir_tiempo
def proceso_pesado():
    # Simulación de carga de datos
    time.sleep(1)

proceso_pesado()
```

---

## 3. Gestión de Errores en Producción (Try-Except)

En un pipeline de datos, un solo dato corrupto puede detener un proceso de 10 horas. Un profesional no ignora los errores, los gestiona.

```python
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return 0.0 # Valor neutro para no romper el cálculo
    except TypeError:
        print("Error: Uno de los valores no es numérico")
        return None
```

---

## 🎯 Reto de Arquitectura
**Escenario**: Tienes que crear un pipeline que:
1. Cargue un CSV.
2. Limpie los nombres de las columnas.
3. Calcule el promedio de una columna.
4. Guarde el resultado en un archivo.

**Enfoque Novato**: Un script largo con código secuencial.
**Enfoque Profesional**: 
- Una función `load_data(path)` $\to$ retorna DataFrame.
- Una función `clean_columns(df)` $\to$ retorna DataFrame limpio.
- Una función `calc_avg(df, col)` $\to$ retorna un float.
- Una función `save_result(val)` $\to$ escribe en disco.
- Un **Orquestador** (función principal) que llama a cada una en orden.

**Ventaja**: Si la limpieza de columnas cambia, solo modificas una función. Si quieres cambiar el CSV por una base de datos, solo modificas `load_data`.

#programming #software-engineering #python-pro #pandas #clean-code
