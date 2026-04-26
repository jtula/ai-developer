# Análisis de Código Legacy

## Estrategias y Herramientas
- **AST (Abstract Syntax Tree):** Representación en árbol de la sintaxis. Librerías como `javalang` para Java, o `ast` en Python.
- **Tree-sitter:** Herramienta políglota y ultrarrápida. Genera árboles sintácticos de casi cualquier lenguaje. Ideal para extraer firmas, clases y docstrings sin ejecutar código.
- **Grep / Ripgrep / Ctags:** Búsquedas textuales rápidas para referencias y definiciones a través del Bash tool.

## Extracción de Contexto Eficiente
En vez de pasar 50 archivos Java, generamos un **Esqueleto**:
1. Extraer nombres de clases y sus firmas de métodos.
2. Pasar este "mapa" al LLM.
3. El LLM solicita ver el cuerpo de métodos específicos usando una tool `get_method_body(class, method)`.

## Generación de TO-BE
1. Identificar anti-patrones en el AS-IS (ej. God Classes, SQL injection manual).
2. Prompt al LLM para proponer diseño TO-BE en Hexagonal/Clean Architecture.
3. Generar output en Markdown con diagramas Mermaid integrados.

## Preguntas de Entrevista
**Pregunta:** ¿Cómo analizas dependencias circulares en un monolito viejo?
**Respuesta:** Construyo un grafo de dependencias estático (leyendo imports/requires con Tree-sitter), lo cargo en NetworkX (Python) para buscar ciclos y le paso el resumen al LLM.

**Pregunta:** El código no tiene documentación y las variables se llaman `x`, `y`, `z`. ¿Cómo procedes?
**Respuesta:** Uso un paso previo con un LLM rápido/barato para inferir tipos de datos y propósito basado en el uso, renombro simbólicamente, y luego hago el análisis profundo.