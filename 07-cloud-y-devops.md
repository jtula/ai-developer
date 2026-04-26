# Cloud & DevOps

## Cloud Básico (AWS/Azure)
- **Almacenamiento:** S3 / Azure Blob Storage para guardar repositorios descargados y artefactos de salida (diagramas, PDFs).
- **Cómputo serverless:** AWS Lambda / Azure Functions para tareas cortas de análisis (ojo con los timeouts de 15 min).
- **Colas:** SQS / Azure Service Bus para orquestar trabajos pesados asíncronos.

## Git y Docker
- **Git Avanzado:** Necesario para analizar histórico. `git log -p` para ver evolución de archivos, `git diff` para entender qué cambió.
- **Docker:** Empaquetar la API de FastAPI y los scripts de LangGraph en contenedores.
  - Multi-stage builds para reducir tamaño de imagen.
  - No correr contenedores como `root`.

## Preguntas de Entrevista
**Pregunta:** Necesitas desplegar un worker que consume mucha RAM haciendo RAG. ¿Dónde lo alojas?
**Respuesta:** Evitaría Serverless (Lambda) por el límite de RAM/tiempo y el cold start al cargar modelos. Usaría un contenedor en ECS (AWS) o Azure Container Apps, escalando según la cola de mensajes.

**Pregunta:** ¿Cómo manejas las API Keys de OpenAI en tu infraestructura?
**Respuesta:** Nunca en código. Inyectadas mediante variables de entorno desde un Secret Manager (AWS Secrets Manager / Azure Key Vault) gestionado por Terraform.