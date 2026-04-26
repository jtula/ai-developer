# LangChain & LangGraph

## LangChain: Conceptos Esenciales
- **Chains:** Secuencias lineales (Prompt -> LLM -> Output Parser).
- **Tools:** Funciones ejecutables por el modelo.
- **Memory:** Retención de contexto conversacional. Hoy en día se delega al estado del grafo.

## LangGraph
Extensión de LangChain para crear flujos cíclicos y agentes stateful. Se basa en grafos dirigidos (DAGs con ciclos).
- **Nodes:** Funciones de Python (ej. llamar al LLM, ejecutar una tool).
- **Edges:** Conexiones condicionales (ej. `if tool_call_exists -> ir a nodo Tools, else -> END`).
- **State:** Un objeto (ej. `TypedDict` o `Pydantic`) que muta a medida que avanza por los nodos.

## Orquestación y Paralelismo
LangGraph permite ramificar la ejecución. Por ejemplo, el agente AS-IS delega el análisis de frontend y backend a dos sub-agentes en paralelo, y un nodo "Sintetizador" junta los resultados.

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List

class AgentState(TypedDict):
    messages: List[str]
    code_summary: str

def analyze_node(state: AgentState):
    return {"code_summary": "Legacy Java code found."}

workflow = StateGraph(AgentState)
workflow.add_node("analyze", analyze_node)
workflow.set_entry_point("analyze")
workflow.add_edge("analyze", END)
app = workflow.compile()
```

## Preguntas de Entrevista
**Pregunta:** ¿Cuándo usarías LangGraph en lugar de un bucle `while` normal en Python?
**Respuesta:** Cuando el flujo requiere memoria persistente (checkpoints), human-in-the-loop (pausar para aprobación manual), ciclos complejos o streaming de estados intermedios.

**Pregunta:** ¿Cómo comunicas dos agentes distintos en LangGraph?
**Respuesta:** Mediante el `State` compartido. Un agente escribe sus hallazgos en una clave del estado (ej. `state['backend_analysis']`) que el siguiente agente lee como parte de su prompt.