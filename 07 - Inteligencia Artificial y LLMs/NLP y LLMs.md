# 🗣️ NLP y el Camino hacia los LLMs | NLP & The Road to LLMs

## ¿Qué es? | What is it?
**ES:** El Procesamiento de Lenguaje Natural (NLP) es la capacidad de las máquinas para entender, interpretar y generar lenguaje humano. Ha evolucionado desde reglas gramaticales rígidas hasta la comprensión semántica profunda mediante el uso de vectores.

**EN:** Natural Language Processing (NLP) is the ability of machines to understand, interpret, and generate human language. It has evolved from rigid grammatical rules to deep semantic understanding through the use of vectors.

---

## 🛠️ La Evolución de la Representación del Texto | Text Representation Evolution

Para que una IA entienda el texto, debemos convertir palabras en números.

### 1. One-Hot Encoding (El inicio)
Cada palabra es un vector con un solo "1" y el resto "0". 
- **Problema**: Si tienes 50,000 palabras, cada palabra es un vector de 50,000 dimensiones. No hay noción de significado ( a "perro" y un "canino" son vectores totalmente distintos).

### 2. Word Embeddings (El salto semántico)
Palabras representadas en espacios vectoriales de baja dimensión (Ej: 300 dimensiones) donde palabras similares están físicamente cerca.
- **Ejemplos**: Word2Vec, GloVe.
- **Propiedad Mágica**: $\text{Vector}(\text{"Rey"}) - \text{Vector}(\text{"Hombre"}) + \text{Vector}(\text{"Mujer"}) \approx \text{Vector}(\text{"Reina"})$.

### 3. Contextual Embeddings (La revolución)
Una misma palabra tiene significados distintos según el contexto (Ej: "Banco" de sentarse vs "Banco" de dinero). Los modelos modernos (BERT, GPT) generan el vector de la palabra basándose en las palabras que la rodean.

---

## ⚙️ Conceptos Técnicos Fundamentales | Core Technical Concepts

### 1. Tokenización (Tokenization)
El proceso de dividir el texto en unidades más pequeñas llamadas **Tokens**. Un token no es necesariamente una palabra; puede ser una parte de una palabra (subword).
- **Ejemplo**: "Inconcebible" $\rightarrow$ `["In", "conceb", "ible"]`.

### 2. Atención (Attention Mechanism)
La capacidad del modelo de "enfocarse" en las palabras más importantes de una oración para entender el significado.
- **Fórmula Simplificada**: $\text{Atención} = \text{Softmax}(\frac{QK^T}{\sqrt{d_k}})V$
  - $Q$ (Query): ¿Qué estoy buscando?
  - $K$ (Key): ¿Qué información tiene esta palabra?
  - $V$ (Value): El valor real de la palabra.

---

## 💻 Implementación Técnica (HuggingFace) | Technical Implementation

La librería estándar para NLP hoy es `Transformers` de HuggingFace.

#### Ejemplo: Análisis de Sentimiento
```python
from transformers import pipeline

# Cargar un pipeline pre-entrenado de análisis de sentimiento
classifier = pipeline("sentiment-analysis")

results = classifier([
    "The machine is working perfectly, I am very happy!", 
    "The sensor is failing and the production stopped."
])

for res in results:
    print(f"Label: {res['label']}, Score: {res['score']:.4f}")
```

---

## Conceptos Relacionados | Related Concepts
- [[01 - Matemáticas|Álgebra Lineal]] $\longleftrightarrow$ Los Embeddings son vectores de alta dimensión.
- [[07 - Inteligencia Artificial y LLMs|Fundamentos de IA y Deep Learning]] $\rightarrow$ El NLP moderno se basa en redes neuronales profundas.
- [[07 - Inteligencia Artificial y LLMs|Transformers y LLMs]] $\rightarrow$ La arquitectura que implementa la Atención.

---
#nlp #embeddings #huggingface #tokens
