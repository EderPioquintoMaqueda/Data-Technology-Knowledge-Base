# 🚀 Transformers y LLMs | Transformers & Large Language Models

## ¿Qué es? | What is it?
**ES:** El Transformer es la arquitectura de red neuronal que revolucionó la IA en 2017 (artículo *"Attention is All You Need"*). A diferencia de las redes antiguas que leían el texto palabra por palabra (secuencialmente), el Transformer lee toda la secuencia a la vez, permitiéndole entender el contexto global de manera masiva y paralela.

**EN:** The Transformer is the neural network architecture that revolutionized AI in 2017 (*"Attention is All You Need"* paper). Unlike older networks that read text word by word (sequentially), the Transformer reads the entire sequence at once, allowing it to understand the global context massively and in parallel.

---

## 🛠️ La Arquitectura del Transformer | Transformer Architecture

El Transformer se compone de dos bloques principales: el **Encoder** (Codificador) y el **Decoder** (Decodificador).

### 1. El Encoder (El Entendedor)
Su trabajo es crear una representación rica del texto de entrada. 
- **Self-Attention**: Cada palabra "mira" a todas las demás palabras de la oración para entender su significado. 
  - *Ejemplo:* En "El banco está cerrado", la palabra "banco" mira a "cerrado" para saber que se refiere a una entidad financiera y no a un asiento de madera.

### 2. El Decoder (El Generador)
Usa la información del Encoder para generar una respuesta palabra por palabra.
- **Masked Self-Attention**: El decoder no puede "mirar el futuro"; solo puede ver las palabras que ya ha generado para predecir la siguiente.

### 3. LLMs (Large Language Models)
Los modelos como GPT (Generative Pre-trained Transformer) son esencialmente **Decoders gigantes**. Han sido entrenados con billones de palabras para predecir la siguiente palabra más probable en una secuencia.

---

## ⚙️ Conceptos Técnicos Clave | Key Technical Concepts

### 1. Tokens y Vocabulario
Los LLMs no ven palabras, ven **Tokens** (trozos de palabras).
- **Sustancia**: Un modelo con un vocabulario de 50,000 tokens representa cada palabra como un índice en una matriz gigante.

### 2. Temperatura ($\text{Temperature}$)
Es un parámetro que controla la creatividad de la respuesta:
- **Temp $\rightarrow 0$**: Respuestas deterministas, precisas y repetitivas (Ideal para código o datos).
- **Temp $\rightarrow 1+$**: Respuestas creativas, diversas y con más riesgo de "alucinaciones" (Ideal para poesía o brainstorming).

---

## 💻 Implementación de un LLM Local (Ollama/HuggingFace) | Technical Implementation

Hoy en día, no necesitas entrenar un modelo desde cero; usas modelos pre-entrenados.

#### Ejemplo: Generación de Texto con Transformers
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

model_name = "gpt2" # Ejemplo con un modelo pequeño
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# 1. Tokenización de la entrada
input_text = "The future of mechanical engineering is"
inputs = tokenizer(input_text, return_tensors="pt")

# 2. Generación de la respuesta
outputs = model.generate(**inputs, max_length=50, temperature=0.7, do_sample=True)

# 3. Decodificación a texto humano
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

---

## Conceptos Relacionados | Related Concepts
- [[07 - Inteligencia Artificial y LLMs|NLP y LLMs]] $\longleftrightarrow$ El Transformer es la implementación técnica del mecanismo de Atención.
- [[07 - Inteligencia Artificial y LLMs|Fundamentos de IA y Deep Learning]] $\rightarrow$ Los Transformers son redes neuronales profundas optimizadas.

---
#transformers #llm #gpt #deep-learning #ai
