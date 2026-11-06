# 🧠 Fundamentos de IA y Deep Learning: De la Neurona al Grafo de Computación | Deep Learning Foundations: From Neurons to Computational Graphs

**ES:** El Deep Learning (DL) no es más que una extensión masiva del Machine Learning. Mientras que el ML tradicional depende de la "ingeniería de características" (el humano decide qué es importante), el DL utiliza el **Aprendizaje de Representaciones**: la red descubre por sí misma qué características son relevantes.

**EN:** Deep Learning (DL) is essentially a massive extension of Machine Learning. While traditional ML depends on "feature engineering" (the human decides what is important), DL uses **Representation Learning**: the network discovers for itself which features are relevant.

---

## 1. The Artificial Neuron: The Mathematical Unit | La Neurona Artificial: La Unidad Matemática

Una neurona es, en esencia, una función matemática que realiza una suma ponderada y aplica una transformación no lineal.

### A. La Operación Fundamental
$$\mathbf{z} = \sum_{i=1}^n (w_i x_i) + b$$
Donde:
- $x_i$: Entrada (Input).
- $w_i$: Peso (Weight) $\rightarrow$ Representa la **importancia** de la entrada.
- $b$: Sesgo (Bias) $\rightarrow$ Permite desplazar la función para ajustarse mejor a los datos.

### B. The Activation Function: The "On/Off" Switch
Sin una función de activación, una red neuronal, sin importar cuántas capas tenga, sería solo una gran regresión lineal. La **No Linealidad** es lo que permite a la IA aprender patrones complejos.

- **Sigmoide $\sigma(x)$:** Mapea a $[0, 1]$. Usada en la capa de salida para clasificación binaria.
- **Tanh:** Mapea a $[-1, 1]$. Mejor que la sigmoide para capas ocultas porque centra los datos en 0.
- **ReLU (Rectified Linear Unit):** $\max(0, x)$. El estándar de la industria. Elimina la negatividad y soluciona parcialmente el *Vanishing Gradient* (conectando con [[01 - Matemáticas/Cálculo y Optimización/CAL - Derivadas.md]]).

---

## 2. Deep Neural Networks (MLP) | Redes Neuronales Profundas (Multilayer Perceptron)

Una red "profunda" es aquella que tiene una capa de entrada, una capa de salida y múltiples **Capas Ocultas (Hidden Layers)**.

### A. Forward Propagation (El Flujo de la Información)
El dato viaja desde la entrada, pasando por transformaciones lineales y no lineales en cada capa, hasta llegar a la predicción final. Cada capa actúa como un **filtro de abstracción**:
- **Capa 1:** Detecta bordes o patrones simples.
- **Capa 2:** Combina bordes en formas (círculos, cuadrados).
- **Capa 3:** Combina formas en objetos (ojos, narices, ruedas).

### B. Backpropagation: The Learning Mechanism
Es la aplicación masiva de la **Regla de la Cadena** del cálculo.
1. Se calcula el error en la salida (**Loss Function**).
2. Se calcula la derivada del error respecto a cada peso, desde la última capa hacia la primera.
3. Se actualizan los pesos usando el **Descenso del Gradiente**.

---

## 3. Loss Functions: Measuring the Error | Funciones de Pérdida: Midiendo el Error

La elección de la función de pérdida define qué "castiga" el modelo.

- **MSE (Mean Squared Error):** $\frac{1}{n} \sum (y - \hat{y})^2$. Estándar para Regresión. Castiga fuertemente los outliers.
- **Binary Cross-Entropy (Log Loss):** Estándar para Clasificación Binaria. Mide la distancia entre dos distribuciones de probabilidad.
- **Categorical Cross-Entropy:** Para clasificación multiclase (ej. reconocer 10 dígitos diferentes).

---

## 4. Deep Learning Internals: Optimization & Stability | Internos de DL: Optimización y Estabilidad

Para entrenar redes profundas sin que colapsen, los expertos usan:

- **Weight Initialization:** Usar `He Initialization` o `Xavier Initialization` para evitar que los pesos empiecen siendo demasiado grandes o pequeños.
- **Batch Normalization:** Normalizar las activaciones entre capas. Esto acelera el entrenamiento y actúa como un regularizador.
- **Dropout:** Apagar neuronas aleatoriamente durante el entrenamiento. Obliga a la red a no depender de una sola neurona, reduciendo drásticamente el **Overfitting**.

---

## 💻 Professional Implementation (PyTorch Concept)

A diferencia de Scikit-Learn, en Deep Learning definimos la arquitectura del grafo.

```python
import torch
import torch.nn as nn
import torch.optim as optim

# Definición de una arquitectura Profesional
class DeepNet(nn.Module):
    def __init__(self):
        super(DeepNet, self).__init__()
        self.layer1 = nn.Linear(10, 64) # Entrada 10 -> Oculta 64
        self.relu = nn.ReLU()
        self.layer2 = nn.Linear(64, 32) # Oculta 64 -> Oculta 32
        self.output = nn.Linear(32, 1)  # Oculta 32 -> Salida 1

    def forward(self, x):
        x = self.relu(self.layer1(x))
        x = self.relu(self.layer2(x))
        return torch.sigmoid(self.output(x))

model = DeepNet()
criterion = nn.BCELoss() # Binary Cross Entropy
optimizer = optim.Adam(model.parameters(), lr=0.001)
```

#ai #deep-learning #neural-networks #pytorch #backpropagation #ml-internals
