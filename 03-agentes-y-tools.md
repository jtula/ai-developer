# Agentes de IA y Tools

## ¿Qué es un Agente?
Un sistema donde el LLM actúa como el "cerebro" para tomar decisiones dinámicas y ejecutar acciones mediante herramientas (Tools) en un bucle (ReAct: Reason + Act) hasta alcanzar un objetivo.

## Tipos de Agentes para este rol
- **Agente AS-IS (Descubrimiento):** Explora el código actual. Usa tools como `read_file`, `search_regex`, `get_ast_tree` para mapear dependencias y arquitecturas legacy.
- **Agente TO-BE (Diseño):** Toma el contexto del AS-IS y usa tools de validación de diseño, generación de diagramas (Mermaid) y redacción de RFCs para proponer modernizaciones.

## Diseño de Tools (Function Calling)
- **Firma y Descripción:** El LLM elige la tool basándose en el docstring o la descripción del esquema JSON.
- **Integración:** El LLM devuelve un JSON con `{ "name": "tool", "arguments": {...} }`. El backend ejecuta la función y devuelve el resultado al LLM.

```python
# Ejemplo de definición de Tool (LangChain/OpenAI)
from langchain_core.tools import tool

@tool
def find_references(function_name: str) -> list[str]:
    """Busca todas las referencias a una función en el código."""
    # Lógica con grep o tree-sitter
    return ["src/main.py:45", "src/utils.py:12"]
```

## Preguntas de Entrevista
**Pregunta:** ¿Por qué usar Tool Calling en lugar de pedirle al LLM que devuelva un JSON crudo?
**Respuesta:** Tool Calling está fine-tuneado a nivel de modelo para ser determinista, asegurando tipos de datos correctos y manejando múltiples llamadas simultáneas (Parallel Tool Calling) de forma nativa.

**Pregunta:** ¿Qué haces si el agente se queda en un bucle infinito llamando a la misma tool?
**Respuesta:** Implemento un contador máximo de iteraciones (ej. `max_steps=5`) en el pipeline de orquestación y devuelvo un error controlado al modelo para obligarlo a cambiar de estrategia.