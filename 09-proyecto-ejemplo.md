# Proyecto Ejemplo End-to-End

## Título: Legacy2Modernizer

Plataforma que ingesta un repo legacy, mapea dependencias y genera un RFC con propuesta de microservicios.

## Arquitectura (Ascii)

[GitHub Webhook] -> [FastAPI] -> [Encola en Celery]
[Worker Python]
|-> 1. Clona Repo (Subprocess)
|-> 2. AST Parser extrae esqueletos -> [Pinecone/VectorStore]
|-> 3. LangGraph (Agente Orquestador)
|-> Sub-Agente AS-IS (lee BD vectorial, consulta tools de grep)
|-> Sub-Agente TO-BE (recibe contexto, genera diseño hexagonal)
|-> 4. LLM genera Markdown + Mermaid
|-> 5. Output guardado en S3 y notifica al usuario.

## Pitch de 2 Minutos para Entrevista

"He diseñado un flujo donde recibimos el repo en una API FastAPI. En background, parseamos la estructura con Tree-sitter para crear índices y vectores exactos. Luego, un grafo de LangGraph toma el control: un Agente 'AS-IS' usa tools de búsqueda para mapear la deuda técnica y un Agente 'TO-BE' recibe esa radiografía para generar un diseño moderno. Uso Claude 4.6 Sonnet por su ventana de contexto masiva. Todo el output se estandariza en Markdown con diagramas Mermaid y se devuelve al cliente de forma asíncrona. Así evitamos saturar tokens y garantizamos cero alucinaciones basándonos estrictamente en el código existente."
