---
area: programming
nivel: fundamentos
estado: estudiando
---
# Bucles y Repeticiones

## ¿Qué es?
Un bucle (o loop) es una estructura de control que permite ejecutar un bloque de código repetidamente mientras se cumpla una condición determinada. En lugar de escribir la misma línea de código 1,000 veces, escribes un bucle que lo haga por ti.

## ¿Por qué es importante?
El análisis de datos consiste en procesar colecciones de información. Los bucles son la herramienta fundamental para la **automatización**.
- **Procesamiento masivo**: Recorrer cada fila de un archivo CSV para limpiar el texto.
- **Búsqueda**: Revisar una lista de transacciones hasta encontrar la primera que sea fraudulenta.
- **Iteración de modelos**: Entrenar un modelo de Machine Learning repitiendo el proceso de aprendizaje miles de veces hasta que el error sea mínimo.

## Fundamentos
En Python, existen dos tipos principales de bucles:

1. **Bucle `for`**: Se utiliza cuando sabemos de antemano cuántas veces queremos repetir el código o queremos recorrer todos los elementos de una colección (una lista, un diccionario o una matriz).
2. **Bucle `while`**: Se utiliza cuando no sabemos cuántas veces se repetirá el proceso, sino que depende de que una condición siga siendo verdadera.

## ¿Cómo funciona?
### El Bucle `for` (El más usado en datos)
Recorre una secuencia elemento por elemento.
```python
precios = [100, 250, 300, 150]
for p in precios:
    print(f"El precio con IVA es: {p * 1.16}")
```

### El Bucle `while` (Control basado en condición)
Se ejecuta hasta que la condición cambie a `False`.
```python
contador = 0
while contador < 5:
    print("Procesando lote...")
    contador += 1
```

## Ejemplo conceptual
Imagina que tienes una caja con 100 facturas físicas y debes sumar el total:
- **Acción**: Tomas una factura $\rightarrow$ Sumas el valor al total $\rightarrow$ Dejas la factura a un lado.
- **Bucle**: Repites este proceso **PARA CADA** factura en la caja hasta que la caja esté vacía.

## Aplicación práctica
- **Limpieza de Datos**: Recorrer una columna de nombres para eliminar espacios en blanco al inicio y al final de cada celda.
- **Web Scraping**: Recorrer una lista de URLs para extraer el precio de un producto en cada página.
- **Sistemas de Alerta**: Un bucle `while` que monitorea un sensor de presión cada segundo y dispara una alarma si el valor excede un límite.

## Herramientas relacionadas
- **Python**: El uso de `range()` para generar secuencias numéricas y `enumerate()` para obtener el índice y el valor al mismo tiempo.
- **Pandas**: Aunque Pandas usa bucles internamente, nos ofrece la función `.apply()`, que es una forma optimizada y más rápida de aplicar un bucle sobre una columna entera.

## Conceptos relacionados
- [[Variables y Tipos de Datos]]
- [[Operadores y Condicionales]]
- [[Vectores]]

## Próximos pasos
Ahora que sabemos repetir acciones, aprenderemos a empaquetar esa lógica en bloques reutilizables en [[Funciones]].
