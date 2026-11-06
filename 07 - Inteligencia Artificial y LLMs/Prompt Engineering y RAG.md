# 🎯 Prompt Engineering y Arquitecturas RAG: Sistemas de Conocimiento | Prompt Engineering & RAG: Knowledge Systems

**ES:** Un LLM por sí solo es una "mente" brillante pero sin memoria fresca. El **Prompt Engineering** es la interfaz de control para dirigir ese razonamiento, mientras que el **RAG (Retrieval Augmented Generation)** es la infraestructura que le otorga una memoria externa, veraz y actualizada.

**EN:** An LLM on its own is a brilliant "mind" but with no fresh memory. **Prompt Engineering** is the control interface to steer that reasoning, while **RAG (Retrieval Augmented Generation)** is the infrastructure that grants it an external, truthful, and updated memory.

---

## 1. Industrial Prompting: The Control Contract | Prompting Industrial: El Contrato de Control

Para aplicaciones profesionales, el prompt no es una pregunta, es un **Contrato de Entrada**. Un experto no "prueba" prompts, diseña estructuras de control.

### A. Advanced Reasoning Techniques
- **Chain-of-Thought (CoT):** Forzar al modelo a generar una secuencia de razonamiento paso a paso antes de la respuesta final. 
  - *Prompt:* "Piensa paso a paso y analiza los pros y contras antes de dar la conclusión".
- **Few-Shot Prompting:** Proporcionar ejemplos exactos de la relación $\text{entrada} \to \text{salida}$. Esto reduce la varianza y asegura que el formato de salida (ej. JSON) sea perfecto.
- **ReAct (Reason + Act):** El modelo razona sobre la tarea, decide ejecutar una herramienta (como una búsqueda SQL) y luego observa el resultado para razonar el siguiente paso.

### B. The System Prompt (The Guardrails)
Es la capa de instrucciones invisibles que define la personalidad, las restricciones y el formato del modelo.
- **Estructura Pro:** `[Rol] + [Contexto] + [Restricciones] + [Formato de Salida]`.
- **Ejemplo:** "Actúa como un auditor de riesgos bancarios. Analiza el texto adjunto. Restricción: No inventes datos. Formato: Devuelve solo un JSON con las llaves 'riesgo' y 'evidencia'".

---

## 2. RAG Deep Dive: The Knowledge Pipeline | RAG: El Pipeline de Conocimiento

El RAG soluciona las alucinaciones y la falta de datos privados. Transforma al LLM de un "generador de texto" a un "analista de documentos".

### A. The Vector Engine (How it works internally)
1. **Chunking:** Los documentos se dividen en fragmentos (ej. 500 tokens) con un *overlap* (solapamiento) para que la información no se corte a la mitad de una idea.
2. **Embeddings:** Cada fragmento se convierte en un vector mediante un modelo de embedding (ej. `text-embedding-3-small`).
3. **Vector DB & HNSW:** Los vectores se guardan en bases de datos como Pinecone o Milvus. Utilizan el índice **HNSW (Hierarchical Navigable Small World)**, que permite buscar los vectores más cercanos en milisegundos mediante un grafo de proximidad.

### B. The RAG Loop (The Workflow)
$\text{Consulta Usuario} \rightarrow \text{Embedding de Consulta} \rightarrow \text{Búsqueda de Coseno en Vector DB} \rightarrow \text{Top K fragmentos} \rightarrow \text{Prompt Aumentado} \rightarrow \text{LLM} \rightarrow \text{Respuesta}$.

---

## 3. Professional Implementation: LangChain & Vector Stores | Implementación Profesional

```python
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain.chains import RetrievalQA

# 1. Construcción del Cerebro de Datos (Vector Store)
embeddings = OpenAIEmbeddings()
# Cargamos documentos y los indexamos en el espacio vectorial
vector_db = Chroma.from_texts(
    texts=["El torque del tornillo M12 es 110Nm", "El aceite se cambia cada 500h"], 
    embedding=embeddings
)

# 2. Configuración del Agente de Respuesta
llm = ChatOpenAI(model="gpt-4o", temperature=0)
rag_chain = RetrievalQA.from_chain_type(
    llm=llm, 
    retriever=vector_db.as_retriever(search_kwargs={"k": 3}) # Recupera los 3 fragmentos más similares
)

# 3. Consulta con Evidencia (Sourced Answer)
response = rag_chain.invoke("¿Cuál es el torque del tornillo M12?")
print(response)
```

---

## 4. The Next Frontier: Autonomous Agents | La Siguiente Frontera: Agentes Autónomos

Un sistema RAG es pasivo (solo lee). Un **Agente** es activo (puede actuar).
Un agente utiliza un ciclo de **Planificación $\rightarrow$ Acción $\rightarrow$ Observación**:
1. **Plan:** "Para responder esto, primero necesito consultar la DB de ventas y luego buscar el manual en la web".
2. **Act:** Ejecuta una función de Python o una API.
3. **Observe:** Analiza el resultado y ajusta el plan.

#rag #llms #prompt-engineering #langchain #vector-db #ai-agents #embeddings
