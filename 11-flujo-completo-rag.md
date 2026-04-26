# Deep Dive: Flujo Completo de un Sistema RAG

Para una entrevista de Senior AI Developer, no basta con listar los pasos. Debes demostrar que entiendes *qué pasa por debajo de la caja negra* en cada etapa de Retrieval-Augmented Generation. 

Aquí tienes la explicación técnica y detallada del pipeline:

## 1. Data Ingestion & Parsing (Limpieza)
Antes de vectorizar, la data debe estar limpia. Meter basura genera vectores basura.
- **Texto regular:** Limpiar HTML tags, caracteres invisibles, normalizar espacios.
- **PDFs/Imágenes:** Usar OCR (Tesseract, AWS Textract) o librerías de layout (Unstructured) para extraer texto sin perder el orden (ej. tablas, columnas).
- **Código:** Usar Tree-sitter para aislar bloques lógicos (métodos, clases) y quitar comentarios generados por IDEs si no aportan valor.

## 2. Estrategias de Chunking (Segmentación)
Los modelos de embedding tienen un límite de tokens (ej. 8192 para `text-embedding-3`). 
- **Fixed-size Chunking:** Dividir en bloques de 500 tokens con un "overlap" (solapamiento) de 50 tokens para no cortar ideas por la mitad. *Fácil pero propenso a romper contexto.*
- **Semantic Chunking:** Romper el documento por oraciones o párrafos lógicos usando NLP (NLTK o Spacy).
- **Document-based (Código):** Separar cada clase/función entera en su propio chunk. Si la función es muy larga, generar un resumen de la función, vectorizar el resumen, pero al recuperar, devolver la función entera (patrón *Parent-Child Retrieval*).

## 3. Generación de Embeddings (El "Cerebro" de la similitud)
Un modelo de embedding (como `OpenAI text-embedding-3-large` o `Cohere`) es una red neuronal pre-entrenada para mapear texto a un espacio matemático n-dimensional (ej. 1536 dimensiones).
- **¿Qué es un Vector?** Una lista de miles de números float. Ej: `[0.012, -0.045, 0.881, ...]`.
- **Propiedad Clave:** Textos con significados similares (ej. "perro" y "cachorro") terminan siendo vectores matemáticamente muy cercanos en este espacio multidimensional, aunque no compartan letras.

## 4. Vector Store (Almacenamiento e Indexación)
Buscar comparando 1 vector contra 10 millones de vectores secuencialmente (KNN - K-Nearest Neighbors) es lentísimo. Las BDs Vectoriales (Pinecone, Qdrant, Milvus, pgvector) usan índices aproximados (ANN - Approximate Nearest Neighbors):
- **HNSW (Hierarchical Navigable Small World):** El estándar de la industria. Construye grafos en capas. Es rapidísimo para buscar el vecino más cercano sin recorrer toda la base de datos.
- **Metadatos:** Junto al vector se guardan JSONs (ej. `{"file": "auth.py", "date": "2024-01"}`). Fundamental para hacer "Pre-filtering" (ej. "Busca esta función, pero SÓLO en archivos de 2024").

## 5. Retrieval (Búsqueda y Comparación)
El usuario hace una pregunta ("¿Cómo funciona el login?").
1. **Embedding del query:** La pregunta se pasa por el mismo modelo de embedding para generar su vector.
2. **Cálculo de Distancia:** La BD vectorial compara el vector de la pregunta contra la BD usando funciones matemáticas:
   - **Distancia del Coseno (Cosine Similarity):** Mide el ángulo entre los vectores (ignora la longitud, se enfoca en la dirección/significado). Es la más usada.
   - **Producto Punto (Dot Product):** Multiplicación de vectores (útil si los vectores están normalizados).
3. **Top-K:** La BD devuelve los `K` chunks más cercanos (ej. los 5 mejores resultados).

## 6. Técnicas Avanzadas de Retrieval (Para Senioridad)
Lo que separa a un Junior de un Senior en RAG:
- **Búsqueda Híbrida:** Unir Búsqueda Vectorial (semántica) + BM25 (keyword exacta). Útil porque a veces el usuario busca un ID específico de error (`ERR-909`) y la similitud semántica falla.
- **Reranking (Cross-Encoders):** El retrieval vectorial es rápido pero "barato" en precisión. Tras sacar los Top-20 con vectores, pasamos esos 20 a un modelo Reranker (ej. `Cohere Rerank`) que hace una comparación profunda texto vs texto y los re-ordena, quedándose con los Top-5 reales.
- **Query Expansion / HyDE:** El usuario pregunta "¿Problema de red?". El LLM expande la pregunta a "Problemas de red, timeouts de TCP, latencia DNS" ANTES de vectorizar la pregunta, mejorando la coincidencia.

## 7. Prompt Synthesis (Generación)
Se toma el Top-K de chunks recuperados, se insertan en un *System Prompt* y se envía al LLM generativo (GPT-4 / Claude):
```text
Eres un experto. Responde a la pregunta basándote SÓLO en el siguiente contexto:
---
[Chunk 1: Extracto de auth.py sobre tokens JWT]
[Chunk 2: Documentación interna de expiración de tokens]
---
Pregunta del usuario: ¿Por qué caducan mis sesiones en 5 mins?
```
El LLM sintetiza la respuesta sin alucinar, ya que está anclado a la verdad provista en los chunks.