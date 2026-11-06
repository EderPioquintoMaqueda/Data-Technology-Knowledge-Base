# 🎯 Prompt Engineering y Arquitecturas RAG (Sistemas de Conocimiento)

El Prompt Engineering es la interfaz de control del LLM, mientras que el RAG es la infraestructura que le da una "memoria" externa y veraz.

## 1. Prompting de Grado Industrial

Para aplicaciones profesionales, el prompt no es una pregunta, es un **Contrato de Entrada**.

### Técnicas de Control de Salida:
- **Few-Shot Prompting**: Proporcionar ejemplos exactos del formato $\text{entrada} \to \text{salida}$. Reduce drástCamente la varianza de la respuesta.
- **Chain-of-Thought (CoT)**: Forzar al modelo a generar una secuencia de razonamiento lógico antes de dar la respuesta final.
- **System Prompting**: Definir el "espacio de estados" del modelo (rol, restricciones, formato de salida JSON).

---

## 2. RAG: Retrieval Augmented Generation

El RAG soluciona el problema de las alucinaciones y la falta de datos actualizados en los LLMs.

### El Pipeline Técnico del RAG:
1. **Ingesta y Chunking**: Dividir documentos masivos en trozos pequeños (chunks) con solapamiento para no perder el contexto.
2. **Vectorización**: Convertir chunks en embeddings mediante un modelo como `text-embedding-3-small`.
3. **Almacenamiento**: Guardar los vectores en una **Vector Database** (ChromaDB, Pinecone) utilizando índices de proximidad (HNSW).
4. **Recuperación**: Al recibir una pregunta $\mathbf{q}$, se buscan los $k$ vectores más cercanos mediante la **Similitud del Coseno**.
5. **Generación**: El LLM recibe: `Contexto recuperado` + `Pregunta` $\to$ `Respuesta basada en hechos`.

---

## 3. Implementación con LangChain y Vector DBs

```python
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain.chains import RetrievalQA

# 1. Crear el cerebro de datos (Vector Store)
embeddings = OpenAIEmbeddings()
vector_db = Chroma.from_texts(
    texts=["El torque del tornillo M12 es 110Nm", "El aceite se cambia cada 500h"], 
    embedding=embeddings
)

# 2. Configurar la cadena de respuesta
llm = ChatOpenAI(model="gpt-4o", temperature=0)
rag_chain = RetrievalQA.from_chain_type(
    llm=llm, 
    retriever=vector_db.as_retriever(search_kwargs={"k": 3})
)

# 3. Consulta con evidencia
print(rag_chain.invoke("¿Cada cuánto se cambia el aceite?"))
```

---

## 🎯 El Futuro: Agentes Autónomos
El siguiente paso es pasar del RAG (solo lectura) a los **Agentes** (capaces de ejecutar acciones). Un agente puede:
1. Analizar la pregunta.
2. Decidir usar la herramienta `SQL_Query` para obtener datos de la base de datos.
3. Usar la herramienta `Search_Web` para buscar manuales.
4. Sintetizar todo en una respuesta final.

#rag #llms #prompt-engineering #langchain #vector-db #ai-agents
