## Ecosistema de Automatización IA Autónomo para Negocios

## Pipeline de generación de contenido con RAG, Claude y Human-in-the-Loop

**Autora:** Julieta Canzio Otero  
**Orquestador:** Make  
**Base de datos:** Airtable  
**Motor de IA:** Anthropic Claude  
**Validación humana:** Human-in-the-Loop  
**Canal de salida:** Gmail  

---

## 1. Descripción del proyecto

Este proyecto implementa un ecosistema de automatización para la generación, revisión y gestión controlada de contenido para LinkedIn.

El sistema parte de una **Idea Semilla almacenada en Airtable**. Make detecta los registros que cumplen las condiciones de entrada, consulta una **Base de Conocimiento RAG**, construye el contexto necesario y utiliza **Anthropic Claude** para generar un borrador.

El contenido generado no se envía automáticamente. Antes de ejecutar una acción final, el flujo crea una solicitud de **Human-in-the-Loop (HITL)** para que una persona pueda aprobar o rechazar el contenido.

Una vez recibida la decisión humana, un segundo escenario de Make procesa la respuesta:

- Si el contenido es aprobado, se envía mediante Gmail y el registro se actualiza como `Publicado`.
- Si el contenido es rechazado, se actualiza como `Rechazado` y Gmail no se ejecuta.

La solución incorpora además:

- Base de Conocimiento RAG.
- Variables dinámicas.
- Prevención de reprocesamiento.
- Error Handlers.
- Registro de errores.
- Human-in-the-Loop.
- Dashboard ejecutivo.
- Shared View pública.
- Evidencias de pruebas.
- Blueprints exportables de Make.

---

# 2. Arquitectura general

El ecosistema está dividido en dos escenarios principales.

## Escenario 1 — Generación y solicitud de aprobación

Flujo principal:

`Airtable → Tools → RAG → Claude → Airtable → HITL`

Proceso:

1. Airtable detecta un contenido a generar.
2. Make almacena dinámicamente la Idea Semilla y el Record ID.
3. Airtable consulta la Base de Conocimiento RAG.
4. Make unifica el contexto recuperado.
5. Claude genera el borrador de LinkedIn.
6. Airtable guarda el borrador y cambia el estado a `En revisión`.
7. HITL crea una solicitud de aprobación humana.

### Diagrama de arquitectura

[Ver carpeta de diagramas](./diagramas/)

[Escenario 1 — Arquitectura](./diagramas/Escenario_1_FINAL_PROFESIONAL.pdf)

---

## Escenario 2 — Resolución de la decisión humana

Flujo principal:

`HITL → Airtable → Router`

### Si la respuesta es Aprobar

`Gmail → Airtable → Publicado`

### Si la respuesta es Rechazar

`Airtable → Rechazado`

La arquitectura garantiza que la acción final dependa de una decisión humana explícita.

[Escenario 2 — Arquitectura](./diagramas/Escenario_2_FINAL_PROFESIONAL.pdf)

---

# 3. Estructura de datos

Airtable funciona como sistema de persistencia y control del ecosistema.

La base contiene tres tablas principales.

## Control de Contenidos

Tabla central del pipeline.

Campos relevantes:

- Idea Semilla
- Borrador IA
- Estado
- Aprobado
- Motivo Rechazo
- Canal
- Fecha última ejecución
- Fecha publicación
- Resultado final
- Última modificación
- Indicador Error
- Indicador Éxito
- relación con LOG de Errores

Estados utilizados:

`Generando → En revisión → Publicado / Rechazado / Error`

---

## Base de Conocimiento RAG

Contiene información validada utilizada como contexto para Claude.

Incluye:

- tono;
- audiencia;
- reglas de contenido;
- lineamientos de marca;
- servicios o focos temáticos;
- restricciones;
- frases prohibidas.

La relación entre esta tabla y el contenido es **lógica y funcional**.

Make consulta la Base de Conocimiento RAG durante la ejecución y agrega dinámicamente la información antes de enviarla a Claude.

No existe una relación Linked Record física entre RAG y Control de Contenidos.

---

## LOG de Errores

Registra los incidentes producidos durante las ejecuciones.

Campos principales:

- Error ID
- Fecha
- Escenario
- Módulo
- Record ID
- Tipo de error
- Mensaje
- Severidad
- Resuelto
- Contenido afectado

Existe una relación estructural:

`Control de Contenidos ↔ LOG de Errores`

Esto permite relacionar cada incidente con el contenido que lo produjo.

---

# 4. Transferencia de datos y variables dinámicas

El sistema utiliza variables dinámicas para evitar depender de valores cargados manualmente.

Entre los datos transferidos se encuentran:

- Record ID de Airtable.
- Idea Semilla.
- Contexto RAG.
- Borrador generado.
- Respuesta HITL.
- Estado del contenido.
- Resultado final.

Ejemplo conceptual del contexto transferido a HITL:

```json
{
  "airtable_record_id": "{{Record ID dinámico}}"
}
```

Ejemplo conceptual de una respuesta aprobada:

```json
{
  "response_data": "aprobar",
  "context": {
    "airtable_record_id": "ID del registro"
  }
}
```

Ejemplo conceptual de una respuesta rechazada:

```json
{
  "response_data": "rechazar",
  "context": {
    "airtable_record_id": "ID del registro"
  }
}
```

De esta manera, el segundo escenario puede recuperar exactamente el registro asociado con la solicitud humana.

---

# 5. Optimización de costos y recursos

La estrategia de optimización se basa en utilizar IA solamente cuando existe una necesidad real de interpretación o generación.

Operaciones como filtros, almacenamiento de IDs, fechas, cambios de estado y registro de errores son realizadas directamente mediante Make y Airtable.

| Tipo de tarea | Herramienta / estrategia | Decisión |
|---|---|---|
| Filtros, IDs, fechas y estados | Make + Airtable | No requieren LLM |
| Tareas simples o repetitivas | Modelo económico como GPT-4o-mini | Estrategia prevista para tareas de baja complejidad |
| Generación con contexto RAG | Claude Haiku 4.5 | Implementado |
| Lectura/contexto extenso | Claude | Adecuado para escenarios de mayor densidad contextual |
| Procesamiento masivo no urgente | Message Batches | Estrategia de escalabilidad |
| Contexto estático repetitivo | Prompt Caching | Estrategia de escalabilidad |

## Decisión aplicada al proyecto

En el flujo actual, **Claude Haiku 4.5** se utiliza exclusivamente para la generación del borrador.

El modelo recibe dinámicamente:

- Idea Semilla.
- Contexto recuperado desde RAG.
- Reglas de generación.

El módulo posee además un límite aproximado de **700 tokens de salida**, evitando generaciones innecesariamente extensas.

## Message Batches

Para un escenario futuro con cientos o miles de generaciones no urgentes, podría utilizarse Message Batches para realizar procesamiento asíncrono a menor costo.

No se implementa actualmente porque el flujo es interactivo y depende de una instancia de aprobación humana.

## Prompt Caching

Si la Base de Conocimiento RAG aumentara considerablemente y se reutilizara el mismo contexto estático en muchas solicitudes, Prompt Caching podría evitar el reprocesamiento repetitivo de ese contenido.

Se documenta como estrategia futura de escalabilidad y no como una funcionalidad implementada actualmente.

---

# 6. Seguridad, privacidad y resiliencia

## Minimización de datos

El flujo procesa solamente los datos necesarios para identificar, generar y gestionar cada contenido.

Las credenciales y API Keys no se encuentran expuestas en:

- el repositorio;
- los diagramas;
- las evidencias;
- el video demostrativo.

Las conexiones se administran desde Make.

---

## Human-in-the-Loop

La principal medida de control antes de una acción crítica es el HITL.

Claude puede generar el contenido, pero el sistema **no ejecuta directamente la acción final**.

Primero se crea una solicitud de aprobación donde una persona debe seleccionar:

- `Approve`
- `Reject`

Solo después de esta decisión continúa el proceso.

---

## Manejo de errores

Se implementaron Error Handlers en puntos críticos.

### Error de Claude

`Claude → Registrar error → LOG → Notificación Gmail → Commit`

### Error al guardar en Airtable

`Airtable → LOG → Notificación Gmail → Commit`

### Error de Gmail

`Gmail → Registrar error de envío → LOG → Commit`

No se utiliza Gmail para intentar enviar una segunda notificación cuando el propio servicio Gmail es el componente que produjo el error.

Los nodos Commit permiten finalizar la ejecución afectada de manera controlada después de registrar el incidente.

---

# 7. Prevención de loops y reprocesamiento

El trigger principal utiliza la vista:

`Por Generar`

Sus condiciones son:

- `Estado = Generando`
- `Idea Semilla` no está vacía.

Cuando el registro avanza en el proceso, su estado cambia y deja de cumplir la condición de entrada.

La prevención de loops fue validada mediante dos ejecuciones consecutivas.

En la primera ejecución, el registro fue procesado normalmente.

En la segunda ejecución, Make devolvió:

`No bundles were generated by this operation`

Esto demuestra que el mismo registro no volvió a ser procesado.

Las evidencias correspondientes son:

- `E13_AntiLoop_Primera_Ejecucion`
- `E14_AntiLoop_Segunda_Ejecucion_Sin_Bundles`

---

# 8. Pruebas realizadas

Se realizaron cinco pruebas funcionales principales.

## Test 1 — Camino feliz

Registro válido en estado `Generando`.

Resultado:

`Generación → HITL → Approve → Gmail → Publicado`

**Resultado: exitoso.**

---

## Test 2 — Rechazo humano

Se seleccionó `Reject` en HITL.

Resultado:

- rama Reject ejecutada;
- Gmail no ejecutado;
- registro actualizado como `Rechazado`.

**Resultado: exitoso.**

---

## Test 3 — Datos incompletos

Se utilizó un registro con:

- Estado = `Generando`
- Idea Semilla vacía.

El filtro `Por Generar` impidió que ingresara al escenario.

**Resultado: exitoso.**

---

## Test 4 — Manejo de error

Durante las pruebas se produjo un error real:

`404 — NOT_FOUND`

El incidente quedó registrado en la tabla `LOG de Errores`, incluyendo información del error y el contenido afectado.

**Resultado: registro de incidente verificado.**

---

## Test 5 — Anti-loop

Se realizaron dos ejecuciones consecutivas sin volver a modificar el registro.

La segunda ejecución devolvió:

`No bundles were generated by this operation`

**Resultado: reprocesamiento evitado.**

---

# 9. Dashboard Ejecutivo

El proyecto cuenta con un Dashboard Ejecutivo en Airtable.

Indicadores registrados durante la documentación:

- Total de contenidos: **13**
- Publicados: **3**
- Rechazados: **3**
- En revisión: **3**
- Tasa de éxito: **23,1 %**
- Tasa de error: **7,7 %**

También se incluye una visualización de distribución de contenidos por estado.

---

# 10. Dashboard / Shared View pública

La información de monitoreo puede consultarse mediante una vista pública de Airtable en modo lectura.

### Acceso público

https://airtable.com/appE2ti2m7OuckfQ0/shrkquKwRuhnPU9I1

Esta vista permite inspeccionar el estado de los registros sin otorgar acceso de edición a la base.

---

# 11. Video demostrativo

El video demuestra el funcionamiento end-to-end del ecosistema.

Incluye:

- registro inicial en Airtable;
- trigger del Escenario 1;
- recuperación del contexto RAG;
- generación del borrador con Claude;
- actualización a `En revisión`;
- creación de solicitud HITL;
- aprobación humana;
- procesamiento de la respuesta en el Escenario 2;
- Router de decisión;
- envío mediante Gmail;
- actualización a `Publicado`;
- visualización del sistema de monitoreo.

La prueba específica de prevención de loops se encuentra documentada mediante evidencias independientes dentro del repositorio.

### Video de demostración

[▶ Ver video demostrativo en Google Drive](https://drive.google.com/file/d/1JQm1HIwfARJNoBBY6tUheFoC8ZpGZBKX/view?usp=drive_link)

---

# 12. Evidencias

Las capturas utilizadas para demostrar las pruebas y configuraciones se encuentran en:

### [Ver carpeta `/evidencias`](./evidencias/)

Incluyen evidencia de:

- filtro inteligente del trigger;
- datos incompletos;
- ejecución dinámica;
- Escenario 1;
- Error Handlers;
- error real 404;
- generación exitosa;
- solicitud HITL;
- aprobación;
- publicación;
- rechazo;
- prevención de loops;
- Dashboard;
- estructura relacional;
- Shared View pública;
- Escenario 2.

Los archivos están identificados mediante nomenclatura correlativa `E01` a `E18`.

---

# 13. Blueprints de Make

Los escenarios exportados desde Make se encuentran disponibles en:

### [Ver carpeta `/blueprints`](./blueprints/)

Archivos:

- `Entrega Final - Pipeline HITL.blueprint.json`
- `Entrega Final - HITL Aprobación.blueprint.json`

Estos archivos permiten inspeccionar e importar la configuración técnica de ambos escenarios.

---

# 14. Diagramas de arquitectura

Los diagramas exportados en PDF están disponibles en:

### [Ver carpeta `/diagramas`](./diagramas/)

Incluyen:

- Escenario 1 — Generación, RAG y HITL.
- Escenario 2 — Resolución HITL y salida final.

---

# 15. Documentación final

La documentación completa del proyecto incluye:

- descripción del caso de uso;
- arquitectura;
- estructuras de datos;
- esquemas JSON;
- optimización de costos;
- seguridad;
- Error Handlers;
- HITL;
- prevención de loops;
- pruebas;
- Dashboard;
- evidencias;
- enlaces públicos.

### Documento final

[Ver documentación final en PDF](./documentacion/Entrega_Final_Ecosistema_Automatizacion_IA.pdf)

---

# 16. Estructura del repositorio

```text
ecosistema-automatizacion-ia-julietacanziootero/
│
├── README.md
│
├── blueprints/
│   ├── Entrega Final - Pipeline HITL.blueprint.json
│   └── Entrega Final - HITL Aprobación.blueprint.json
│
├── diagramas/
│   ├── Escenario_1_FINAL_PROFESIONAL.pdf
│   └── Escenario_2_FINAL_PROFESIONAL.pdf
│
├── evidencias/
│   ├── E01...
│   ├── E02...
│   ├── ...
│   └── E18...
│
└── documentacion/
    └── Entrega_Final_Ecosistema_Automatizacion_IA.pdf
```

---

# 17. Entregables

| Entregable | Ubicación |
|---|---|
| Repositorio GitHub | Este repositorio |
| Arquitectura Escenario 1 | `/diagramas` |
| Arquitectura Escenario 2 | `/diagramas` |
| Blueprint Escenario 1 | `/blueprints` |
| Blueprint Escenario 2 | `/blueprints` |
| Evidencias | `/evidencias` |
| Documento final | `/documentacion` |
| Base pública Airtable | Link incluido en este README |
| Dashboard | Airtable + evidencia E15 |
| Video demostrativo | Google Drive |

---

# 18. Correspondencia con los criterios de evaluación

## 1. Mapa de Arquitectura

Se documentan ambos escenarios con:

- triggers;
- módulos;
- RAG;
- Claude;
- HITL;
- Router;
- Gmail;
- rutas de error;
- capa de datos y monitoreo.

**Evidencia:** carpeta `/diagramas`.

## 2. Estructuras de datos documentadas

Se documentan:

- Control de Contenidos;
- Base de Conocimiento RAG;
- LOG de Errores;
- relación estructural Control ↔ LOG;
- integración lógica con RAG;
- estructuras JSON de transferencia.

## 3. Optimización de costos y recursos

Se incluye:

- utilización de herramientas sin LLM para tareas mecánicas;
- Claude Haiku 4.5 para generación contextual;
- límite de tokens;
- criterio de uso de modelos económicos;
- estrategia Message Batches para procesamiento masivo;
- estrategia Prompt Caching para contexto repetitivo.

## 4. Seguridad, privacidad y resiliencia

Se implementan y documentan:

- minimización de datos;
- ocultamiento de credenciales;
- Human-in-the-Loop;
- Error Handlers;
- LOG de Errores;
- Commit;
- prevención de loops.

## 5. Dashboard de Control Ejecutivo

Se incluye:

- Dashboard Ejecutivo;
- KPIs;
- indicadores de éxito y error;
- distribución de estados;
- Shared View pública.

---

# 19. Tecnologías utilizadas

- **Make**
- **Airtable**
- **Anthropic Claude**
- **Human-in-the-Loop**
- **Gmail**
- **Google Drive**
- **GitHub**

---

# 20. Resultado final

El proyecto implementa un flujo automatizado de extremo a extremo en el que la IA participa como motor de generación, mientras que Make controla la orquestación, Airtable mantiene la trazabilidad y el estado del sistema, y HITL conserva la intervención humana antes de ejecutar una acción crítica.

La arquitectura incorpora además mecanismos de resiliencia, registro de errores, prevención de reprocesamiento, optimización de recursos y monitoreo mediante indicadores ejecutivos.
