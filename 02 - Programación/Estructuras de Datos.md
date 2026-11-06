# 🛠️ Estructuras de Datos: Optimización y Complejidad Computacional

En el nivel profesional, no elegimos una estructura de datos porque "sea más fácil", sino basándonos en la **Complejidad Temporal (Big O Notation)** y el uso de memoria. Para trabajar en IA y Big Data, es imperativo entender cómo escala el código cuando pasamos de 1,000 a 1,000,000 de registros.

## 1. Análisis de Complejidad en Estructuras de Python

| Estructura | Acceso (Índice) | Búsqueda (Valor) | Inserción | Eliminación |
| :--- | :--- | :--- | :--- | :--- |
| **Lista `[]`** | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Diccionario `{}`** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Set `{}`** | N/A | $O(1)$ | $O(1)$ | $O(1)$ |
| **Tupla `()`** | $O(1)$ | $O(n)$ | N/A (Inmutable) | N/A |

### 🧠 Análisis Profundo: El costo de la Lista
Cuando haces `if "cliente" in lista_de_clientes:`, Python debe recorrer la lista uno por uno hasta encontrar el valor. Si la lista tiene 1 millón de elementos, en el peor caso hará 1 millón de comparaciones ($O(n)$). 
Si conviertes esa lista a un **Set** o un **Diccionario**, la búsqueda es instantánea ($O(1)$) gracias a la función de *Hashing*.

---

## 2. Estructuras Especializadas para Ciencia de Datos

Las estructuras nativas de Python no son suficientes para el procesamiento masivo. Aquí entran las estructuras de **NumPy** y **Pandas**.

### A. El NumPy ndarray (Tensors)
A diferencia de la lista, el `ndarray` almacena los datos en un bloque contiguo de memoria y obliga a que todos sean del mismo tipo (homogéneos). 
- **Ventaja**: Permite el uso de **Strides** (saltos de memoria) para acceder a datos multidimensionales sin copiar la matriz.

### B. El Pandas DataFrame
Es una estructura híbrida: una colección de Series (vectores) que comparten un índice común.
- **Uso Profesional**: El DataFrame es la representación de una tabla de base de datos en memoria. Su potencia reside en la **operación vectorizada** (aplicar una función a toda la columna sin bucles).

---

## 3. Patrones de Diseño con Estructuras de Datos

### Patrón 1: Mapeo de Categorías (Lookup Tables)
En lugar de usar múltiples `if/elif` para categorizar datos, usa un diccionario.
```python
# ❌ Mal: Difícil de mantener
if tipo == 'A': score = 10
elif tipo == 'B': score = 20...

# ✅ Pro: Escalable y rápido O(1)
mapeo_scores = {'A': 10, 'B': 20, 'C': 30}
score = mapeo_scores.get(tipo, 0) # Valor por defecto 0
```

### Patrón 2: Eliminación de Duplicados Masiva
Para limpiar un dataset de IDs únicos:
```python
# ❌ Lento: O(n^2)
unicos = []
for id in ids_totales:
    if id not in unicos:
        unicos.append(id)

# ✅ Pro: O(n)
unicos = list(set(ids_totales))
```

---

## 🔗 Conexiones con la IA y los Datos
- **Tensores**: Un Tensor en PyTorch es esencialmente un `ndarray` de NumPy con la capacidad de vivir en una GPU y rastrear sus propios gradientes.
- **Bases de Datos**: Un índice en SQL (como el B-Tree) busca imitar la eficiencia $O(\log n)$ o $O(1)$ que tenemos con los diccionarios en memoria.
- **Embeddings**: Los embeddings de palabras son vectores almacenados en matrices; la búsqueda de similitud entre ellos se hace mediante el producto punto (Álgebra Lineal).

#programming #data-structures #big-o #optimization #numpy #pandas
