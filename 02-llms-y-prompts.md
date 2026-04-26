# LLMs y Prompt Engineering

## Funcionamiento Básico
- **Tokens:** Unidad básica de texto (aprox 3/4 de palabra). Los modelos cobran y limitan por token.
- **Context Window:** Límite máximo de tokens (input + output). GPT-4 Turbo: 128k, Claude 3 Opus: 200k.
- **Temperatura:** Controla la aleatoriedad. `0.0` = determinista (ideal para código/JSON), `0.7+` = creativo.

## Proveedores Principales
- **OpenAI (GPT-4o):** Excelente razonamiento general y tool calling. Estándar de la industria.
- **Anthropic (Claude 3.5 Sonnet):** Superior en tareas de codificación, lectura de contextos masivos (200k) y generación de XML/Markdown. Ideal para análisis de código legacy.
- **Google (Gemini 1.5 Pro):** Contexto hipermasivo (1M-2M tokens). Útil para meter un repositorio entero en el prompt sin chunking agresivo.

## Prompt Engineering
- **System Prompts:** Define el rol, restricciones y formato de salida.
- **Few-Shot:** Proveer 2-3 ejemplos de input/output. Mejora drásticamente la estructura de salida.
- **Chain-of-Thought (CoT):** Pedir al modelo que "piense paso a paso" antes de responder. Reduce alucinaciones en lógica compleja.
- **Structured Output:** Forzar la salida en JSON usando `response_format={ "type": "json_object" }` (OpenAI) o schemas de Pydantic.

## Optimización de Contexto
- **Chunking:** Partir código en funciones/clases para no saturar la ventana de contexto.
- **Filtrado:** Ignorar `node_modules`, `tests`, `binarios`.
- **Resumen Previo:** Usar un modelo pequeño/barato para resumir archivos, y pasar los resúmenes al modelo grande.

## Preguntas de Entrevista
**Pregunta:** ¿Cómo evitas que el LLM alucine dependencias inexistentes en código legacy?
**Respuesta:** Le obligo a citar el número de línea exacto del código proporcionado o uso un flag de temperatura `0.0` junto con un prompt de "responde solo basado en el contexto".

**Pregunta:** Tienes un archivo legacy de 500k tokens y usas GPT-4 (128k). ¿Qué haces?
**Respuesta:** Parsing semántico. Extraigo solo las firmas de funciones usando AST, pido al LLM que seleccione cuáles le interesan, y luego le envío solo el cuerpo de esas funciones.