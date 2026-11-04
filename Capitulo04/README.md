# Práctica 4. Analizar documentación técnica con Copilot Studio, generar el documento paso a paso de un proceso de despliegue y enviarlo a revisión mediante Power Automate. La práctica documenta el pipeline y sus controles, pero no ejecuta un pipeline de CI/CD.

## Metadatos

| Atributo | Detalle |
| :--- | :--- |
| **Duración** | **75 minutos** (prerrequisitos 7, fuente 8, procedimiento 20, Word 12, flujo 20, prueba 8). |
| **Complejidad** | Intermedia. |
| **Nivel Taxonómico de Bloom** | **Crear (nivel 6)**: producir un procedimiento documentado y organizar su aprobación verificable. |
| **Licencia Requerida** | Copilot Studio Trial activa para probar el agente; Microsoft 365 con SharePoint Online, Word para la Web, Outlook y Power Automate para conectores estándar, incluido **Approvals**. El entorno debe tener **Dataverse disponible para Approvals**. |
| **Plataformas Principales** | Copilot Studio (**standard harness**), SharePoint Online, Word para la Web, Power Automate (SharePoint, Approvals y Outlook estándar). |

---

## Descripción General

El equipo Atlas prepara la documentación de la versión **2.4.0** del Portal Atlas para el entorno de **preproducción (staging)**. Debe describir las fases del pipeline, las verificaciones, las responsabilidades y una reversa prevista sin ejecutar ninguna automatización de CI/CD. En el laboratorio 3 se preparó el agente `ATLAS - Asistente PMO y QA` con las fuentes SharePoint del proyecto; ahora se reutilizará ese agente para examinar una especificación técnica diferente.

Analizarás la especificación con prompts controlados, generando un procedimiento Word revisable. Después construirás un flujo de Power Automate que detecte el documento cargado en SharePoint, cree una **aprobación formal** con el conector estándar **Approvals** y comunique por Outlook la respuesta del revisor. La aprobación será **documental**, no autorizara ni ejecutará un despliegue real.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] **Extraer y estructurar información técnica**, identificando etapas, prerrequisitos, roles, validaciones, condiciones de suspensión y reversa a partir de una fuente SharePoint.
- [ ] **Elaborar y verificar un procedimiento**, produciendo un documento de despliegue que no inventa comandos ni resultados de ejecución.
- [ ] **Automatizar la revisión documental**, enviando un archivo de SharePoint a **Start and wait for an approval** y comunicando el resultado de la decisión.
- [ ] **Distinguir las autorizaciones**, comprobando que la aprobación del documento no ejecuta ni aprueba operativamente el pipeline.

---

## Prerrequisitos

### Conocimientos Previos

- Haber utilizado el agente de Copilot Studio del laboratorio 3 para fundamentar respuestas en SharePoint.
- Comprender la diferencia entre documentar un cambio, aprobar un documento y ejecutar una liberación.
- Conocer disparadores, condiciones y contenido dinámico de Power Automate.

### Requisitos de Acceso y Entorno

- Haber completado el laboratorio 3 y conservar el agente `ATLAS - Asistente PMO y QA` y la carpeta `ProyectoAtlas/FuentesAgente`.
- Disponer de [Especificacion_Despliegue_Atlas.docx](recursos/Especificacion_Despliegue_Atlas.docx) en **SharePoint**. Idealmente se cargo durante la preparación del laboratorio 3; comprobar que la fuente es consultable antes de iniciar los prompts.
- Tener cuenta de Microsoft 365 con permisos de escritura en la biblioteca SharePoint, Word para la Web, Outlook y derecho a crear flujos estándar en Power Automate.
- Disponer de un **revisor de laboratorio real**, con una cuenta autorizada del mismo tenant y permiso para abrir documentos SharePoint y responder solicitudes Approvals. Coordinar su disponibilidad para responder durante la práctica; no usar una dirección inventada.
- Confirmar **antes de la práctica, mediante una solicitud de aprobación de prueba previamente completada por el administrador o responsable del entorno**, que **Approvals funciona en el entorno elegido**: el conector es estándar, pero sus registros se almacenan en **Dataverse**. En entornos **no predeterminados**, la primera provisión de Dataverse para aprobaciones puede requerir una primera ejecución por un **administrador del entorno**. En el entorno predeterminado, Power Automate puede aprovisionar la base automáticamente, sujeto a políticas del tenant.
- Tener habilitadas las conexiones SharePoint, Approvals y Office 365 Outlook. No se requieren pipelines, servidores reales, HTTP, AI Builder, licencias Premium ni publicación del agente Trial.

> **Condición de inicio:** Si Approvals/Dataverse no está disponible o falta un aprobador autorizado, resolver ese requisito **antes** de la sesión. No reemplazar la aprobación formal por un simple correo porque dejaria incompleto el objetivo del temario.

---

## Entorno de Laboratorio

### Especificaciones de Software y Servicios Cloud

| Herramienta / Servicio | Versión / Release | Enlace Oficial / Origen | Licencia / Permisos |
| :--- | :--- | :--- | :--- |
| Copilot Studio | Cloud vigente, **standard harness**; panel de pruebas. | [Portal](https://copilotstudio.microsoft.com/) / [Trial](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-licensing) | Trial activa, agente del laboratorio 3; **sin publicación**. |
| SharePoint Online | Cloud vigente; `When a file is created (properties only)`. | [Conector SharePoint](https://learn.microsoft.com/en-us/connectors/sharepointonline/) | Leer fuente y crear archivos en biblioteca de práctica. |
| Word para la Web | Servicio Microsoft 365 vigente. | [Microsoft 365](https://www.microsoft365.com/) | Crear, editar y guardar `.docx` en SharePoint/OneDrive. |
| Power Automate / Approvals | `Start and wait for an approval` (acción actual, no deprecated). | [Conector Approvals](https://learn.microsoft.com/en-us/connectors/approvals/) / [requisitos Dataverse](https://learn.microsoft.com/en-us/power-automate/get-started-approvals) | Conector estándar; base Dataverse habilitada y revisor autorizado. |
| Office 365 Outlook | `Send an email (V2)` vigente. | [Conector Outlook](https://learn.microsoft.com/en-us/connectors/office365/) | Buzón de notificaciones de laboratorio. |

**Datos de consulta**

| Parámetro / Campo | Valor / Configuración |
| :--- | :--- |
| Fuente | [Especificacion_Despliegue_Atlas.docx](recursos/Especificacion_Despliegue_Atlas.docx). |
| Proyecto / cambio | `PRJ-ATLAS` / `CHG-ATLAS-024`. |
| Versión propuesta y anterior | `2.4.0` / `2.3.9`. |
| Ambiente | `staging` (no producción). |
| Artefacto de referencia | `atlas-web_2.4.0.zip` (nombre documental, no archivo entregado ni desplegable). |
| Pipeline de referencia | `PIPE-ATLAS-REL` (solo descripción, nunca ejecución). |
| Agente | `ATLAS - Asistente PMO y QA`, fuente `Fuentes_Atlas`. |
| Documento resultado | `Procedimiento_Despliegue_Atlas_v2_4_0.docx`. |
| Carpeta de revisión | `ProyectoAtlas/EnRevision` en la biblioteca SharePoint del sitio de práctica. |
| Flujo | `ATLAS_04_Aprobacion_Despliegue`. |

---

## Instrucciones Paso a Paso

### Paso 1: Verificar los accesos para aprobaciones (7 min)

**Objetivo:** Comprobar las condiciones previas para que la revisión sea una aprobación real y no una notificación sin respuesta.

1. Abrir Power Automate con la misma cuenta del laboratorio 3 y seleccionar el entorno autorizado.
2. Abrir **Conexiones** y comprobar que SharePoint y Office 365 Outlook están autorizados. La conexión **Approvals** puede crearse al insertar por primera vez la acción en un flujo y no tiene que aparecer previamente en la lista. Confirmar con el administrador que **Approvals** está permitido por la política DLP y que el entorno tiene **Dataverse preparado** para registrar aprobaciones. Si aparece un error de provisión, no continuar hasta resolverlo.
3. Abrir el sitio SharePoint de práctica, comprobar la biblioteca **Documentos** y confirmar que puede crearse la carpeta `ProyectoAtlas/EnRevision`.
4. Confirmar con el revisor asignado que puede abrir esa biblioteca, recibir una aprobación y responder desde Outlook o el centro de aprobaciones de Power Automate.
5. Comprobar que el agente Atlas continua disponible en **Test your agent**. No publicar ni habilitar canales del agente.

> **Nota técnica:** El conector Approvals es estándar; este laboratorio no requiere comprar una licencia de Dataverse para aplicaciones personalizadas solo para crear aprobaciones. Se usa Dataverse **como soporte de aprobaciones**, sin crear tablas ni soluciones Premium.

**Resultado esperado:** Hay un entorno que admite aprobaciones y un revisor real autorizado.

**Comprobación:** Se puede seleccionar Approvals y el revisor confirma acceso SharePoint; no existe bloqueo de licencia o base pendiente de provisión.

---

### Paso 2: Contrastar la especificación en SharePoint (8 min)

**Objetivo:** Identificar los datos técnicos presentes en la fuente y los límites que deben conservarse al documentar.

1. Abrir [Especificacion_Despliegue_Atlas.docx](recursos/Especificacion_Despliegue_Atlas.docx) o su copia en la carpeta `ProyectoAtlas/FuentesAgente` de SharePoint.
2. Localizar las secciones **Entorno, componentes y artefactos**, **Condiciones de entrada**, **Validaciones**, **Reversión prevista** y **Dudas que deben permanecer explicitas**.
3. Abrir `ATLAS - Asistente PMO y QA` en Copilot Studio y verificar el origen SharePoint `Fuentes_Atlas` sin sustituirlo por un documento cargado directamente desde el dispositivo.
4. Preguntar en el panel de prueba: `Para el cambio CHG-ATLAS-024 del Portal Atlas 2.4.0 en staging, ¿que etapas se describen y cuál es la versión de reversa?`.
5. Contrastar la respuesta: etapas `Construcción`, `Pruebas QA`, `Validación`, `Despliegue a staging`, `Verificación`; retorno previsto a `2.3.9`. Si la fuente no responde, detener los prompts y aplicar las correcciones de **Solución de problemas**.

> **Nota técnica:** `https://staging.atlas.example/health`, cuando aparezca en la fuente, es una dirección **ilustrativa**. No intentar conectarse a ese dominio; este laboratorio no prueba una API real.

**Resultado esperado:** El agente reconoce el cambio documental, su entorno, etapas y versión anterior.

**Comprobación:** La respuesta coincide con la especificación y no presenta comandos o validaciones ya ejecutadas.

---

### Paso 3: Generar el procedimiento documental con el agente (20 min)

**Objetivo:** Obtener una primera versión estructurada y corregible a partir de hechos documentados.

1. Enviar en el panel del agente el siguiente prompt para levantar datos verificables:

   **Prompt 1 - Extracción de controles**

   ```text
   Analiza solamente la especificación técnica del cambio CHG-ATLAS-024,
   versión 2.4.0 del Portal Atlas para staging, ubicada en las fuentes
   SharePoint autorizadas de PRJ-ATLAS.
   Extrae en una tabla: versión, entorno, artefacto, responsables, entradas,
   etapas del pipeline, criterios de avance, pruebas previstas, suspensión,
   reversa y evidencias.
   Cita la sección que respalda cada dato. Separa en 'Pendiente de
   confirmación' los comandos, rutas, ventanas, umbrales o autorizaciones
   que el documento no contiene. No ejecutes ningún procedimiento.
   ```

2. Verificar contra la fuente los datos `2.4.0`, `2.3.9`, `CHG-ATLAS-024`, `atlas-web_2.4.0.zip`, `PIPE-ATLAS-REL` y `staging`.
3. Enviar el prompt para redactar el resultado final:

   **Prompt 2 - Procedimiento de despliegue para revisión**

   ```text
   Con la misma especificación del cambio CHG-ATLAS-024, redacta un
   PROCEDIMIENTO DOCUMENTAL de despliegue para el Portal Atlas 2.4.0
   en preproducción. Estructura exactamente estos apartados:
   1. Objetivo y alcance.
   2. Identificador, versión y entorno.
   3. Roles y responsables.
   4. Prerrequisitos, entradas y dependencias.
   5. Verificaciones documentales previas.
   6. Etapas previstas del pipeline, numeradas y sin ejecución.
   7. Controles posteriores y checklist de pruebas previstas.
   8. Criterios de éxito y suspensión.
   9. Acciones de reversa documentadas.
   10. Evidencias y comunicación a PMO.
   11. Datos pendientes de confirmación.
   Para cada etapa indica el responsable y la condición de avance si la
   fuente lo especifica. No inventes comandos, rutas, credenciales, horarios,
   aprobadores operativos ni resultados. Diferencia una aprobación del
   DOCUMENTO de una autorización para EJECUTAR el despliegue.
   Incluye una advertencia clara: 'Solo revisión documental. No ejecutar'.
   ```

4. Contrastar que la secuencia se documenta en el orden `Construcción > Pruebas QA > Validación > Despliegue a staging > Verificación`, sin ponerla en marcha.
5. Enviar un último prompt para descubrir errores antes de la revisión:

   **Prompt 3 - Auditoría de omisiones**

   ```text
   Compara el procedimiento propuesto con la especificación de
   CHG-ATLAS-024. Identifica apartados no respaldados, controles omitidos
   y supuestos que deben consultar a Operaciones/PMO.
   Examina especialmente respaldo 2.3.9, autorización, evidencia QA,
   pruebas de humo, reglas de suspensión y datos faltantes.
   Devuelve correcciones por apartado. No simules que se realizaron
   pruebas, aprobaciones, respaldos o un despliegue.
   ```

6. Aplicar solo las correcciones sustentadas en el documento. Conservar como preguntas los vacíos sobre ventana, rutas, umbrales y aprobaciones operativas.

**Resultado esperado:** Borrador documental con 11 apartados y correcciones justificadas.

**Comprobación:** El procedimiento distingue fases previstas, controles y datos faltantes; no presenta la URL ilustrativa como servicio activo ni el respaldo como operacion realizada.

---

### Paso 4: Crear el documento final en Word para la Web (12 min)

**Objetivo:** Entregar un archivo editable sobre el que pueda decidir un revisor humano.

1. Crear en SharePoint o Word para la Web un documento nuevo llamado `Procedimiento_Despliegue_Atlas_v2_4_0.docx` en la carpeta de trabajo del proyecto (no en `EnRevision` todavia).
2. Agregar el título `Procedimiento documental de despliegue - Portal Atlas 2.4.0` y el encabezado `CHG-ATLAS-024 | Entorno: staging | Estado: Pendiente de revisión documental`.
3. Copiar el contenido **revisado** del Prompt 2 con sus correcciones a los **11 apartados**. Mantener las referencias a roles, entradas, etapa del pipeline, validaciones, criterios de suspensión, reversa y temas por confirmar.
4. Incluir una línea visible junto al alcance: **`Solo revisión documental. No ejecutar ni interpretar como autorización operativa.`**
5. Guardar el documento, abrirlo para comprobar que contiene todas sus secciones y obtener una copia `.docx` si es necesario cargarla desde OneDrive o el dispositivo a la carpeta de revisión.

> **Nota técnica:** No es necesaria ninguna acción automatizada de Word Online (Business) ni un conector Premium. La creación y revisión del DOCX se realizan manualmente en Word para la Web.

**Resultado esperado:** `Procedimiento_Despliegue_Atlas_v2_4_0.docx` completo y listo para la revisión del equipo.

**Comprobación:** El documento abre en Word, conserva los 11 apartados y las restricciones, y no existe todavia una aprobación documental asociada.

---

### Paso 5: Crear un flujo de aprobación documental e integrarlo como herramienta del agente (20 min)

**Objetivo:** Configurar un flujo de Power Automate que pueda ejecutarse desde un agente de Copilot Studio para solicitar la revisión formal de un procedimiento de despliegue, registrar la decisión de un aprobador y comunicar el resultado mediante Outlook.

1. **Verificar los requisitos del entorno.** Antes de comenzar, comprobar las siguientes condiciones:

   - Tener acceso a Power Automate y Copilot Studio dentro del mismo entorno de Power Platform.
   - Disponer de capacidad de Copilot Studio que permita crear o convertir flujos de agente. La licencia Trial, por sí sola, no garantiza esta capacidad.
   - Contar con SharePoint Online y Outlook de Microsoft 365.
   - Tener habilitado el servicio **Approvals**, con Dataverse disponible y las conexiones autorizadas.
   - Identificar el correo electrónico real de un usuario que actuará como revisor.
   - Tener disponible el agente creado durante los pasos anteriores de la práctica, utilizando el **standard harness**.

2. **Preparar la ubicación del documento.** Abrir SharePoint, acceder a la biblioteca **Documentos** y crear la carpeta `ProyectoAtlas/EnRevision`, si todavía no existe.

   Esta carpeta se utilizará para almacenar el documento `Procedimiento_Despliegue_Atlas_v2_4_0.docx`, generado y revisado durante las actividades anteriores. El archivo se cargará en el paso 6 de la práctica, cuando se ejecute la prueba.

3. **Crear el flujo en Power Automate.** Acceder a [Power Automate](https://make.powerautomate.com/), verificar el entorno seleccionado y crear un flujo de nube instantáneo con las siguientes características:

   | Parámetro | Configuración |
   | :--- | :--- |
   | Nombre | `ATLAS_04_Aprobacion_Despliegue` |
   | Tipo | Flujo de nube instantáneo |
   | Desencadenador | `When an agent calls the flow` |
   | Plataforma | Power Automate |
   | Finalidad | Solicitar aprobación documental desde Copilot Studio |

   Seleccionar el desencadenador **When an agent calls the flow**. Este permitirá iniciar la ejecución desde el agente, en lugar de utilizar un disparador de SharePoint.

4. **Configurar los parámetros de entrada.** Abrir el desencadenador `When an agent calls the flow` y agregar dos entradas de tipo **Text / Texto**:

   | Parámetro | Tipo | Finalidad |
   | :--- | :--- | :--- |
   | `nombreDocumento` | Text | Identificar el nombre exacto del archivo Word que se enviará a revisión |
   | `vinculoDocumento` | Text | Recibir la dirección real de SharePoint que permite abrir el documento |

   Estos parámetros recibirán sus valores desde la conversación con el agente de Copilot Studio.

   El participante proporcionará ambos datos durante el paso 6:

   - `nombreDocumento`: nombre del archivo Word generado durante la práctica, por ejemplo, `Procedimiento_Despliegue_Atlas_v2_4_0.docx`.
   - `vinculoDocumento`: dirección obtenida al seleccionar el archivo en SharePoint y utilizar **Copiar vínculo / Copy link**.

   > **Nota técnica:** En esta etapa únicamente se definen los parámetros. No introducir valores fijos ni direcciones de ejemplo. Los datos se recibirán cuando el agente ejecute el flujo. La configuración de conocimiento de SharePoint del agente no implica que pueda obtener automáticamente el vínculo exacto del archivo.

5. **Configurar la respuesta inicial al agente.** Agregar la acción **Respond to the agent** inmediatamente después del desencadenador.

   Crear una salida con los siguientes valores:

   | Campo | Configuración |
   | :--- | :--- |
   | Nombre | `estadoSolicitud` |
   | Tipo | Text |
   | Valor | `Se recibió la solicitud de revisión documental. El proceso de aprobación continuará en Power Automate. Esta confirmación no significa que el documento esté aprobado ni que la solicitud de aprobación ya se haya creado.` |

   Abrir **Settings / Configuración** de esta acción y establecer **Asynchronous response = Off / Desactivado**.

   Esta respuesta permite que el agente termine su interacción inicial sin esperar a que el revisor apruebe o rechace el documento.

6. **Agregar la aprobación formal.** Después de `Respond to the agent`, incorporar la acción estándar **Approvals > Start and wait for an approval**, seleccionando la versión que no está marcada como obsoleta.

   Configurar los siguientes valores:

   | Campo | Valor / Configuración |
   | :--- | :--- |
   | **Approval type** | `Approve/Reject - First to respond` |
   | **Title** | `Revisión documental \| CHG-ATLAS-024 \|` seguido del contenido dinámico `nombreDocumento` |
   | **Assigned to** | Correo electrónico real del revisor |
   | **Details** | `Revisar los 11 apartados del procedimiento, el respaldo de la versión 2.3.9, las condiciones de suspensión, las acciones de reversa y los vacíos documentales. Aprobar o rechazar únicamente el documento. Esta aprobación NO autoriza ejecutar un despliegue.` |
   | **Item link** | Contenido dinámico `vinculoDocumento` del desencadenador |
   | **Item link description** | `Abrir procedimiento de despliegue en SharePoint` |

   Seleccionar los parámetros `nombreDocumento` y `vinculoDocumento` mediante el panel de **Contenido dinámico**. No escribir sus nombres como texto literal cuando se necesite utilizar el valor recibido.

   Comprobar que el revisor tiene permisos para abrir el documento almacenado en SharePoint.

7. **Configurar la condición de aprobación.** Después de `Start and wait for an approval`, agregar **Condition / Condición** con la siguiente configuración:

   | Campo | Configuración |
   | :--- | :--- |
   | Valor izquierdo | `Outcome / Resultado` de la acción de aprobación |
   | Operador | `is equal to / es igual a` |
   | Valor derecho | `Approve` |

   Utilizar el valor técnico `Approve` para identificar la respuesta de aprobación. El resultado deberá verificarse posteriormente en el historial de ejecución.

8. **Configurar la notificación de aprobación.** En la rama **If yes / Verdadera**, agregar **Office 365 Outlook > Send an email (V2)**.

   | Campo | Configuración |
   | :--- | :--- |
   | **To / Para** | Correo del buzón de prueba |
   | **Subject / Asunto** | `ATLAS \| Procedimiento aprobado documentalmente \| CHG-ATLAS-024` |
   | **Body / Cuerpo** | `El revisor aprobó el procedimiento documental. Esta decisión NO autoriza ejecutar PIPE-ATLAS-REL.` |

   Incorporar en el cuerpo del mensaje los siguientes elementos mediante **Contenido dinámico**:

   - `nombreDocumento`: nombre del documento revisado.
   - `vinculoDocumento`: vínculo para abrir el documento en SharePoint.

   De esta manera, el correo permitirá identificar el documento específico que fue aprobado.

9. **Configurar la notificación de rechazo.** En la rama **If no / Falsa**, agregar otra acción **Send an email (V2)** con los siguientes valores:

   | Campo | Configuración |
   | :--- | :--- |
   | **To / Para** | Correo del buzón de prueba |
   | **Subject / Asunto** | `ATLAS \| Procedimiento requiere cambios \| CHG-ATLAS-024` |
   | **Body / Cuerpo** | `El procedimiento fue rechazado durante la revisión documental. Consultar la decisión y los comentarios del revisor en Approvals antes de generar una nueva versión.` |

   Agregar también los contenidos dinámicos `nombreDocumento` y `vinculoDocumento` recibidos desde el desencadenador.

   Los comentarios del revisor podrán consultarse en el registro de Approvals.

10. **Guardar el flujo y comprobar su estructura.** Guardar los cambios en Power Automate y verificar que la secuencia de acciones sea la siguiente:

    1. `When an agent calls the flow`
    2. `Respond to the agent`
    3. `Start and wait for an approval`
    4. `Condition`
    5. `Send an email (V2)` en la rama de aprobación
    6. `Send an email (V2)` en la rama de rechazo

    Comprobar que las conexiones de Approvals y Outlook se encuentran habilitadas y que el flujo no presenta errores de configuración.

11. **Incorporar el flujo a una solución de Power Platform.** Para permitir su administración desde Copilot Studio, realizar las siguientes acciones:

    1. En Power Automate, abrir **Soluciones / Solutions**.
    2. Seleccionar una solución no administrada existente o crear una nueva denominada `ATLAS_Automatizacion`.
    3. Abrir la solución y seleccionar **Agregar existente / Add existing**.
    4. Elegir **Automatización / Automation > Flujo de nube / Cloud flow**.
    5. Abrir la pestaña **Outside Dataverse / Fuera de Dataverse**, cuando corresponda.
    6. Buscar `ATLAS_04_Aprobacion_Despliegue`.
    7. Seleccionar el flujo y utilizar **Agregar / Add**.
    8. Verificar que el flujo aparece dentro de la solución.

    Si el flujo ya se creó dentro de una solución, no es necesario repetir este procedimiento.

12. **Cambiar el plan del flujo a Copilot Studio.** Abrir la página de detalles del flujo en Power Automate y realizar lo siguiente:

    1. Verificar que el flujo pertenece a una solución y se encuentra en el mismo entorno que el agente.
    2. Seleccionar **Edit / Editar**.
    3. Localizar la propiedad **Plan** del flujo.
    4. Cambiar el plan a **Copilot Studio**.
    5. Guardar los cambios.
    6. Confirmar la conversión cuando Power Automate lo solicite.
    7. Comprobar que el flujo convertido aparece en **Flows / Flujos** de Copilot Studio.

    > **Nota técnica:** La conversión requiere capacidad de Copilot Studio asignada al entorno y modifica el modelo de facturación del flujo. Es una operación irreversible. Si la opción **Plan** aparece deshabilitada, comprobar que el flujo pertenece a una solución y que el entorno cuenta con capacidad suficiente. No continuar con la conversión si esta condición no se cumple.

13. **Agregar el flujo como herramienta del agente.** Regresar a [Copilot Studio](https://copilotstudio.microsoft.com/), abrir el agente utilizado para analizar la documentación técnica y realizar las siguientes acciones:

    1. Seleccionar **Tools / Herramientas**.
    2. Elegir **Add a tool / Agregar una herramienta**.
    3. Seleccionar **Flow / Flujo** entre los tipos de herramientas disponibles.
    4. Buscar `ATLAS_04_Aprobacion_Despliegue`.
    5. Seleccionar **Add and configure / Agregar y configurar**.
    6. En **Details / Detalles**, asignar el nombre visible `Solicitar revisión documental`.
    7. Introducir la siguiente descripción de la herramienta:

       > Permite iniciar la revisión formal de un procedimiento de despliegue mediante Power Automate. Utilizar únicamente cuando el usuario solicite explícitamente enviar un documento a aprobación. Antes de invocar la herramienta, obtener el nombre exacto del archivo y su vínculo real de SharePoint. Si falta alguno de estos datos, solicitarlo al usuario. No inventar enlaces ni interpretar la generación del documento como autorización para enviarlo. La herramienta inicia el proceso de revisión, pero no aprueba documentos ni autoriza despliegues.

    8. Mantener habilitada la selección dinámica de la herramienta por el agente, cuando esta configuración esté disponible.
    9. Habilitar la confirmación del usuario antes de ejecutar la herramienta, si la interfaz proporciona esa opción.

    > **Importante:** No utilizar el conector **Power Automate Management** ni seleccionar las acciones `Turn On Flow`, `Get Flow` o `Resubmit Flow`. Estas acciones administran flujos existentes, pero no incorporan el flujo de aprobación como herramienta del agente.

14. **Configurar las entradas dinámicas de la herramienta.** En la configuración de `Solicitar revisión documental`, abrir **Inputs / Entradas**.

    Para el parámetro `nombreDocumento`:

    1. Localizar la entrada `nombreDocumento`.
    2. En **Fill using / Rellenar con**, seleccionar **Rellenar dinámicamente con IA**.
    3. Seleccionar **Customize / Personalizar**, si aparece la opción.
    4. En la descripción del parámetro, introducir:

       > Obtener el nombre exacto del documento Word que el usuario desea enviar a revisión, incluyendo la extensión .docx. Extraer el valor de la conversación. Si el usuario no proporciona el nombre, solicitarlo antes de ejecutar la herramienta. No inventar nombres de archivo.

    5. Mantener el tipo de entrada **Text / Texto**.

    Para el parámetro `vinculoDocumento`:

    1. Localizar la entrada `vinculoDocumento`.
    2. En **Fill using / Rellenar con**, seleccionar **Rellenar dinámicamente con IA**.
    3. Seleccionar **Customize / Personalizar**, si aparece la opción.
    4. En la descripción del parámetro, introducir:

       > Obtener el vínculo real de SharePoint al documento que se enviará a revisión. Extraer la dirección proporcionada por el usuario en la conversación. El vínculo debe corresponder al archivo específico y no únicamente al sitio de SharePoint. Si no se proporciona, solicitarlo antes de ejecutar la herramienta. No construir ni inventar direcciones.

    5. Mantener el tipo de entrada **Text / Texto**.

    > **Nota técnica:** **Rellenar dinámicamente con IA** permite que Copilot Studio obtenga los valores desde la conversación. Si no identifica alguno de ellos, puede solicitarlo al participante. No seleccionar **Valor personalizado**, porque el objetivo es que el agente procese el documento indicado en cada solicitud y no un archivo fijo.

15. **Configurar la respuesta de finalización de la herramienta.** Abrir **Completion / Finalización** y realizar las siguientes acciones:

    1. Seleccionar **Send specific response / Enviar una respuesta específica**, si está disponible.
    2. Incorporar, mediante el selector de variables, la salida `estadoSolicitud` definida en la acción `Respond to the agent`.
    3. Comprobar que el mensaje indique que la solicitud fue recibida y que el proceso de aprobación continúa en Power Automate.
    4. Si la opción de respuesta específica no está disponible, seleccionar **Write the response with generative AI / Escribir respuesta con IA generativa** y verificar durante la prueba que utiliza correctamente el valor de `estadoSolicitud`.
    5. Guardar la configuración de la herramienta.

    El agente no debe afirmar que el documento fue aprobado, rechazado o enviado correctamente al revisor antes de que Power Automate registre la operación correspondiente.

16. **Verificar la integración antes de ejecutar la prueba.** Abrir nuevamente la herramienta `Solicitar revisión documental` y comprobar:

    - El flujo asociado es `ATLAS_04_Aprobacion_Despliegue`.
    - Los parámetros `nombreDocumento` y `vinculoDocumento` utilizan **Rellenar dinámicamente con IA**.
    - Las descripciones de las entradas indican cómo obtener los datos desde la conversación.
    - La salida `estadoSolicitud` está configurada para comunicar la recepción de la solicitud.
    - La herramienta se guarda sin errores.
    - El flujo contiene las acciones de aprobación y notificación por Outlook.
    - No existen acciones que ejecuten ni autoricen un pipeline de CI/CD.

> **Nota técnica:** El flujo responde inicialmente al agente mediante `Respond to the agent` y continúa con `Start and wait for an approval` mientras el revisor toma una decisión. Esta estructura evita mantener bloqueada la conversación durante la aprobación humana. La respuesta inicial confirma únicamente que el flujo recibió la solicitud; la creación efectiva de la aprobación y la decisión del revisor deben comprobarse en el historial de Power Automate. La integración puede probarse desde el panel de pruebas de Copilot Studio sin publicar el agente en un canal externo.

**Resultado esperado:** Flujo `ATLAS_04_Aprobacion_Despliegue` incorporado como herramienta `Solicitar revisión documental` del agente de Copilot Studio, con dos parámetros configurados para recibir dinámicamente el nombre y vínculo del documento. Power Automate contiene la lógica necesaria para solicitar la aprobación, registrar la decisión del revisor y enviar la notificación correspondiente por Outlook.

**Comprobación:** Verificar que el flujo aparece entre las herramientas del agente, que `nombreDocumento` y `vinculoDocumento` están configurados mediante **Rellenar dinámicamente con IA**, que la salida `estadoSolicitud` está disponible y que todas las acciones y conexiones se guardan sin errores. La invocación del flujo, la recepción de los valores y la aprobación real se comprobarán durante el paso 6.

---

### Paso 6: Solicitar la aprobación desde el agente y validar el resultado (8 min)

**Objetivo:** Ejecutar desde Copilot Studio el flujo de revisión documental, verificar que Power Automate registra la solicitud, obtener una decisión humana y comprobar la notificación correspondiente.

1. **Preparar el documento para revisión.** Abrir SharePoint y cargar el archivo `Procedimiento_Despliegue_Atlas_v2_4_0.docx`, generado y revisado durante los pasos anteriores, en la carpeta `ProyectoAtlas/EnRevision`. Si el documento ya existe, utilizar la versión preparada para esta prueba y evitar enviar solicitudes duplicadas. Seleccionar el archivo y utilizar **Copiar vínculo / Copy link** para obtener su dirección real de SharePoint.

2. **Abrir el panel de pruebas del agente.** Regresar a Copilot Studio, seleccionar el agente que contiene la herramienta `Solicitar revisión documental` y abrir **Test your agent / Probar agente**.

3. **Solicitar el inicio de la revisión documental** mediante el siguiente mensaje, sustituyendo el marcador del vínculo por el enlace copiado de SharePoint:

   > Necesito enviar a revisión formal el procedimiento de despliegue del proyecto PRJ-ATLAS, correspondiente al cambio CHG-ATLAS-024. El archivo se llama `Procedimiento_Despliegue_Atlas_v2_4_0.docx` y está almacenado en SharePoint. Su vínculo es: [PEGAR VÍNCULO REAL]. Utiliza la herramienta de revisión documental para iniciar el proceso de aprobación y confirma cuando se haya recibido la solicitud. No autorices ni ejecutes ningún despliegue.

4. **Verificar la invocación de la herramienta.** Comprobar en el panel de pruebas que el agente selecciona `Solicitar revisión documental` y utiliza los parámetros recibidos. Si solicita confirmación de los valores, validar el nombre y vínculo del documento antes de continuar.

5. **Comprobar la respuesta del agente.** Verificar que el agente devuelve la confirmación configurada en `estadoSolicitud`. Esta respuesta indica que el flujo recibió la solicitud, pero no representa todavía una decisión de aprobación.

6. **Inspeccionar la ejecución en Power Automate.** Abrir el historial del flujo `ATLAS_04_Aprobacion_Despliegue` y comprobar:

   - El desencadenador `When an agent calls the flow` recibió el nombre y vínculo del documento.
   - La acción `Respond to the agent` se ejecutó correctamente.
   - La acción `Start and wait for an approval` creó una solicitud de aprobación y permanece **Running / Waiting** mientras el revisor no responda.
   - La ejecución no termina necesariamente al responder al agente.

7. **Responder como revisor autorizado.** Abrir la solicitud desde Outlook o el centro **Approvals** de Power Automate. Acceder al documento mediante el vínculo recibido, revisar sus apartados y seleccionar:

   - **Approve / Aprobar:** cuando el procedimiento cumple los controles documentales establecidos.
   - **Reject / Rechazar:** cuando existen inconsistencias, información incompleta o aspectos técnicos que requieren corrección.

   En caso de rechazo, registrar un comentario que describa la corrección necesaria.

8. **Comprobar la decisión y la notificación final.** Regresar al historial del flujo y verificar que `Outcome` refleja la respuesta seleccionada y que únicamente se ejecutó la rama correspondiente. Abrir el correo recibido en Outlook y comprobar el asunto, el código `CHG-ATLAS-024`, el nombre del documento y su vínculo de SharePoint.

> **Nota técnica:** La respuesta inicial del agente y la decisión del revisor representan dos momentos diferentes del proceso. El agente confirma la recepción de la solicitud; Power Automate registra la aprobación o el rechazo mediante el servicio Approvals. Si la creación de la aprobación falla después de la respuesta inicial, identificar el error en el historial de ejecución. No presentar la solicitud como aprobada mientras no exista una decisión humana registrada. La prueba no requiere publicar el agente en Teams ni ejecutar un pipeline de CI/CD.

**Resultado esperado:** El participante consigue iniciar desde la conversación con Copilot Studio un flujo de aprobación documental, recibe la confirmación inicial del agente y comprueba que Power Automate solicita una decisión humana y comunica el resultado mediante Outlook.

**Comprobación:** El historial de ejecución muestra la invocación desde el agente, los parámetros recibidos, la respuesta inicial, la solicitud de aprobación, la decisión del revisor y la notificación correspondiente. El documento permanece accesible en SharePoint, la respuesta registrada coincide con el correo enviado y ninguna acción ejecuta o autoriza un despliegue de CI/CD.

---

## Validación y Pruebas

### Ejecución de Prueba Básica

1. Mostrar en Copilot Studio la consulta de `CHG-ATLAS-024` con la fuente `Fuentes_Atlas` de SharePoint, sin publicar el agente.
2. Abrir el DOCX generado y confirmar 11 apartados, versión `2.4.0`, entorno staging, reversa documentada a `2.3.9` y texto `Solo revisión documental. No ejecutar`.
3. Verificar en Power Automate que existe una acción **Start and wait for an approval** con un revisor real y un **Outcome** posterior a su respuesta.
4. Contrastar la rama de Outlook que se ejecutó con la decisión registrada; confirmar que no se invoca ningún pipeline.

### Pruebas de Resiliencia y Casos Adversos (Adversarial Testing)

Estas pruebas comprueban exactitud documental y flujo de autorización; no simulan un despliegue real. Cuando una variante requiera otra aprobación humana, ejecutarla solo si el revisor está disponible; nunca marcar una prueba no realizada como exitosa.

#### Casos de Prueba Lógica y de Datos

| Escenario | Acción de Prueba | Resultado Esperado en la Validación |
| :--- | :--- | :--- |
| **Aprobación documental** | Revisor responde `Approve` tras abrir el DOCX. | Se ejecuta correo de aprobado; el pipeline no se ejecuta ni se autoriza. |
| **Rechazo de documento** | En una **segunda carga** con nombre único, el revisor selecciona `Reject` y agrega comentario. | Se ejecuta correo de rechazo; no aparece mensaje de aprobado. Registrar el caso como pendiente si no se ejecutó. |
| **Datos técnicos inexistentes** | Pedir al agente la ruta física del servidor y una ventana exacta de despliegue. | No inventar rutas ni fechas; `Pendiente de confirmación`. |
| **Aprobación sin respuesta** | Observar la ejecución antes de que responda el revisor. | La acción permanece esperando; no hay `Outcome` ni correo final mientras no exista decisión. |

---

## Solución de Problemas

### Problema 1: La fuente técnica de SharePoint no aparece en Copilot Studio

- **Síntomas:** El agente no identifica `CHG-ATLAS-024` o mezcla requisitos funcionales con el procedimiento técnico.
- **Causa Raíz:** El documento no está en la ubicación registrada, falta índice/búsqueda o los permisos de lectura no son suficientes.
- **Solución:**
  1. Comprobar que la especificación abre desde SharePoint con la cuenta utilizada por el agente.
  2. Revisar la URL o selección registrada en `Fuentes_Atlas` y esperar que el origen este disponible; preguntar por `CHG-ATLAS-024`, no solo por el nombre del archivo.
  3. Volver a probar la consulta de etapas. No sustituir SharePoint por un archivo local ni afirmar datos no recuperados.

### Problema 2: `Start and wait for an approval` presenta errores de entorno

- **Síntomas:** La acción no se crea, solicita una base o falla con un mensaje asociado a Dataverse.
- **Causa Raíz:** Aprobaciones aún no preparadas, privilegios insuficientes en el primer aprovisionamiento de un entorno no predeterminado o restricciones DLP.
- **Solución:**
  1. Confirmar que **Approvals** esta permitido por las políticas del entorno y que el conector es el estándar.
  2. Solicitar al administrador del entorno que compruebe Dataverse y complete el aprovisionamiento si aplica.
  3. Reabrir la conexión y probar de nuevo; no degradar el ejercicio a un simple correo ni crear tablas Dataverse manuales.

### Problema 3: No llega la solicitud o el flujo no finaliza

- **Síntomas:** El revisor no ve la aprobación o el flujo permanece en ejecución.
- **Causa Raíz:** Correo incorrecto, falta de acceso del revisor, carpeta equivocada o solicitud pendiente de respuesta.
- **Solución:**
  1. Abrir historial para distinguir un disparador sin ejecución de una aprobación en estado **Waiting**.
  2. Confirmar que el archivo fue **creado** en `EnRevision` y que el revisor existe y tiene acceso a SharePoint y Approvals.
  3. Pedir al revisor que responda desde el centro de aprobaciones; al finalizar, comprobar `Outcome` y la rama de Outlook.

### Problema 4: El flujo entra en un ciclo o envía correos repetidos

- **Síntomas:** Varias ejecuciones se activan al modificar archivos o cargar versiones adicionales.
- **Causa Raíz:** Se eligio un disparador `created or modified` o el flujo esta escribiendo de nuevo en la carpeta supervisada.
- **Solución:**
  1. Cambiar al disparador **When a file is created (properties only)** y verificar `Folder = ProyectoAtlas/EnRevision`.
  2. Eliminar del flujo acciones que vuelvan a guardar o modificar documentos en esa carpeta.
  3. Guardar, subir una sola copia nueva y comprobar que corresponde a una solicitud de aprobación.

---

## Limpieza

Una vez validados los resultados:

1. Conservar el archivo `Procedimiento_Despliegue_Atlas_v2_4_0.docx` aprobado o rechazado como evidencia **de revisión documental**, y no de ejecución técnica.
2. Cerrar o cancelar **solicitudes de prueba pendientes** cuando sea posible y solo si no se afectan procesos de otros usuarios.
3. Desactivar los flujos de práctica o retirar las copias temporales del sitio **solo al finalizar el curso** y con autorización del propietario. No eliminar documentos de otros participantes.
4. No ejecutar `PIPE-ATLAS-REL`, no usar la URL `.example` como endpoint real y no publicar el agente Trial.

---

## Resumen

En este laboratorio has aprendido a:

1. **Interpretar documentación de despliegue:** Distinguir etapas previstas, controles documentales, reversa y vacíos de información.
2. **Producir un procedimiento verificable:** Generar un DOCX estructurado sin inventar comandos ni resultados.
3. **Automatizar una decisión formal:** Enviar un archivo SharePoint a Approvals, esperar respuesta y notificarla con Outlook.
4. **Mantener límites operativos:** Separar la aprobación documental de la autorización y ejecución de CI/CD.

**Producto verificable:** procedimiento `.docx`, flujo `ATLAS_04_Aprobacion_Despliegue`, registro de `Outcome` y correo final del revisor; **ningún despliegue ejecutado**.
