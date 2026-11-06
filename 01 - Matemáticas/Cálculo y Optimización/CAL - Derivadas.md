# 📈 CAL - Derivadas: El Motor de la Optimización y el Grafo de Computación | Derivatives: Optimization Engine & Computational Graphs

## 1. From Slopes to Gradient Fields | De Pendientes a Campos de Gradientes

**ES:** En el nivel básico, la derivada es la pendiente de una recta. En el nivel experto, la derivada de una función multivariable es el **Gradiente ($\nabla$)**, un vector que apunta en la dirección de máximo crecimiento de la función.

**EN:** At a basic level, the derivative is the slope of a line. At an expert level, the derivative of a multivariable function is the **Gradient ($\nabla$)**, a vector that points in the direction of the steepest ascent of the function.

### 🔬 The Gradient Vector
Para una función $J(\mathbf{w})$ donde $\mathbf{w} = [w_1, w_2, \dots, w_n]$, el gradiente es el vector de todas las derivadas parciales:
$$\nabla J(\mathbf{w}) = \left[ \frac{\partial J}{\partial w_1}, \frac{\partial J}{\partial w_2}, \dots, \frac{\partial J}{\partial w_n} \right]^T$$
- **Geometría:** El gradiente siempre es perpendicular a las curvas de nivel (iso-costos) de la función.
- **IA Connection:** El objetivo del entrenamiento es movernos en la dirección $-\nabla J(\mathbf{w})$ para encontrar el mínimo del error.

---

## 2. The Chain Rule & Backpropagation | La Regla de la Cadena y el Backpropagation

La **Regla de la Cadena** es la ley matemática que permite que la IA aprenda. Sin ella, no existiría el Deep Learning.

### 🧠 The Computational Graph (Grafo de Computación)
Las librerías modernas (PyTorch, JAX) no derivan fórmulas, sino que construyen un grafo:
1. **Forward Pass:** El dato fluye desde la entrada $\rightarrow$ capas $\rightarrow$ salida. Se guardan los valores intermedios.
2. **Backward Pass:** Se calcula el gradiente del error respecto a la salida y se "propaga" hacia atrás usando la regla de la cadena:
   $$\frac{\partial \text{Loss}}{\partial w_{layer1}} = \frac{\partial \text{Loss}}{\partial \text{out}} \cdot \frac{\partial \text{out}}{\partial \text{layer2}} \cdot \frac{\partial \text{layer2}}{\partial \text{layer1}}$$

**Expert Note:** El Backpropagation es simplemente una implementación eficiente de la regla de la cadena que evita recalcular las mismas derivadas miles de veces (Dynamic Programming).

---

## 3. Second Order Derivatives: The Hessian Matrix | Derivadas de Segundo Orden: La Matriz Hessiana

Si el gradiente es la velocidad del cambio, la **Hessiana ($\mathbf{H}$)** es la aceleración (curvatura).

$$\mathbf{H}_{ij} = \frac{\partial^2 J}{\partial w_i \partial w_j}$$
- **Curvatura:** La Hessiana nos dice si estamos en un mínimo, un máximo o un **punto de silla (Saddle Point)**.
- **Saddle Points (El terror de la IA):** En espacios de miles de dimensiones, los mínimos locales son raros; lo común son los puntos de silla (donde algunas direcciones suben y otras bajan). Aquí el gradiente es 0, pero no es un mínimo.
- **Newton's Method:** Usa la inversa de la Hessiana para saltar directamente al mínimo: $\mathbf{w}_{t+1} = \mathbf{w}_t - \mathbf{H}^{-1} \nabla J(\mathbf{w})$.
    - **Problema:** Calcular $\mathbf{H}^{-1}$ para un modelo de 7B parámetros es computacionalmente imposible ($O(n^3)$). Por eso usamos optimizadores de primer orden (Adam).

---

## 4. Stability: Vanishing & Exploding Gradients | Estabilidad: Gradientes Desvanecientes y Explosivos

Cuando multiplicamos muchas derivadas (regla de la cadena) en redes muy profundas, ocurre un problema numérico crítico:

1. **Vanishing Gradient:** Si las derivadas son $< 1$ (ej. con la función Sigmoide), el gradiente se vuelve infinitesimalmente pequeño al llegar a las primeras capas. **Resultado:** El modelo deja de aprender.
2. **Exploding Gradient:** Si las derivadas son $> 1$, el gradiente crece exponencialmente. **Resultado:** Los pesos se vuelven `NaN` y el modelo diverge.

### 🛠️ Soluciones de Nivel Experto:
- **ReLU Activation:** $\max(0, x)$ mantiene la derivada en 1 para valores positivos, evitando que el gradiente desaparezca.
- **Gradient Clipping:** Forzar el gradiente a un valor máximo (ej. $\max 1.0$) para evitar explosiones.
- **Batch Normalization / LayerNorm:** Normalizar las activaciones para mantener la varianza constante y el flujo del gradiente estable.

---

## 💻 Implementation: Automatic Differentiation (Autograd)

A nivel profesional, no escribimos la derivada. Definimos la función y el framework calcula el gradiente mediante el **Mode de Diferenciación Automática**.

```python
import torch

# Definimos un modelo simple: y = w*x + b
x = torch.tensor([1.5], requires_grad=False)
w = torch.tensor([2.0], requires_grad=True)
b = torch.tensor([0.5], requires_grad=True)

# Forward Pass
y = w * x + b
loss = (y - 10.0)**2 # Queremos que y sea 10

# Backward Pass (Regla de la cadena automática)
loss.backward()

print(f"Gradiente w: {w.grad}") # dLoss/dw
print(f"Gradiente b: {b.grad}") # dLoss/db
```

#math #calculus #backpropagation #pytorch #deep-learning #gradient-descent #optimization
