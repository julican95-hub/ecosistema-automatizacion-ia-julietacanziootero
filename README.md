# Ecosistema de Automatización IA Autónomo para Negocios

## Entrega Final — Julieta Canzio Otero

Proyecto final de automatización end-to-end orientado a la generación, revisión y salida controlada de contenido destinado a LinkedIn.

La solución integra:

- Make como orquestador.
- Airtable como memoria y persistencia.
- Claude Haiku 4.5 como motor de IA.
- Base de Conocimiento RAG.
- Human-in-the-Loop (HITL).
- Router de decisión.
- Gmail como canal de salida implementado.
- Error Handlers.
- Commit.
- LOG de Errores.
- Variables dinámicas.
- Prevención de loops.
- Dashboard Ejecutivo.
- Shared View pública.

---

# 1. Objetivo del proyecto

El sistema automatiza un pipeline de generación de contenido profesional.

El proceso comienza con una **Idea Semilla** registrada en Airtable.

Make detecta únicamente registros elegibles, recupera contexto desde una **Base de Conocimiento RAG**, genera un borrador mediante Claude y guarda el resultado nuevamente en Airtable.

Antes de realizar una acción externa, el flujo crea una solicitud **Human-in-the-Loop (HITL)**.

Una persona debe decidir entre:

- Approve.
- Reject.

La automatización continúa únicamente después de esa decisión.

**LinkedIn es el destino editorial del contenido.**

La implementación **no publica directamente mediante la API de LinkedIn**.

La salida técnica implementada en este proyecto es **Gmail**.

---

# 2. Stack tecnológico

| Componente | Tecnología | Función |
|---|---|---|
| Orquestador | Make | Coordinación del pipeline |
| Base de datos | Airtable | Memoria, estados, RAG y errores |
| IA | Claude Haiku 4.5 | Generación de borradores |
| Contexto | RAG | Reglas y conocimiento validado |
| Validación | Human-in-the-Loop | Aprobación o rechazo humano |
| Decisión | Router de Make | Separación de rutas |
| Salida | Gmail | Envío del contenido aprobado |
| Monitoreo | Airtable Interface | Dashboard Ejecutivo |

---

# 3. Arquitectura del sistema

La solución se divide en dos escenarios de Make.

## Escenario 1 — Entrega Final - Pipeline HITL

Flujo principal:

`Airtable → Variables → RAG → Agregación → Claude → Airtable → HITL`

Funciones principales:

1. Detectar contenido a generar.
2. Guardar Idea Semilla.
3. Guardar Record ID.
4. Buscar contexto RAG.
5. Unificar contexto RAG.
6. Recuperar Idea Semilla.
7. Recuperar Record ID.
8. Generar borrador con Claude.
9. Guardar borrador en Airtable.
10. Cambiar Estado a En revisión.
11. Solicitar aprobación humana mediante HITL.

También incluye Error Handlers para:

- errores de Claude;
- errores de Airtable;
- registro en LOG de Errores;
- notificación administrativa;
- cierre controlado mediante Commit.

### Archivos relacionados

[Ver diagramas oficiales](./diagramas/)

[Ver Blueprints](./blueprints/)

---

## Escenario 2 — Entrega Final - HITL Aprobación

Flujo principal:

`HITL → Airtable → Router`

### Rama Rechazar

`Router → Airtable → Rechazado`

### Rama Aprobar

`Router → Gmail → Airtable → Publicado`

También incluye Error Handler para fallos de Gmail.

Ante un fallo del canal de salida:

- el contenido cambia a Estado = Error;
- se registra Resultado final = Error en envío Gmail;
- se crea una incidencia en LOG de Errores;
- se conserva la trazabilidad mediante Record ID;
- la ejecución finaliza mediante Commit.

### Archivos relacionados

[Ver diagramas oficiales](./diagramas/)

[Ver Blueprints](./blueprints/)

---

# 4. Trigger inteligente

El Escenario 1 utiliza una vista de Airtable denominada:

`Por Generar`

Solo ingresan registros que cumplen simultáneamente:

- `Estado = Generando`
- `Idea Semilla no está vacía`

Esto permite:

- evitar datos incompletos;
- reducir operaciones innecesarias;
- evitar ejecuciones sobre registros no preparados;
- colaborar con la prevención de reprocesamiento.

---

# 5. Persistencia y variables dinámicas

El flujo utiliza variables dinámicas provenientes de módulos anteriores.

Entre ellas:

- Idea Semilla.
- Record ID.
- Contexto RAG.
- Borrador IA.
- Respuesta HITL.
- Estado.
- Canal.
- Fechas.
- Tipo de error.
- Mensaje de error.
- Contenido afectado.

El **Record ID de Airtable** permite mantener trazabilidad sobre el mismo registro durante toda la ejecución.

La Idea Semilla y el Record ID se conservan al inicio del escenario y se recuperan después de consultar y agregar la Base de Conocimiento RAG.

---

# 6. Base de Conocimiento RAG

La Base de Conocimiento RAG almacena información validada utilizada por Claude.

Incluye categorías como:

- Tono de marca.
- Audiencia.
- Reglas de contenido.
- Frases prohibidas.
- Servicios.

Make recupera únicamente registros activos mediante:

`{Activo}=1`

Luego agrega:

- Tema.
- Categoría.
- Contenido validado.

La estructura utilizada para unificar contexto es:

```text
Tema: {{Tema}}
Categoría: {{Categoria}}
Contenido: {{Contenido validado}}
```

**RAG no se utiliza únicamente como concepto teórico.**

Existe una recuperación real de conocimiento externo al prompt antes de cada generación mediante Claude.

---

# 7. Claude

Modelo utilizado:

`Claude Haiku 4.5`

Configuración principal:

`Max Tokens = 700`

El prompt combina dinámicamente:

```text
IDEA SEMILLA:
{{idea_semilla}}

CONTEXTO RAG:
{{contexto_agregado}}
```

Además, el prompt prohíbe inventar:

- estadísticas;
- cifras;
- estudios;
- fuentes;
- clientes;
- casos reales;
- resultados cuantitativos;
- afirmaciones verificables específicas no proporcionadas.

Esto funciona como control preventivo contra alucinaciones.

---

# 8. Human-in-the-Loop

Antes de la acción externa, el sistema crea una solicitud **Human-in-the-Loop**.

Opciones:

- Approve.
- Reject.

La solicitud conserva el Record ID del registro original mediante contexto.

Configuración:

- `processingType = time-sensitive`
- `timeout = 600 segundos`
- `defaultResponse = Reject`

El sistema no ejecuta una acción crítica sin una decisión humana.

---

# 9. Estructuras de datos

Airtable contiene tres tablas principales.

## 9.1. Control de Contenidos

Es la tabla central del pipeline.

Campos principales:

- Idea Semilla.
- Borrador IA.
- Estado.
- Aprobado.
- Motivo Rechazo.
- Canal.
- Fecha última ejecución.
- Fecha publicación.
- Resultado final.
- Última modificación.
- Indicador Error.
- Indicador Éxito.
- Relación con LOG de Errores.

Estados disponibles:

- Generando.
- En revisión.
- Aprobado.
- Rechazado.
- Publicado.
- Error.

El recorrido automatizado principal es:

`Generando → En revisión → Publicado / Rechazado / Error`

---

## 9.2. Base de Conocimiento RAG

Contiene conocimiento validado utilizado durante la generación.

Campos principales:

- Tema.
- Categoria.
- Contenido validado.
- Activo.
- Motivo Rechazo.
- Ultima actualización.

---

## 9.3. LOG de Errores

Registra incidencias técnicas.

Campos principales:

- Error ID.
- Fecha.
- Escenario.
- Módulo.
- Record ID.
- Tipo de error.
- Mensaje.
- Severidad.
- Resuelto.
- Contenido afectado.

---

# 10. Relación entre tablas

La relación física mediante Linked Record existe entre:

`Control de Contenidos ↔ LOG de Errores`

La Base de Conocimiento RAG mantiene una relación lógica con el pipeline mediante Make:

`Base de Conocimiento RAG → Make → Claude → Control de Contenidos`

El esquema fue analizado mediante **Omni AI**.

Omni AI permite identificar:

- Control de Contenidos como tabla central.
- LOG de Errores como tabla de incidencias.
- Base de Conocimiento RAG como fuente de contexto.
- Relación física entre Control de Contenidos y LOG de Errores.
- Relación lógica entre RAG y el pipeline mediante Make.

Evidencias:

- E16a.
- E16b.

---

# 11. JSON de transferencia

Make trabaja internamente mediante bundles y mapeos entre módulos.

Para documentar el contrato de datos de forma legible, los principales intercambios se representan mediante esquemas JSON normalizados.

Estos esquemas documentan los datos transferidos y no implican que cada módulo nativo de Make realice necesariamente una petición HTTP con ese JSON literal.

## 11.1. Entrada hacia Claude

```json
{
  "record_id": "{{record_id}}",
  "idea_semilla": "{{idea_semilla}}",
  "contexto_rag": "{{contexto_agregado}}"
}
```

## 11.2. Salida de Claude hacia Airtable

```json
{
  "record_id": "{{record_id}}",
  "borrador_ia": "{{resultado_claude}}",
  "estado": "En revisión",
  "fecha_ultima_ejecucion": "{{now}}"
}
```

## 11.3. Contexto HITL

```json
{
  "airtable_record_id": "{{record_id}}"
}
```

## 11.4. Respuesta HITL

```json
{
  "response_data": "aprobar | rechazar",
  "context": {
    "airtable_record_id": "recXXXXXXXXXXXX"
  }
}
```

## 11.5. Actualización por aprobación

```json
{
  "record_id": "{{context.airtable_record_id}}",
  "estado": "Publicado",
  "aprobado": true,
  "canal": "{{canal}}",
  "fecha_ultima_ejecucion": "{{now}}",
  "fecha_publicacion": "{{now}}",
  "resultado_final": "{{borrador_ia}}"
}
```

## 11.6. Actualización por rechazo

```json
{
  "record_id": "{{context.airtable_record_id}}",
  "estado": "Rechazado",
  "aprobado": false,
  "fecha_ultima_ejecucion": "{{now}}",
  "resultado_final": "Rechazado por validación humana"
}
```

## 11.7. Registro normalizado de error

```json
{
  "error_id": "ERR-[ORIGEN]-{{record_id}}",
  "fecha": "{{now}}",
  "escenario": "{{nombre_escenario}}",
  "modulo": "{{modulo_origen}}",
  "record_id": "{{record_id}}",
  "tipo_error": "{{error.type}}",
  "mensaje": "{{error.message}}",
  "severidad": "Alta",
  "resuelto": false,
  "contenido_afectado": "{{record_id}}"
}
```

---

# 12. Optimización de costos y recursos

La estrategia utiliza IA únicamente cuando aporta valor.

Make y Airtable resuelven operaciones determinísticas como:

- filtros;
- IDs;
- fechas;
- estados;
- persistencia;
- rutas.

Claude se utiliza para generación contextual.

## 12.1. Matriz de decisión

| Tipo de tarea | Herramienta | Estado |
|---|---|---|
| Filtros, IDs, fechas y estados | Make + Airtable | Implementado |
| Generación con contexto RAG | Claude Haiku 4.5 | Implementado |
| Tareas simples de texto | Modelo económico | Estrategia futura |
| Lectura densa o contexto extenso | Claude | Estrategia futura |
| Procesamiento masivo no urgente | Message Batches | Estrategia futura |
| Contexto repetitivo | Prompt Caching | Estrategia futura |

---

# 13. Message Batches

Message Batches se define como estrategia futura para generación masiva no urgente.

El procesamiento por lotes permite reducir el costo relativo al:

**50% del costo equivalente en tiempo real**

Modelo:

```text
Costo normal = C
Costo con Batches = 0,5 × C
Ahorro = 50%
```

Ejemplo conceptual:

- procesamiento normal de un volumen determinado = 100 unidades de costo;
- procesamiento equivalente mediante Message Batches = 50 unidades;
- ahorro relativo = 50 unidades;
- ahorro porcentual = 50%.

No se implementa actualmente dentro del pipeline HITL interactivo.

La decisión arquitectónica es:

`Pipeline interactivo con HITL → procesamiento en tiempo real`

`Generación masiva no urgente → Message Batches`

---

# 14. Prompt Caching

Prompt Caching se documenta como estrategia complementaria futura.

Su objetivo sería reducir el costo de lectura de instrucciones o contexto estático repetido.

No está implementado en el flujo actual.

---

# 15. Seguridad, privacidad y resiliencia

La arquitectura incorpora:

- minimización de datos;
- variables dinámicas;
- control de alucinaciones;
- Human-in-the-Loop;
- Error Handlers;
- LOG de Errores;
- Commit;
- prevención de loops.

---

# 16. Minimización de datos

Los módulos transfieren únicamente la información necesaria para ejecutar cada etapa.

Entre escenarios se prioriza el uso del Record ID de Airtable en lugar de replicar objetos completos.

Claude recibe:

- Idea Semilla;
- Contexto RAG;
- instrucciones necesarias para la generación.

Las credenciales se mantienen dentro de las conexiones de Make y no se exponen en documentación, diagramas ni video.

---

# 17. Variables dinámicas

La lógica principal utiliza variables provenientes de módulos anteriores.

Entre ellas:

- Idea Semilla.
- Record ID.
- Contexto RAG.
- Borrador IA.
- Respuesta HITL.
- Tipo de error.
- Mensaje de error.
- Fechas.
- Estados.
- Contenido afectado.

No se utilizan IDs de registros escritos manualmente para dirigir la lógica del pipeline.

---

# 18. Control de alucinaciones

El prompt restringe explícitamente la generación de información verificable no proporcionada.

Además, Claude no puede ejecutar por sí mismo la acción externa.

Se aplican dos controles consecutivos:

`Restricciones del prompt → Validación humana HITL`

---

# 19. Error Handlers

## 19.1. Error Handler de Claude

Ante un fallo de generación:

- el contenido pasa a Estado = Error;
- se registra el resultado del fallo;
- se conserva el Record ID;
- se registra la incidencia en LOG de Errores;
- se vincula con el contenido afectado;
- se genera una alerta administrativa;
- la ejecución finaliza mediante Commit.

El objetivo es evitar que un error de IA llegue al HITL como si existiera un borrador válido.

---

## 19.2. Error Handler de Airtable

Ante un fallo de persistencia:

- se registra el error técnico;
- se conserva el Record ID;
- se crea una incidencia en LOG de Errores;
- se genera una alerta administrativa;
- la ejecución finaliza mediante Commit.

---

## 19.3. Error Handler de Gmail

Ante un fallo de envío:

- Estado = Error;
- Resultado final = Error en envío Gmail;
- se registra una incidencia en LOG de Errores;
- se vincula el incidente con el contenido afectado;
- la ejecución finaliza mediante Commit.

No se intenta notificar el fallo mediante un segundo envío de Gmail utilizando el mismo servicio que acaba de fallar.

---

# 20. Commit

Commit permite cerrar de forma controlada una ejecución afectada por un error.

No significa que el escenario quede deshabilitado.

Su objetivo es impedir que una ejecución fallida continúe posteriormente por una ruta de éxito.

---

# 21. Evidencias reales de resiliencia

Durante el desarrollo se registró un error real:

`RuntimeError — [404] NOT_FOUND`

La incidencia quedó persistida en LOG de Errores.

También se realizó una prueba real de fallo en Gmail.

El sistema:

- cambió el Estado a Error;
- registró Error en envío Gmail;
- creó un identificador ERR-GMAIL;
- vinculó el error con el contenido afectado;
- cerró la ejecución mediante Commit.

---

# 22. Prevención de loops

La vista `Por Generar` impide reprocesar registros cuyo Estado ya cambió.

La prueba se realizó mediante dos ejecuciones consecutivas.

Primera ejecución:

- el registro fue detectado;
- el registro fue procesado.

Segunda ejecución:

Make devolvió:

`No bundles were generated by this operation`

Esto demuestra que el mismo registro no fue reprocesado.

Evidencias:

- E13.
- E14.

---

# 23. Validación de tipos de datos

El flujo distingue explícitamente los tipos principales utilizados.

- Aprobado: booleano.
- Estado: selección.
- Canal: selección.
- response_data: texto.
- Fechas: valores temporales.
- Record ID: identificador de Airtable.

Esto evita comparaciones ambiguas entre tipos de datos.

---

# 24. Pruebas realizadas

Se documentaron cinco pruebas principales y una prueba complementaria de resiliencia.

## Test 1 — Camino feliz con aprobación

Entrada:

Registro válido con Idea Semilla y Estado = Generando.

Recorrido:

`Generando → RAG → Claude → En revisión → HITL → Approve → Gmail → Publicado`

Resultado:

**Exitoso.**

Evidencias:

- E07.
- E08.
- E09.
- E10.

---

## Test 2 — Rechazo humano

Condición:

HITL = Reject.

Resultado:

- Estado = Rechazado.
- Aprobado = No.
- Gmail no se ejecuta.

Evidencias:

- E11.
- E12.

---

## Test 3 — Datos incompletos / camino infeliz

Condición:

- Estado = Generando.
- Idea Semilla vacía.

Resultado:

El registro queda excluido por la vista Por Generar antes de consumir procesamiento de IA.

Evidencias:

- E01.
- E02.

---

## Test 4 — Error real

Error:

`RuntimeError — [404] NOT_FOUND`

Resultado:

Incidencia registrada en LOG de Errores.

Evidencia:

- E06.

---

## Test 5 — Anti-loop

Primera ejecución:

El registro se procesa normalmente.

Segunda ejecución consecutiva:

`No bundles were generated by this operation`

Resultado:

**Exitoso. El registro no fue reprocesado.**

Evidencias:

- E13.
- E14.

---

## Prueba complementaria — Error de Gmail

Durante una prueba del canal de salida se produjo un fallo de envío.

Resultado:

- Estado = Error.
- Resultado final = Error en envío Gmail.
- Creación de incidencia en LOG de Errores.
- Identificador ERR-GMAIL.
- Cierre mediante Commit.

---

# 25. Dashboard Ejecutivo

El proyecto utiliza una Airtable Interface como Dashboard Ejecutivo interno.

Valores actuales documentados:

- Total de contenidos: 14.
- Publicados: 4.
- Rechazados: 3.
- En revisión: 3.
- Tasa de éxito: 28,6%.
- Tasa de error: 7,1%.

Evidencia:

- E15.

Los indicadores utilizados son:

- Indicador Éxito.
- Indicador Error.

Los valores son dinámicos y pueden cambiar con nuevas ejecuciones.

---

# 26. Shared View pública

La Shared View permite verificar registros y estados sin permisos de edición.

Enlace público:

https://airtable.com/appE2ti2m7OuckfQ0/shrkquKwRuhnPU9I1

Evidencia:

- E17.

La Shared View pública cumple una función diferente del Dashboard Ejecutivo interno:

- Dashboard Ejecutivo = visualización de KPIs.
- Shared View pública = acceso web de solo lectura a los registros.

---

# 27. Video demostrativo

Video:

https://drive.google.com/file/d/1JQm1HIwfARJNoBBY6tUheFoC8ZpGZBKX/view?usp=drive_link

El video muestra:

- entrada en Airtable;
- ejecución del Escenario 1;
- consulta RAG;
- generación mediante Claude;
- guardado del borrador;
- Estado En revisión;
- solicitud HITL;
- aprobación humana;
- recepción de la respuesta por el Escenario 2;
- Router;
- Gmail;
- actualización final en Airtable.

La prueba anti-loop se documenta de forma separada mediante E13 y E14.

Las credenciales no se muestran.

---

# 28. Instrucciones de ejecución

## Escenario 1 — Generación

1. Crear o seleccionar un registro en Control de Contenidos.
2. Completar Idea Semilla.
3. Establecer Estado = Generando.
4. Ejecutar `Entrega Final - Pipeline HITL` o esperar su programación.
5. Verificar la consulta a Base de Conocimiento RAG.
6. Verificar la generación mediante Claude.
7. Confirmar que Airtable cambie el registro a En revisión.
8. Verificar la creación de la solicitud HITL.

## Escenario 2 — Decisión humana

1. Ejecutar `Entrega Final - HITL Aprobación` en modo de escucha.
2. Responder la solicitud HITL con Approve o Reject.
3. Si se aprueba, verificar Gmail y Estado = Publicado.
4. Si se rechaza, verificar Estado = Rechazado y que Gmail no se ejecute.
5. Ante un fallo, verificar Estado = Error y la incidencia correspondiente en LOG de Errores.

**Importante:** el Escenario 2 debe estar escuchando antes de responder la solicitud HITL.

---

# 29. Evidencias verificables

Todas las evidencias se encuentran en:

[Carpeta de evidencias](./evidencias/)

## E01 — Registro con datos incompletos

Demuestra el camino infeliz con Estado = Generando e Idea Semilla incompleta.

## E02 — Vista Por Generar

Demuestra las condiciones:

- Estado = Generando.
- Idea Semilla no vacía.

## E03 — Trigger dinámico

Demuestra la configuración del trigger de Airtable dentro de Make.

## E04 — Escenario 1 completo

Vista general de:

`Entrega Final - Pipeline HITL`

## E05 — Error Handlers Escenario 1

Demuestra las rutas de contingencia configuradas para Claude y Airtable.

## E06 — Error real 404

Demuestra el registro de:

`RuntimeError — [404] NOT_FOUND`

## E07 — Ejecución exitosa

Demuestra generación y actualización exitosa del contenido.

## E08 — Solicitud HITL

Demuestra la pausa antes de la acción externa y la existencia de Approve / Reject.

## E09 — Aprobación

Demuestra la ejecución de la ruta Approve.

## E10 — Contenido publicado

Demuestra la actualización final del registro después de la aprobación.

## E11 — Rechazo HITL

Demuestra la ejecución de la ruta Reject.

## E12 — Contenido rechazado

Demuestra Estado = Rechazado y Aprobado = No.

## E13 — Anti-loop primera ejecución

Demuestra el procesamiento inicial de un registro válido.

## E14 — Anti-loop segunda ejecución

Demuestra:

`No bundles were generated by this operation`

## E15 — Dashboard Ejecutivo

Demuestra los KPIs internos del sistema.

## E16 1 / E16 2/ E16 3/ E16 4 — Omni AI

Demuestran el análisis del esquema de Airtable.

## E17 — Shared View pública

Demuestra la vista pública de solo lectura.

## E18 — Escenario 2 completo

Vista general de:

`Entrega Final - HITL Aprobación`

---

# 30. Archivos técnicos

## Blueprints

[Ver Blueprints](./blueprints/)

Incluyen:

- `Entrega Final - Pipeline HITL.blueprint.json`
- `Entrega Final - HITL Aprobación.blueprint.json`

Los Blueprints permiten revisar:

- módulos;
- filtros;
- mapeos;
- prompts;
- variables;
- Router;
- Error Handlers;
- Commit;
- rutas;
- configuración técnica.

---

## Diagramas oficiales

[Ver diagramas](./diagramas/)

La carpeta contiene los dos flujos oficiales utilizados en la documentación final:

- Escenario 1 — Entrega Final - Pipeline HITL.
- Escenario 2 — Entrega Final - HITL Aprobación.

---

## Documentación final

[Ver documentación](./documentación/)

La carpeta contiene el PDF final de la Entrega Final.

---

# 31. Matriz de cumplimiento de la rúbrica

| Criterio | Implementación | Evidencia verificable | Cómo verificar |
|---|---|---|---|
| Mapa de Arquitectura del Sistema | Dos escenarios Make con trigger, Claude, Airtable, HITL, Router, Gmail y rutas de error | Diagramas oficiales + E04 + E05 + E18 + Blueprints | Revisar `/diagramas`, `/blueprints` y evidencias |
| Manual Operativo de Estructuras de Datos | Control de Contenidos + Base RAG + LOG + JSON + análisis Omni AI | E16a / E16b + documentación | Revisar estructura de Airtable y esquemas JSON |
| Optimización de Costos y Recursos | Claude Haiku 4.5 + criterio de modelo económico + Message Batches + 50% de ahorro + Prompt Caching | Sección de costos de la documentación | Revisar PDF final |
| Seguridad, Privacidad y Resiliencia | Minimización + control de alucinaciones + HITL + Error Handlers + LOG + Commit + anti-loop | E05 + E06 + E08 + E13 + E14 + prueba Gmail | Revisar Blueprints y evidencias |
| Dashboard Ejecutivo | KPIs + Airtable Interface + Shared View pública | E15 + E17 + enlace público | Abrir Airtable y contrastar KPIs |

---

# 32. Verificación de requisitos técnicos

| Requisito | Estado |
|---|---|
| Orquestador Make | Implementado |
| Base Airtable | Implementado |
| IA Claude | Implementado |
| RAG | Implementado |
| Gmail | Implementado |
| HITL | Implementado |
| Router | Implementado |
| Error Handlers | Implementados |
| LOG de Errores | Implementado |
| Commit | Implementado |
| Variables dinámicas | Implementadas |
| Max Tokens = 700 | Configurado |
| Filtro anti-loop | Implementado y probado |
| Validación de tipos | Implementada |
| 5 pruebas | Documentadas |
| Camino infeliz | Probado |
| Blueprints | Incluidos |
| Diagramas PDF | Incluidos |
| Evidencias E01–E18 | Incluidas |
| Dashboard Ejecutivo | Implementado |
| Shared View pública | Operativa |
| Video demo | Incluido |

---

# 33. Estructura del repositorio

```text
/
├── README.md
├── blueprints/
│   ├── README.md
│   ├── Entrega Final - Pipeline HITL.blueprint.json
│   └── Entrega Final - HITL Aprobación.blueprint.json
├── diagramas/
│   ├── README.md
│   ├── Escenario 1 - Entrega Final 26-09-26.drawio pdf git hub.pdf
│   └── Escenario 2 - Entrega Final 26-09-26.drawio pdf git hub.pdf
├── documentación/
│   ├── README.md
│   └── PDF final de la entrega
└── evidencias/
    ├── README.md
    └── E01 ... E18
```

Cada carpeta contiene su propio README con información específica para facilitar la auditoría.

---

# 34. Alcance final

## Implementado

- Make.
- Airtable.
- Claude.
- RAG.
- Gmail.
- HITL.
- Router.
- Error Handlers.
- LOG de Errores.
- Commit.
- Variables dinámicas.
- Anti-loop.
- Cinco pruebas.
- Dashboard.
- Shared View.
- Blueprints.
- Diagramas.
- Evidencias.

## Estrategias futuras

- Message Batches.
- Prompt Caching.
- Modelos económicos para tareas simples.

## No implementado

- Publicación directa mediante API de LinkedIn.

LinkedIn se utiliza como destino editorial del contenido generado.

Gmail es el canal técnico de salida implementado.

---

# 35. Enlaces públicos

## Shared View pública de Airtable

https://airtable.com/appE2ti2m7OuckfQ0/shrkquKwRuhnPU9I1

## Video demostrativo

https://drive.google.com/file/d/1JQm1HIwfARJNoBBY6tUheFoC8ZpGZBKX/view?usp=drive_link

## Repositorio

https://github.com/julican95-hub/ecosistema-automatizacion-ia-julietacanziootero
