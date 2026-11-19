# 🌌 AL - Espacios Vectoriales: La Geometría de los Datos

En Ciencia de Datos, no vemos los datos como simples filas, sino como puntos en un espacio multidimensional. Si tenemos 10 variables para un cliente, ese cliente es un punto en un espacio de 10 dimensiones.

## 1. Combinaciones Lineales
Una combinación lineal es la creación de un nuevo vector sumando otros vectores multiplicados por escalares.
- **Fórmula**: $\mathbf{v} = c_1\mathbf{v}_1 + c_2\mathbf{v}_2 + \dots + c_n\mathbf{v}_n$
- **Intuición**: Es como mezclar ingredientes. En finanzas, un "Portafolio de Inversión" es una combinación lineal de diferentes activos.

## 2. Independencia Lineal
Un conjunto de vectores es **linealmente independiente** si ninguno de ellos puede escribirse como una combinación lineal de los demás. 

- **Si son dependientes**: Significa que hay información redundante.
- **Si son independientes**: Cada vector aporta una dimensión de información nueva y única.

### 🏦 Caso Bancario: La Trampa de la Redundancia (Multicolinealidad)
Imagina que en tu dataset de crédito tienes estas tres variables:
1. $\mathbf{v}_1$: Ingresos Mensuales.
2. $\mathbf{v}_2$: Ingresos Anuales.
3. $\mathbf{v}_3$: Edad del Cliente.

Aquí $\mathbf{v}_2$ es simplemente $12 \times \mathbf{v}_1$. Son **linealmente dependientes**. 
**Problema**: Si incluyes ambas en un modelo de regresión lineal, el modelo puede volverse inestable (matriz singular), haciendo que las predicciones de riesgo sean erráticas. ¡Debes eliminar la redundancia!

## 3. Base y Dimensión
- **Base**: El conjunto mínimo de vectores linealmente independientes que pueden representar cualquier punto del espacio.
- **Dimensión**: El número de vectores que hay en la base.
- **En Data Science**: La dimensión es el número de *features* (columnas) que realmente aportan valor al modelo.

#matematicas #algebra-lineal #banking #data-cleaning
