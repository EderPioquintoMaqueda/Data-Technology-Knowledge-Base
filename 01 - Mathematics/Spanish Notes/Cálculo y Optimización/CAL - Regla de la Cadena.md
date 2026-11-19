# ⛓️ CAL - Regla de la Cadena: El Motor del Backpropagation

La **Regla de la Cadena** es, probablemente, la regla de derivación más importante para la Inteligencia Artificial. Mientras que las derivadas simples nos dicen cómo cambia una variable, la Regla de la Cadena nos permite calcular la derivada de **funciones compuestas** (funciones dentro de funciones).

## 1. La Intuición: El Efecto Dominó
Imagina una cadena de eventos en un banco:
1. Si la **Tasa de Interés** sube $\rightarrow$ la **Demanda de Créditos** baja.
2. Si la **Demanda de Créditos** baja $\rightarrow$ la **Rentabilidad del Banco** disminuye.

Si queremos saber cómo afecta la **Tasa de Interés** directamente a la **Rentabilidad**, no podemos mirar solo un paso. Tenemos que multiplicar el efecto del primer cambio por el efecto del segundo. **Eso es la Regla de la Cadena**.

## 2. Definición Matemática
Si tenemos una función $y = f(g(x))$, donde $u = g(x)$ es una función interna y $y = f(u)$ es la función externa, la derivada de $y$ respecto a $x$ es:

$$\frac{dy}{dx} = \frac{dy}{du} \cdot \frac{du}{dx}$$

En palabras simples: la derivada del total es la **derivada de la capa exterior** multiplicada por la **derivada de la capa interior**.

---

## 🧠 Aplicación en IA: El Backpropagation

El entrenamiento de una Red Neuronal es básicamente una aplicación masiva de la Regla de la Cadena. Una red neuronal es una composición de miles de funciones:
$$\text{Salida} = \text{Capa}_L(\text{Capa}_{L-1}(\dots \text{Capa}_1(\text{Entrada})\dots))$$

### ¿Cómo funciona el Backpropagation?
1. **Forward Pass**: Los datos pasan por la red y generan una predicción. Se calcula el **Error (Loss)**.
2. **Backward Pass**: Para reducir el error, necesitamos saber cuánto contribuyó cada peso ($w$) de la red al error final.
3. Aplicamos la Regla de la Cadena desde la salida hacia atrás:
   - Calculamos la derivada del Error respecto a la salida.
   - Multiplicamos por la derivada de la salida respecto a la capa anterior.
   - Repetimos hasta llegar a la primera capa.

$$\frac{\partial \text{Error}}{\partial w_{\text{capa1}}} = \frac{\partial \text{Error}}{\partial \text{Out}} \cdot \frac{\partial \text{Out}}{\partial \text{L}_L} \cdot \dots \cdot \frac{\partial \text{L}_2}{\partial w_{\text{capa1}}}$$

**Sin esta regla, sería matemáticamente imposible entrenar modelos como GPT-4 o redes de detección de fraudes complejas.**

---

## 💻 Ejemplo en Python: Composición de Funciones

```python
# Definimos dos funciones:
# g(x) = x^2  (Función interna)
# f(u) = sin(u) (Función externa)
# Composición: h(x) = sin(x^2)

import numpy as np

def g(x): return x**2
def dg(x): return 2*x # Derivada de x^2

def f(u): return np.sin(u)
def df(u): return np.cos(u) # Derivada de sin(u)

def regla_de_la_cadena(x):
    u = g(x)
    # dy/dx = f'(g(x)) * g'(x)
    return df(u) * dg(x)

x_val = 1.5
print(f"Derivada de sin(x^2) en x={x_val} es: {regla_de_la_cadena(x_val)}")
```

#matematicas #calculus #backpropagation #deep-learning #banking #ia
