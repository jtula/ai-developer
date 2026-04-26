# RAG (Retrieval-Augmented Generation)

## ¿Qué es y por qué importa?
Técnica para darle contexto dinámico al LLM desde una base de conocimiento sin re-entrenarlo. Para código legacy, permite consultar repositorios masivos sin superar el context window.

## Pipeline RAG Clásico
1. **Ingesta:** Cargar archivos (ej. `.java`, `.js`).
2. **Chunking:** Dividir en fragmentos lógicos (por función/clase, no por caracteres aleatorios).
3. **Embeddings:** Convertir texto a vectores numéricos (ej. `text-embedding-3-small` de OpenAI).
4. **Vector Store:** Guardar vectores y metadatos (FAISS para local, Pinecone/Chroma/pgvector).
5. **Retrieval:** Buscar chunks con similitud de coseno contra la pregunta del usuario/agente.
6. **Generation:** Pasar los chunks al LLM como contexto.

## RAG para Código
- El chunking basado en caracteres rompe la sintaxis. Se debe usar **AST-based chunking** (Tree-sitter) para separar por nivel estructural (funciones, clases, interfaces).
- **Metadatos vitales:** Archivo, línea, dependencias importadas.

## Métricas de Evaluación
- **Precision:** ¿Los fragmentos devueltos contienen la respuesta?
- **Recall:** ¿Se recuperaron *todos* los fragmentos relevantes?

## Preguntas de Entrevista
**Pregunta:** El RAG está devolviendo fragmentos de código irrelevantes. ¿Cómo lo mejoras?
**Respuesta:** Implemento "HyDE" (Generar una respuesta hipotética primero y vectorizar eso), o uso Búsqueda Híbrida (Vectores + Búsqueda por palabras clave como BM25), además de Re-ranking (ej. Cohere Rerank) al final del retrieval.

**Pregunta:** ¿Cómo manejas un cambio de código en la base de datos vectorial?
**Respuesta:** Calculo un hash del archivo o chunk. Si cambia en Git, elimino los vectores viejos por su ID de metadatos y re-indexo solo los archivos modificados.