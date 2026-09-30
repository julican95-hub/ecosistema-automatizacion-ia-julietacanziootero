# Blueprints de Make

Esta carpeta contiene los Blueprints exportados de los dos escenarios principales implementados en Make.

Los archivos permiten revisar de forma directa la configuración técnica del sistema, incluyendo módulos, filtros, variables dinámicas, mapeos, prompts, rutas de decisión, Error Handlers y Commit.

## Archivos

### Escenario 1 — Pipeline HITL

Archivo:

`Entrega Final - Pipeline HITL.blueprint.json`

Función:

- detectar contenidos elegibles desde Airtable;
- recuperar Idea Semilla y Record ID;
- consultar Base de Conocimiento RAG;
- unificar contexto;
- generar el borrador con Claude Haiku 4.5;
- guardar el borrador en Airtable;
- cambiar el estado a En revisión;
- crear la solicitud Human-in-the-Loop;
- gestionar errores de Claude y Airtable mediante Error Handlers;
- registrar incidencias en LOG de Errores;
- cerrar ejecuciones fallidas mediante Commit.

## Escenario 2 — HITL Aprobación

Archivo:

`Entrega Final - HITL Aprobación.blueprint.json`

Función:

- recibir la respuesta humana del HITL;
- recuperar el contenido evaluado desde Airtable;
- procesar la decisión mediante Router;
- ejecutar la ruta Aprobar o Rechazar;
- enviar mediante Gmail el contenido aprobado;
- actualizar el registro a Publicado o Rechazado;
- gestionar errores de Gmail;
- registrar incidencias en LOG de Errores;
- cerrar ejecuciones fallidas mediante Commit.

## Verificación

Los Blueprints permiten contrastar la arquitectura documentada con la configuración real de Make.

Diagramas oficiales:

- [Escenario 1](../diagramas/)
- [Escenario 2](../diagramas/)

Documentación completa:

- [Documento final](../documentacion/)

Evidencias:

- [Evidencias E01–E18](../evidencias/)
