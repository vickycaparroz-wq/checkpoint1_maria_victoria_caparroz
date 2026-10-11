# IT Support Triage Agent — Checkpoint 1

### Agente de Inteligencia Artificial para Triaje y Registro Automatizado de Tickets IT

**Coderhouse | Proyecto Integrador — Módulo 1: Configuración e Interfaces Agénticas**

**Autora:** María Victoria Caparroz  
**Estado:** Checkpoint 1 completado y validado funcionalmente  
**Tecnologías:** n8n · Groq · GPT-OSS 120B · Google Sheets · Gmail

---

## 1. Descripción del proyecto

**IT Support Triage Agent** es un agente de inteligencia artificial desarrollado en n8n para automatizar el proceso inicial de recepción, clasificación y registro de incidentes de soporte técnico corporativo.

El agente recibe solicitudes en lenguaje natural, identifica el problema principal, determina su categoría y prioridad, evalúa el impacto operativo y recomienda una acción inicial segura.

Cuando se cumplen las condiciones establecidas, utiliza autónomamente una herramienta de Google Sheets para registrar el incidente y permitir su seguimiento.

El workflow también incorpora un sistema de observabilidad mediante Gmail, que genera un reporte automático de cada ejecución con los resultados del agente y las herramientas utilizadas.

Este desarrollo constituye la primera versión de un **Proyecto Integrador evolutivo**, diseñado para incorporar progresivamente arquitecturas multiagente, memoria, integraciones, RAG y otras capacidades.

## 2. Objetivos

- Implementar un agente de IA mediante el nodo AI Agent de n8n.
- Configurar un modelo de lenguaje conectado de forma nativa.
- Diseñar un System Message profesional, modular y con restricciones operativas.
- Implementar guardrails para controlar el número máximo de iteraciones.
- Integrar una herramienta nativa de productividad que pueda ser activada autónomamente.
- Diferenciar la automatización determinista de la toma de decisiones probabilística.
- Registrar tickets estructurados en Google Sheets.
- Incorporar observabilidad y supervisión humana mediante Gmail.
- Validar el funcionamiento del workflow con una ejecución real.

## 3. Arquitectura del workflow

```text
                  ┌──────────────────────────┐
                  │       Chat Trigger       │
                  │   Recepción del mensaje  │
                  └────────────┬─────────────┘
                               │
                               ▼
                  ┌──────────────────────────┐
                  │     AI Agent             │
                  │   IT Support Triage      │
                  │                          │
                  │   Max Iterations: 7      │
                  │   System Message         │
                  │   Guardrails             │
                  └────────────┬─────────────┘
                               │
             ┌─────────────────┴─────────────────┐
             │                                   │
             ▼                                   ▼
    ┌─────────────────┐               ┌──────────────────┐
    │ Groq Chat Model │               │ Google Sheets    │
    │ GPT-OSS 120B    │               │ Tool             │
    │                 │               │ Append Row       │
    └─────────────────┘               └──────────────────┘

                  AI Agent
                      │
                      ▼
             ┌─────────────────────┐
             │ Gmail               │
             │ Observabilidad      │
             │ Reporte automático  │
             └─────────────────────┘
```

### Arquitectura híbrida

El workflow combina dos tipos de procesamiento:

**Automatización determinista:** el Chat Trigger recibe la solicitud y el nodo Gmail genera el reporte de observabilidad al finalizar la ejecución del agente.

**Autonomía probabilística:** el AI Agent interpreta el mensaje, clasifica el incidente, evalúa su impacto y decide si debe activar la herramienta de Google Sheets.

El modelo de lenguaje y la herramienta están conectados lateralmente al AI Agent mediante los puertos `ai_languageModel` y `ai_tool`.

Esta arquitectura permite separar las acciones lineales del proceso de las decisiones autónomas del agente.

## 4. Tecnologías utilizadas

| Tecnología | Función |
|---|---|
| n8n | Orquestación del workflow |
| AI Agent (Tools Agent) | Motor de razonamiento y selección de herramientas |
| Groq | Proveedor de inferencia del modelo |
| GPT-OSS 120B | Interpretación y clasificación de incidentes |
| Google Sheets | Almacenamiento estructurado de tickets |
| Gmail | Notificaciones y observabilidad |
| GitHub | Versionado y entrega del proyecto |

## 5. Justificación del uso de Groq

Para este checkpoint se utilizó **Groq Chat Model** con el modelo `openai/gpt-oss-120b`, como alternativa a la API comercial de OpenAI.

La decisión responde a criterios académicos, técnicos y económicos.

### Contexto académico — Coderhouse

Aunque la consigna escrita recomienda preferentemente OpenAI Chat Model o Anthropic Chat Model, durante la cursada la docente utilizó Groq Chat Model en su workflow de referencia.

Por este motivo, Groq se adoptó como una alternativa utilizada y aceptada en la práctica de clase para desarrollar el ejercicio, sin que esto implique afirmar una autorización institucional general no documentada.

### Motivos de la elección

- **Costo:** la API de OpenAI funciona con facturación por consumo, mientras que Groq ofrece una modalidad gratuita sujeta a límites de uso.
- **Compatibilidad:** Groq dispone de un nodo nativo en n8n compatible con AI Agent.
- **Capacidad técnica:** el modelo permite clasificación de incidentes, generación de respuestas estructuradas y llamadas a herramientas.
- **Continuidad:** la integración puede utilizarse como base para los próximos módulos del proyecto.

**Aclaración técnica:** Groq es el proveedor de inferencia y `openai/gpt-oss-120b` es el modelo utilizado. No se está utilizando la API comercial de OpenAI.

Esta elección permitió completar y validar el checkpoint sin contratar una API paga de OpenAI.

## 6. Funcionalidades del agente

El agente realiza las siguientes tareas:

1. Recibe solicitudes de soporte técnico en lenguaje natural.
2. Identifica el problema principal.
3. Clasifica el incidente por categoría.
4. Determina su prioridad según el impacto operativo.
5. Evalúa si el usuario puede continuar trabajando.
6. Recomienda una acción inicial segura.
7. Determina si corresponde intervención humana.
8. Verifica si existe información suficiente para registrar el ticket.
9. Ejecuta autónomamente Google Sheets Tool cuando corresponde.
10. Genera un resultado estructurado para su supervisión.

### Categorías de incidentes

- Acceso
- Hardware
- Red
- Software
- Automatización
- Otro

### Niveles de prioridad

| Prioridad | Criterio |
|---|---|
| Urgente | Incidentes graves de seguridad, caída de sistemas críticos, pérdida importante de datos o interrupción generalizada |
| Alta | Bloqueo de funciones laborales importantes o impacto operativo significativo |
| Media | Problemas que afectan el trabajo, pero permiten continuar mediante una alternativa temporal |
| Baja | Problemas menores o consultas técnicas sin impacto operativo inmediato |

## 7. System Message y guardrails

El AI Agent utiliza un System Message modular que define su comportamiento operativo.

### Estructura del System Message

**Rol y ámbito:** agente corporativo de triaje IT, limitado a incidentes técnicos internos.

**Objetivo operativo:** transformar solicitudes desestructuradas en información útil para el seguimiento de tickets.

**Reglas de herramientas:** condiciones para activar Google Sheets, datos mínimos obligatorios y prevención de registros duplicados.

**Escalamiento:** criterios para solicitar intervención humana ante incidentes críticos, acciones sensibles o situaciones que excedan las capacidades del agente.

**Estilo y respuesta:** español profesional, formato estructurado y exclusión de variantes artificiales de lenguaje inclusivo.

### Guardrails configurados

| Control | Configuración |
|---|---|
| Max Iterations | 7 |
| Return Intermediate Steps | Habilitado |
| Acciones destructivas | Prohibidas |
| Modificación de sistemas productivos | Prohibida |
| Invención de información | Prohibida |
| Solicitud de credenciales sensibles | Prohibida |
| Escalamiento humano | Definido por criterios operativos |
| Registro duplicado | Restringido por instrucciones |
| Confirmación de registro | Solo después del éxito de la herramienta |

El límite de siete iteraciones restringe la cantidad máxima de ciclos del agente, reduciendo el riesgo de ejecuciones excesivas y consumo innecesario de recursos.

Estas restricciones son instrucciones operativas del agente y no sustituyen los controles de acceso, permisos y seguridad de las integraciones.

## 8. Google Sheets Tool — Registro de tickets

Se integró una herramienta nativa de Google Sheets mediante la operación `Append Row`.

La herramienta está conectada lateralmente al AI Agent y dispone de una descripción semántica que explica cuándo debe utilizarse, qué información necesita y qué acciones están prohibidas.

### Condiciones de activación

El agente puede registrar un ticket cuando:

- Existe un incidente técnico real.
- El usuario proporciona un medio de contacto.
- El problema está suficientemente descrito.
- Es posible determinar una categoría y una prioridad.
- El incidente requiere seguimiento operativo.

Si falta información indispensable, el agente debe solicitar una aclaración antes de registrar.

### Estructura de Google Sheets

| Campo | Descripción |
|---|---|
| `fecha_hora` | Fecha y hora del registro |
| `contacto` | Medio de contacto del usuario |
| `descripcion_problema` | Descripción del incidente |
| `categoria` | Clasificación técnica |
| `prioridad` | Nivel de prioridad |
| `impacto` | Consecuencias operativas |
| `puede_continuar_trabajando` | Sí, No o No determinado |
| `recomendacion` | Próxima acción sugerida |
| `requiere_escalamiento` | Sí o No |

Los campos de clasificación y análisis utilizan expresiones `$fromAI()`, que permiten al agente proporcionar valores estructurados a la herramienta.

La fecha y hora se generan automáticamente mediante `$now.toISO()`.

La herramienta se limita a incorporar registros y no modifica ni elimina tickets existentes.

## 9. Observabilidad mediante Gmail

El workflow incluye un nodo final de Gmail que envía automáticamente un reporte de observabilidad al finalizar la ejecución del AI Agent.

### Información registrada

- Fecha y hora.
- Identificador de ejecución.
- Mensaje original recibido.
- Categoría y prioridad asignadas.
- Necesidad de escalamiento humano.
- Herramientas invocadas.
- Respuesta final del agente.
- Parámetros enviados a las herramientas.
- Resultados observables de las herramientas.

Para obtener esta información se habilitó la opción `Return Intermediate Steps` del AI Agent.

El reporte permite auditar las acciones observables del agente sin exponer su razonamiento interno.

## 10. Prueba funcional y validación

Se realizó una ejecución manual desde el Chat Trigger para comprobar el funcionamiento integral del workflow.

### Caso de prueba

Una usuaria reportó que su notebook no encendía, que ya había revisado el cargador y que no disponía de otro equipo para continuar trabajando.

### Resultado del agente

| Campo | Resultado |
|---|---|
| Categoría | Hardware |
| Prioridad | Alta |
| Puede continuar trabajando | No |
| Requiere escalamiento | Sí |
| Herramienta utilizada | Google Sheets Tool |
| Estado del registro | Registrado |

### Validaciones realizadas

- Recepción correcta del mensaje.
- Interpretación del incidente.
- Clasificación y priorización.
- Decisión autónoma de ejecutar la herramienta.
- Registro exitoso del ticket en Google Sheets.
- Generación de la respuesta estructurada.
- Envío exitoso del correo de observabilidad.
- Verificación de los parámetros y resultados de la herramienta.

**Ejecución de referencia:** `Execution ID 405`  
**Fecha de prueba:** 11 de octubre de 2026

La prueba confirmó el funcionamiento completo del recorrido principal del workflow.

## 11. Instalación y configuración

Para reutilizar este proyecto en otra instancia de n8n:

1. Importar el archivo `checkpoint1_maria_victoria_caparroz.json`.
2. Configurar las credenciales de Groq.
3. Seleccionar el modelo `openai/gpt-oss-120b`.
4. Configurar las credenciales OAuth2 de Google Sheets.
5. Crear o seleccionar una hoja con las nueve columnas requeridas.
6. Actualizar la referencia al documento y a la pestaña correspondiente.
7. Configurar las credenciales OAuth2 de Gmail.
8. Establecer el correo destinatario del reporte.
9. Verificar el System Message y el límite de siete iteraciones.
10. Ejecutar una prueba manual desde el Chat Trigger.
11. Confirmar que el ticket se registra y que el reporte se recibe correctamente.

### Consideraciones de seguridad

El archivo JSON exportado no incluye las claves secretas de las credenciales configuradas en n8n.

Sin embargo, antes de publicar el workflow deben revisarse y eliminarse identificadores privados, direcciones de correo personales y referencias a recursos internos que no sean necesarios para la entrega.

Cada usuario debe configurar sus propias credenciales y recursos antes de ejecutar el workflow importado.

## 12. Evolución del Proyecto Integrador

Este checkpoint constituye la base de un proyecto que continuará desarrollándose durante el curso.

| Módulo | Evolución prevista |
|---|---|
| M1 | Agente base, System Message, herramientas y observabilidad |
| M2 | Arquitectura multiagente con Manager y Workers |
| M3 | Memoria y contexto por sesión |
| M4 | Integraciones con CRM, calendario y Workspace |
| M5 | RAG y recuperación documental |
| M6 | Procesamiento y generación de voz |
| M7–M11 | Ampliaciones progresivas hasta el Proyecto Final Integrador |

La estrategia consiste en reutilizar y extender el workflow existente, conservando su arquitectura y agregando capacidades de forma modular.

## 13. Entregable

**Archivo:** `checkpoint1_maria_victoria_caparroz.json`

**Formato:** JSON exportado desde n8n.

**Contenido:** configuración del workflow, nodos, conexiones, System Message, guardrails, herramientas y observabilidad.

**Estado:** configuración completada, prueba funcional exitosa y archivo exportado.

---

### Autora

**María Victoria Caparroz**

Proyecto desarrollado para el curso de automatización e inteligencia artificial de **Coderhouse**.

**Stack tecnológico:** n8n · Groq · GPT-OSS 120B · Google Sheets · Gmail · GitHub
