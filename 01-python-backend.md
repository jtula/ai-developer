# Python Backend & APIs

## Conceptos Clave
- **Async/Await:** Esencial para I/O bound tasks (llamadas a LLMs, base de datos). Libera el event loop mientras se espera la respuesta.
- **Decorators:** Funciones que modifican otras. Útiles para logging de agentes, manejo de reintentos o inyección de dependencias.
- **Dataclasses / Pydantic:** Para definir estructuras de datos claras (estados de LangGraph, inputs/outputs de tools). Pydantic valida tipos en runtime.
- **Type Hints:** Obligatorios en código moderno. Mejoran el autocompletado y previenen errores estáticos (usando `mypy`).

## FastAPI: Estructura
Framework asíncrono basado en Pydantic y Starlette.
- **Endpoints:** Rutas definidas con decoradores (`@app.post("/analyze")`).
- **Autenticación:** Vía `Depends()` inyectando validación de API Keys o JWT tokens en los headers.

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel

app = FastAPI()

class CodeRequest(BaseModel):
    repo_url: str

@app.post("/api/v1/analyze")
async def analyze_repo(req: CodeRequest):
    # Procesamiento asíncrono
    return {"status": "processing", "url": req.repo_url}
```

## Manejo de Archivos y Subprocess
- **Lectura:** Usar `pathlib.Path` para navegación segura. Para archivos grandes, leer en chunks o usar generadores.
- **Subprocess:** Para ejecutar git clone, grep o CLI tools. Usar `subprocess.run(..., capture_output=True, text=True)`.

## Preguntas de Entrevista
**Pregunta:** ¿Diferencia entre multithreading y asyncio en Python?
**Respuesta:** Multithreading usa hilos del SO (sujeto al GIL de Python). Asyncio usa un solo hilo con un event loop cooperativo.
**Ejemplo:** Asyncio es ideal para hacer 10 peticiones a la API de OpenAI en paralelo sin overhead de hilos.

**Pregunta:** ¿Cómo manejas tareas de larga duración en FastAPI (ej. análisis de un repo de 1GB)?
**Respuesta:** No bloqueo la respuesta HTTP. Devuelvo un `task_id` (202 Accepted) y encolo el trabajo en Celery o un background task de FastAPI.