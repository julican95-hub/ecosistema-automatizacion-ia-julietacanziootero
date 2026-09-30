# Ecosistema de Automatización IA Autónomo para Negocios

## Entrega Final — Julieta Canzio Otero

Proyecto final de automatización end-to-end orientado a la generación, revisión y salida controlada de contenido destinado a LinkedIn.

La solución integra:

- Make como orquestador;
- Airtable como memoria y persistencia;
- Claude Haiku 4.5 como motor de IA;
- Base de Conocimiento RAG;
- Human-in-the-Loop (HITL);
- Router de decisión;
- Gmail como canal de salida implementado;
- Error Handlers;
- Commit;
- LOG de Errores;
- prevención de loops;
- Dashboard Ejecutivo;
- Shared View pública.

---

# 1. Objetivo del proyecto

El sistema automatiza un pipeline de generación de contenido profesional.

El proceso comienza con una Idea Semilla registrada en Airtable.

Make detecta únicamente registros elegibles, recupera contexto desde una Base de Conocimiento RAG, genera un borrador mediante Claude y guarda el resultado nuevamente en Airtable.

Antes de realizar una acción externa, el flujo crea una solicitud Human-in-the-Loop.

Una persona debe decidir entre:

- Approve
- Reject

La automatización continúa únicamente después de esa decisión.

LinkedIn es el destino editorial del contenido.

La implementación no publica directamente mediante la API de LinkedIn.

La salida técnica implementada en este proyecto es Gmail.

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

1. detectar contenido a generar;
2. guardar Idea Semilla;
3. guardar Record ID;
4. buscar contexto RAG;
5. unificar contexto;
6. recuperar variables;
7. generar borrador con Claude;
8. guardar borrador en Airtable;
9. cambiar Estado a En revisión;
10. solicitar aprobación humana mediante HITL.

También incluye Error Handlers para:

- errores de Claude;
- errores de Airtable;
- registro en LOG de Errores;
- notificación administrativa;
- Commit para cierre controlado.

Diagramas oficiales:

[Ver diagramas](./diagramas/)

Blueprint:

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

El error se registra en Airtable y en LOG de Errores antes del cierre mediante Commit.

Diagramas oficiales:

[Ver diagramas](./diagramas/)

Blueprint:

[Ver Blueprints](./blueprints/)

---

# 4. Trigger inteligente

El Escenario 1 utiliza una vista de Airtable denominada:

`Por Generar`

Solo ingresan registros que cumplen simultáneamente:

- Estado = Generando
- Idea Semilla no está vacía

Esto permite:

- evitar datos incompletos;
- reducir operaciones innecesarias;
- impedir reprocesamiento del mismo contenido.

---

# 5. Variables dinámicas

El flujo utiliza variables dinámicas provenientes de módulos anteriores.

Entre ellas:

- Idea Semilla;
- Record ID;
- contexto RAG;
- Borrador IA;
- respuesta HITL;
- Estado;
- fechas;
- tipo de error;
- mensaje de error;
- contenido afectado.

El Record ID permite mantener trazabilidad sobre el mismo registro durante toda la ejecución.

---

# 6. Base de Conocimiento RAG

La Base de Conocimiento RAG almacena información validada utilizada por Claude.

Incluye categorías como:

- Tono de marca;
- Audiencia;
- Reglas de contenido;
- Frases prohibidas;
- Servicios.

Make recupera únicamente registros activos mediante:

`{Activo}=1`

Luego agrega:

- Tema;
- Categoría;
- Contenido validado.

RAG no se utiliza únicamente como concepto teórico.

Existe una recuperación real de conocimiento antes de cada generación mediante Claude.

---

# 7. Claude

Modelo utilizado:

`Claude Haiku 4.5`

Configuración principal:

`Max Tokens = 700`

El prompt combina dinámicamente:

- Idea Semilla;
- Contexto RAG;
- instrucciones de generación.

Además, el prompt prohíbe inventar:

- estadísticas;
- cifras;
- estudios;
- fuentes;
- clientes;
- casos reales;
- resultados cuantitativos;
- afirmaciones verificables no proporcionadas.

Esto funciona como control preventivo contra alucinaciones.

---

# 8. Human-in-the-Loop

Antes de la acción externa, el sistema crea una solicitud HITL.

Opciones:

- Approve
- Reject

La solicitud conserva el Record ID del registro original.

Configuración:

- processingType: time-sensitive
- timeout: 600 segundos
- default response: Reject

El sistema no ejecuta una acción crítica sin decisión humana.

---

# 9. Estructuras de datos

Airtable contiene tres tablas principales.

## Control de Contenidos

Tabla central del pipeline.

Campos principales:

- Idea Semilla;
- Borrador IA;
- Estado;
- Aprobado;
- Motivo Rechazo;
- Canal;
- Fecha última ejecución;
- Fecha publicación;
- Resultado final;
- Última modificación;
- Indicador Error;
- Indicador Éxito;
- relación con LOG de Errores.

Estados disponibles:

- Generando;
- En revisión;
- Aprobado;
- Rechazado;
- Publicado;
- Error.

El recorrido automatizado principal es:

`Generando → En revisión → Publicado / Rechazado / Error`

---

## Base de Conocimiento RAG

Contiene el conocimiento validado utilizado durante la generación.

Campos principales:

- Tema;
- Categoria;
- Contenido validado;
- Activo;
- Motivo Rechazo;
- Ultima actualización.

---

## LOG de Errores

Registra incidencias técnicas.

Campos principales:

- Error ID;
- Fecha;
- Escenario;
- Módulo;
- Record ID;
- Tipo de error;
- Mensaje;
- Severidad;
- Resuelto;
- Contenido afectado.

---

# 10. Relación entre tablas

La relación física mediante Linked Record existe entre:

`Control de Contenidos ↔ LOG de Errores`

La Base de Conocimiento RAG mantiene una relación lógica con el pipeline mediante Make:

`RAG → Make → Claude → Control de Contenidos`

El esquema fue analizado mediante Omni AI.

Evidencias:

- E16a
- E16b

---

# 11. JSON de transferencia

Los intercambios principales se documentan mediante esquemas JSON normalizados.

## Entrada hacia Claude

```json
{
  "record_id": "{{record_id}}",
  "idea_semilla": "{{idea_semilla}}",
  "contexto_rag": "{{contexto_agregado}}"
}

Salida de Claude hacia Airtable
{
  "record_id": "{{record_id}}",
  "borrador_ia": "{{resultado_claude}}",
  "estado": "En revisión",
  "fecha_ultima_ejecucion": "{{now}}"
}

Contexto HITL
{
  "airtable_record_id": "{{record_id}}"
}

Respuesta HITL
{
  "response_data": "aprobar | rechazar",
  "context": {
    "airtable_record_id": "recXXXXXXXXXXXX"
  }
}

Actualización por aprobación
{
  "record_id": "{{context.airtable_record_id}}",
  "estado": "Publicado",
  "aprobado": true,
  "canal": "{{canal}}",
  "fecha_ultima_ejecucion": "{{now}}",
  "fecha_publicacion": "{{now}}",
  "resultado_final": "{{borrador_ia}}"
}

Actualización por rechazo
{
  "record_id": "{{context.airtable_record_id}}",
  "estado": "Rechazado",
  "aprobado": false,
  "fecha_ultima_ejecucion": "{{now}}",
  "resultado_final": "Rechazado por validación humana"
}

Registro normalizado de error
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

12. Optimización de costos y recursos
La estrategia utiliza IA únicamente cuando aporta valor.
Make y Airtable resuelven:
- filtros;
- IDs;
- fechas;
- estados;
- persistencia;
- rutas.
Claude se utiliza para generación contextual.
Matriz de decisión
Tipo de tarea	Herramienta	Estado
Filtros, IDs, fechas y estados	Make + Airtable	Implementado
Generación con contexto RAG	Claude Haiku 4.5	Implementado
Tareas simples de texto	Modelo económico	Estrategia futura
Procesamiento masivo no urgente	Message Batches	Estrategia futura
Contexto repetitivo	Prompt Caching	Estrategia futura


13. Message Batches
Message Batches se define como estrategia futura para generación masiva no urgente.
El procesamiento por lotes permite reducir el costo relativo al:
50% del costo equivalente en tiempo real
Modelo:
Costo normal = C
Costo con Batches = 0,5 × C
Ahorro = 50%
No se implementa actualmente dentro del pipeline HITL interactivo.
14. Prompt Caching
Prompt Caching se documenta como estrategia complementaria futura.
Su objetivo sería reducir el costo de lectura de instrucciones o contexto estático repetido.
No está implementado en el flujo actual.
15. Seguridad, privacidad y resiliencia
La arquitectura incorpora:
- minimización de datos;
- variables dinámicas;
- control de alucinaciones;
- Human-in-the-Loop;
- Error Handlers;
- LOG de Errores;
- Commit;
- prevención de loops.
16. Minimización de datos
Los módulos transfieren únicamente la información necesaria para ejecutar cada etapa.
Entre escenarios se prioriza el uso del Record ID de Airtable en lugar de replicar objetos completos.
Claude recibe:
- Idea Semilla;
- contexto RAG;
- instrucciones necesarias para la generación.
Las credenciales se mantienen dentro de las conexiones de Make y no se exponen en documentación ni video.
17. Control de alucinaciones
El prompt restringe explícitamente la generación de información verificable no proporcionada.
Además, el contenido no continúa automáticamente hacia una acción externa.
Se aplican dos controles consecutivos:
restricciones del prompt → validación humana HITL
18. Error Handlers
Claude
Ante un fallo de generación:
- el contenido pasa a Error;
- se registra la incidencia;
- se conserva el Record ID;
- se registra el error en LOG;
- se genera alerta administrativa;
- la ejecución finaliza mediante Commit.
Airtable
Ante un fallo de persistencia:
- se registra el error técnico;
- se conserva el Record ID;
- se crea incidencia en LOG;
- se genera alerta;
- la ejecución finaliza mediante Commit.
Gmail
Ante un fallo de envío:
- Estado = Error;
- Resultado final = Error en envío Gmail;
- se registra incidencia en LOG;
- la ejecución finaliza mediante Commit.
19. Commit
Commit permite cerrar de forma controlada una ejecución afectada por un error.
No significa que el escenario quede deshabilitado.
Su objetivo es evitar que una ejecución fallida continúe por una ruta de éxito después de registrar la contingencia.
20. Evidencias reales de resiliencia
Se registró un error real:
RuntimeError — [404] NOT_FOUND
La incidencia quedó persistida en LOG de Errores.
También se realizó una prueba real de fallo en Gmail.
El sistema:
- cambió el Estado a Error;
- registró Error en envío Gmail;
- creó un identificador ERR-GMAIL;
- vinculó el error con el contenido afectado.
21. Prevención de loops
La vista Por Generar impide reprocesar registros cuyo Estado ya cambió.
La prueba se realizó mediante dos ejecuciones consecutivas.
Primera ejecución:
el registro fue procesado.
Segunda ejecución:
Make devolvió:
No bundles were generated by this operation
Evidencias:
- E13
- E14
22. Validación de tipos de datos
El flujo distingue explícitamente los tipos principales utilizados.
- Aprobado: booleano.
- Estado: selección.
- response_data: texto.
- fechas: valores temporales.
- Record ID: identificador de Airtable.
Esto evita comparaciones ambiguas entre tipos de datos.
23. Pruebas realizadas
Se documentaron cinco pruebas principales y una prueba complementaria de resiliencia.
Test 1 — Camino feliz
Recorrido:
Generando → RAG → Claude → En revisión → HITL → Approve → Gmail → Publicado
Resultado:
Exitoso.
Evidencias:
- E07
- E08
- E09
- E10
Test 2 — Rechazo humano
HITL = Reject.
Resultado:
- Estado = Rechazado;
- Aprobado = No;
- Gmail no se ejecuta.
Evidencias:
- E11
- E12
Test 3 — Datos incompletos
Condición:
- Estado = Generando;
- Idea Semilla vacía.
Resultado:
el registro queda excluido por la vista Por Generar.
Evidencias:
- E01
- E02
Test 4 — Error real
Error:
RuntimeError — [404] NOT_FOUND
Resultado:
incidencia registrada en LOG de Errores.
Evidencia:
- E06
Test 5 — Anti-loop
Segunda ejecución consecutiva:
No bundles were generated by this operation
Evidencias:
- E13
- E14
Prueba complementaria — Error de Gmail
Resultado:
- Estado = Error;
- Resultado final = Error en envío Gmail;
- creación de registro en LOG de Errores;
- cierre mediante Commit.
24. Dashboard Ejecutivo
Valores actuales documentados:
- Total de contenidos: 14
- Publicados: 4
- Rechazados: 3
- En revisión: 3
- Tasa de éxito: 28,6%
- Tasa de error: 7,1%
Evidencia:
- E15
Los valores son dinámicos y pueden cambiar con nuevas ejecuciones.
25. Shared View pública
La Shared View permite verificar registros y estados sin permisos de edición.
Enlace:
https://airtable.com/appE2ti2m7OuckfQ0/shrkquKwRuhnPU9I1
Evidencia:
- E17
26. Video demostrativo
Video:
https://drive.google.com/file/d/1JQm1HIwfARJNoBBY6tUheFoC8ZpGZBKX/view?usp=drive_link
El video muestra:
- entrada en Airtable;
- Escenario 1;
- consulta RAG;
- generación mediante Claude;
- Estado En revisión;
- solicitud HITL;
- aprobación;
- Escenario 2;
- Router;
- Gmail;
- actualización final en Airtable.
La prueba anti-loop se documenta de forma separada mediante E13 y E14.
Las credenciales no se muestran.
27. Instrucciones de ejecución
Escenario 1
1. Crear o seleccionar un registro en Control de Contenidos.
2. Completar Idea Semilla.
3. Establecer Estado = Generando.
4. Ejecutar Entrega Final - Pipeline HITL.
5. Verificar consulta RAG.
6. Verificar generación con Claude.
7. Confirmar Estado = En revisión.
8. Verificar solicitud HITL.
Escenario 2
1. Ejecutar Entrega Final - HITL Aprobación en modo de escucha.
2. Responder HITL con Approve o Reject.
3. Si se aprueba, verificar Gmail y Estado = Publicado.
4. Si se rechaza, verificar Estado = Rechazado.
5. Ante un fallo, verificar Estado = Error y LOG de Errores.
28. Evidencias verificables
Todas las evidencias se encuentran en:
[Carpeta de evidencias](./evidencias/)
E01 — Registro con datos incompletos
Demuestra el camino infeliz con Idea Semilla incompleta.
E02 — Vista Por Generar
Demuestra las condiciones:
- Estado = Generando
- Idea Semilla no vacía
E03 — Trigger dinámico
Demuestra la configuración del trigger de Airtable en Make.
E04 — Escenario 1 completo
Vista general de Entrega Final - Pipeline HITL.
E05 — Error Handlers Escenario 1
Demuestra las rutas de contingencia de Claude y Airtable.
E06 — Error real 404
Demuestra el registro de:
RuntimeError — [404] NOT_FOUND
E07 — Ejecución exitosa
Demuestra generación y actualización exitosa del contenido.
E08 — Solicitud HITL
Demuestra la pausa antes de la acción externa.
E09 — Aprobación
Demuestra la ejecución de la ruta Approve.
E10 — Contenido publicado
Demuestra la actualización final del registro.
E11 — Rechazo HITL
Demuestra la ejecución de la ruta Reject.
E12 — Contenido rechazado
Demuestra Estado = Rechazado.
E13 — Anti-loop primera ejecución
Demuestra el procesamiento inicial.
E14 — Anti-loop segunda ejecución
Demuestra:
No bundles were generated by this operation
E15 — Dashboard Ejecutivo
Demuestra los KPIs internos del sistema.
E16a / E16b — Omni AI
Demuestran el análisis del esquema de Airtable.
E17 — Shared View pública
Demuestra la vista pública de solo lectura.
E18 — Escenario 2 completo
Vista general de Entrega Final - HITL Aprobación.
29. Archivos técnicos
Blueprints
[Ver Blueprints](./blueprints/)
Incluyen:
- Entrega Final - Pipeline HITL.blueprint.json
- Entrega Final - HITL Aprobación.blueprint.json
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
Diagramas oficiales
[Ver diagramas](./diagramas/)
La carpeta contiene los dos flujos oficiales utilizados en la documentación final.
Documentación final
[Ver documentación](./documentación/)
Contiene el PDF final de la Entrega Final.
30. Matriz de cumplimiento de la rúbrica
Criterio	Implementación	Evidencia verificable	Cómo verificar
Mapa de Arquitectura del Sistema	Dos escenarios Make con trigger, Claude, Airtable, HITL, Router, Gmail y rutas de error	Diagramas oficiales + E04 + E05 + E18 + Blueprints	Revisar /diagramas, /blueprints y evidencias
Manual Operativo de Estructuras de Datos	Control de Contenidos + Base RAG + LOG + JSON + análisis Omni AI	E16a / E16b + documentación	Revisar estructura de Airtable y esquemas JSON
Optimización de Costos y Recursos	Claude Haiku 4.5 + criterio de modelo económico + Message Batches + 50% de ahorro + Prompt Caching	Secciones de costos del PDF	Revisar documentación final
Seguridad, Privacidad y Resiliencia	Minimización + control de alucinaciones + HITL + Error Handlers + LOG + Commit + anti-loop	E05 + E06 + E08 + E13 + E14 + prueba Gmail	Revisar Blueprints y evidencias
Dashboard Ejecutivo	KPIs + Airtable Interface + Shared View pública	E15 + E17 + enlace público	Abrir Airtable y contrastar KPIs


31. Verificación de requisitos técnicos
Requisito	Estado
Orquestador Make	Implementado
Base Airtable	Implementado
IA Claude	Implementado
RAG	Implementado
Gmail	Implementado
HITL	Implementado
Router	Implementado
Error Handlers	Implementados
LOG de Errores	Implementado
Commit	Implementado
Variables dinámicas	Implementadas
Max Tokens = 700	Configurado
Filtro anti-loop	Implementado y probado
Validación de tipos	Implementada
5 pruebas	Documentadas
Camino infeliz	Probado
Blueprints	Incluidos
Diagramas PDF	Incluidos
Evidencias E01–E18	Incluidas
Dashboard Ejecutivo	Implementado
Shared View pública	Operativa
Video demo	Incluido


32. Estructura del repositorio
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

Cada carpeta contiene su propio README con información específica para facilitar la auditoría.
33. Alcance final
Implementado
- Make;
- Airtable;
- Claude;
- RAG;
- Gmail;
- HITL;
- Router;
- Error Handlers;
- LOG de Errores;
- Commit;
- variables dinámicas;
- anti-loop;
- cinco pruebas;
- Dashboard;
- Shared View;
- Blueprints;
- diagramas;
- evidencias.
Estrategias futuras
- Message Batches;
- Prompt Caching;
- modelos económicos para tareas simples.
No implementado
- publicación directa mediante API de LinkedIn.
LinkedIn se utiliza como destino editorial del contenido generado.
Gmail es el canal técnico de salida implementado.
34. Enlaces públicos
Shared View pública de Airtable
https://airtable.com/appE2ti2m7OuckfQ0/shrkquKwRuhnPU9I1
Video demostrativo
https://drive.google.com/file/d/1JQm1HIwfARJNoBBY6tUheFoC8ZpGZBKX/view?usp=drive_link
Repositorio
https://github.com/julican95-hub/ecosistema-automatizacion-ia-julietacanziootero
