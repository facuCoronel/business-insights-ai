# Business Insights AI

Backend en Python que transforma consultas en lenguaje natural en reportes de negocio, planes de marketing y dashboards declarativos mediante inteligencia artificial. Combina una API REST con FastAPI y un servidor MCP para consultar datos de un sistema de negocio a través de un catálogo de capacidades.

## Funcionalidades

- **Reportes de negocio:** genera resúmenes ejecutivos, hallazgos y recomendaciones a partir de los datos consultados.
- **Planes de marketing:** propone objetivos, segmentos, acciones y métricas respaldados por la información disponible.
- **Dashboards declarativos:** devuelve un esquema JSON de componentes para que un frontend pueda representar indicadores, gráficos, tablas y recomendaciones.
- **Planificación y ejecución controlada:** el modelo elige capacidades de un catálogo descubierto; el backend valida el plan y ejecuta únicamente operaciones GET permitidas.
- **Validación de respuestas:** usa esquemas Pydantic, un intento de reparación y advertencias sobre datos incompletos o consultas fallidas.
- **Herramientas MCP:** permite descubrir y ejecutar capacidades del negocio desde clientes compatibles.

## Cómo funciona

1. Recibe una pregunta y un tipo de consulta (`business_report` o `marketing_plan`).
2. Carga la skill YAML correspondiente, con sus prompts, restricciones y esquema de salida.
3. Descubre el catálogo de capacidades de la API de negocio.
4. Usa un modelo de lenguaje para planificar las consultas y valida el plan contra el catálogo.
5. Ejecuta las consultas, reúne los resultados y registra su calidad y posibles fallos.
6. Genera y valida el reporte; cuando la skill lo habilita, también genera la UI declarativa.

El proyecto tiene un nombre genérico, pero el adaptador HTTP implementado actualmente integra FastFixPro. Para conectarlo a otro sistema hay que implementar un adaptador compatible con el puerto de capacidades y registrarlo en la inyección de dependencias.

## Tecnologías

Python 3.11+, FastAPI, Pydantic, Microsoft Agent Framework, Ollama mediante una API compatible con OpenAI, MCP, SQLAlchemy, PostgreSQL, pgvector y Alembic.

La organización sigue Clean Architecture: dominio, casos de uso e interfaces, adaptadores de infraestructura y capa HTTP. Incluye entidades y repositorios para documentos y fragmentos RAG; el flujo actual de `/business-chat` obtiene sus datos mediante el catálogo de capacidades.

## Instalación local

Requisitos: Python 3.11 o superior, PostgreSQL con pgvector para las tablas RAG, acceso a un modelo mediante Ollama y acceso a la API de negocio integrada.

```bash
git clone https://github.com/facuCoronel/business-insights-ai.git
cd business-insights-ai
python -m venv .venv
```

Activá el entorno virtual:

```powershell
# Windows PowerShell
.venv\Scripts\Activate.ps1
```

```bash
# Linux / macOS
source .venv/bin/activate
```

Instalá las dependencias y creá la configuración local:

```bash
python -m pip install -e ".[dev]"
```

Copiá `.env.example` como `.env` y completá las variables:

| Variable | Uso |
| --- | --- |
| `DB_CONNECTION_STRING` | Conexión PostgreSQL en formato `Host=...;Port=...;Database=...;Username=...;Password=...`. |
| `OLLAMA_BASE_URL` | URL base del servicio Ollama; el cliente agrega `/v1`. |
| `OLLAMA_API_KEY` | Credencial del servicio de modelos. |
| `OLLAMA_MODEL` | Modelo utilizado para planificar y generar respuestas. |
| `HUGGINGFACE_API_KEY` | Configuración de Hugging Face requerida por Settings. |
| `FASTFIXPRO_API_BASE_URL` | URL de la API de negocio del adaptador actual. |
| `FASTFIXPRO_API_TOKEN` | Token utilizado para descubrir y ejecutar capacidades. |
| `Rag__*` | Parámetros de fragmentación, embeddings y recuperación RAG. |

La configuración se carga desde `.env` y variables de entorno. Conservá los nombres `FASTFIXPRO_*` mientras uses el adaptador actual.

Aplicá las migraciones de las tablas gestionadas por este servicio:

```bash
alembic upgrade head
```

## API REST

```bash
python -m uvicorn main:app --reload --port 8000
```

- Swagger UI: `http://localhost:8000/docs`
- Estado del servicio: `GET /health`
- Consulta de negocio: `POST /business-chat`

Ejemplo de cuerpo de una consulta:

```json
{
  "intent_type": "business_report",
  "question": "¿Cómo evolucionaron las ventas y qué acciones recomendás?"
}
```

La respuesta incluye `request_id`, `intent_type`, `report`, `ui`, `data_quality` y `warnings`. `ui` puede ser `null`: su generación depende de la skill y puede fallar sin interrumpir la entrega del reporte. Los planes de marketing se solicitan con `"intent_type": "marketing_plan"`.

El campo opcional `params` se acepta en el request, pero actualmente no participa de la planificación ni de la ejecución. Los parámetros efectivos provienen del plan generado por el modelo.

## Servidor MCP

Iniciá el servidor con transporte stdio desde la raíz del proyecto, usando la misma configuración `.env`:

```bash
python mcp_server.py
```

Herramientas disponibles:

- `list_business_capabilities`: descubre las operaciones disponibles y sus parámetros.
- `call_business_capability`: ejecuta una capacidad del catálogo y devuelve los datos como JSON.

## Estructura del proyecto

```text
config/                 Configuración y variables de entorno
src/api/                Endpoints, esquemas HTTP e inyección de dependencias
src/application/        Casos de uso, interfaces y pasos del pipeline
src/domain/             Entidades, objetos de valor y contratos de repositorios
src/infrastructure/     Adaptadores de IA, HTTP, persistencia y skills YAML
src/mcp/                Catálogo de capacidades y herramientas MCP
alembic/                Migraciones de base de datos
tests/                  Pruebas unitarias y de integración
docs/                   Documentación del flujo de negocio
```

## Pruebas

```bash
python -m pytest
```

Para más detalle sobre el pipeline, consultá [Business Chat Process](docs/business_chat_process.md).
