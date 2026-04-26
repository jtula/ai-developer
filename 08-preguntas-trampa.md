# Preguntas Trampa y Difíciles

**Pregunta:** ¿Cuándo NO usarías un agente de IA?
**Respuesta:** Cuando el flujo es 100% predecible, requiere latencia de milisegundos, o la precisión matemática es crítica sin margen de error. Un pipeline determinista (código tradicional) es mejor si no hay toma de decisiones ambiguas.

**Pregunta:** Tu agente sigue alucinando una librería que no existe en el proyecto. ¿Cómo lo arreglas?
**Respuesta:** Le quito el "libre albedrío". En lugar de dejar que infiera dependencias, le paso la salida del `package.json` o `pom.xml` en el prompt base y le doy una instrucción estricta de validación cruzada.

**Pregunta:** El cliente se queja de que el análisis tarda demasiado (10 minutos por repo).
**Respuesta:** Verifico los cuellos de botella. Cambio la ejecución secuencial de LangGraph a paralela para archivos independientes. Uso modelos más rápidos (Claude 3 Haiku) para ruteo/triage y guardo el modelo grande solo para la síntesis final.

**Pregunta:** ¿Qué haces si el LLM simplemente ignora el system prompt?
**Respuesta:** Coloco la instrucción más crítica al *final* del prompt (los LLMs sufren de "lost in the middle"). Si falla, fuerzo el output con JSON Schema estricto.