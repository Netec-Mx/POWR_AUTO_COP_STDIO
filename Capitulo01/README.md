# Práctica 1. Crear un flujo básico de seguimiento que detecte un cambio en una lista de SharePoint, evalúe el estado de una actividad y envíe una notificación automática por correo.

## Metadatos

| Atributo | Detalle |
| :--- | :--- |
| **Duración** | **35 minutos** (preparación 5, lista 10, flujo 12, pruebas 8). |
| **Complejidad** | Básica. |
| **Nivel Taxonómico de Bloom** | **Aplicar (nivel 3)**: configurar, ejecutar y comprobar un flujo. |
| **Licencia Requerida** | Microsoft 365 con derechos de Power Automate para conectores estándar, SharePoint Online y Exchange Online / Outlook de Microsoft 365. |
| **Plataformas Principales** | Power Automate (cloud vigente), SharePoint Online / Microsoft Lists y Office 365 Outlook. |

---

## Descripción General

El equipo del proyecto interno **PRJ-ATLAS** necesita conocer cuando una actividad se bloquea para actuar oportunamente. Aún no dispone de una notificación automatizada: el coordinador consulta la lista de seguimiento manualmente y puede no detectar un bloqueo. En este primer laboratorio crearás la fuente común que se reutilizará para el reporte del laboratorio 2.

Crearás una lista real en SharePoint con seis actividades, un flujo de nube que se active cuando cambie un elemento y una condición que envíe un correo solamente cuando el estado del elemento sea `Bloqueada`. Comprobarás el camino afirmativo, el negativo y la información que llega al mensaje, sin servicios Premium.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] **Preparar el origen de datos**, creando una lista SharePoint con los campos y registros requeridos para el seguimiento.
- [ ] **Configurar la automatización**, conectando un disparador, una condición de estado y una acción de correo electrónico.
- [ ] **Validar el comportamiento**, identificando en el historial cuando corresponde enviar un aviso y cuando debe omitirse.

---

## Prerrequisitos

### Conocimientos Previos

- Usar listas, archivos y correo de Microsoft 365 a nivel básico.
- Reconocer estados de actividades de un proyecto; no se requiere programación.

### Requisitos de Acceso y Entorno

- Disponer de una cuenta de trabajo/educativa de Microsoft 365 con permiso **Editar** en un **sitio SharePoint exclusivo de prácticas**, y permiso para **crear listas** en ese sitio. No usar un sitio productivo.
- Acceder a [Power Automate](https://make.powerautomate.com/) y verificar que se pueden crear flujos de nube con los conectores estándar SharePoint y Office 365 Outlook.
- Acceder a [Outlook](https://outlook.office.com/) con un buzón de prueba autorizado (puede ser el propio buzón corporativo). Confirmar que puede enviarse y recibirse un correo de prueba.
- Utilizar Edge o Chrome actualizado, conexión a internet y ventanas emergentes de Microsoft 365 permitidas. No se requieren máquinas virtuales, CLI, claves ni licencias Premium.
- Identificar la URL de tu sitio SharePoint y el correo de prueba; son valores **propios de tu tenant**, no datos que deban inventarse o publicarse.

---

## Entorno de Laboratorio

### Especificaciones de Software y Servicios Cloud

| Herramienta / Servicio | Versión / Release | Enlace Oficial / Origen | Licencia / Permisos |
| :--- | :--- | :--- | :--- |
| Power Automate | Servicio cloud vigente; flujo de nube automatizado. | [Portal](https://make.powerautomate.com/) | Derecho de crear flujos con conectores estándar; conexiones habilitadas por políticas del tenant. |
| SharePoint Online | Servicio Microsoft 365 vigente. | [Conector y disparadores](https://learn.microsoft.com/en-us/connectors/sharepointonline/) | Permiso para crear lista, columnas y elementos en sitio de práctica. |
| Office 365 Outlook | Conector `Send an email (V2)` vigente. | [Conector oficial](https://learn.microsoft.com/en-us/connectors/office365/) | Buzón Exchange Online de prueba y conexión autorizada. |

**Datos constantes de este laboratorio**

| Parámetro / Campo | Valor / Configuración |
| :--- | :--- |
| Proyecto | `PRJ-ATLAS` |
| Lista SharePoint | `SeguimientoActividades` |
| Flujo | `ATLAS_01_Aviso_Bloqueo` |
| Columna para la condición | `Estado` (tipo **Elección**, opción `Bloqueada`) |
| Correo de destino | Correo de prueba autorizado, no una dirección ficticia de ejemplo. |

---

## Instrucciones Paso a Paso

### Paso 1: Comprobar los permisos y las conexiones (5 min)

**Objetivo:** Asegurar que el entorno permite crear un flujo con acciones estándar antes de modificar datos.

1. Abrir el sitio SharePoint de práctica y comprobar que se dispone de **Contenido del sitio** y de la acción **Nuevo > Lista** (o la opción equivalente en Microsoft Lists).
2. Abrir [Power Automate](https://make.powerautomate.com/) con la misma cuenta; seleccionar el entorno de formacion autorizado y localizar **Crear**.
3. Abrir **Conexiones** y verificar o crear las conexiones **SharePoint** y **Office 365 Outlook**. Autorizarlas con la identidad corporativa que accede al sitio y al buzón.
4. Abrir Outlook y comprobar que el buzón receptor está disponible. Conservar sus datos para los pasos siguientes.

> **Nota técnica:** No usar el entorno de producción del cliente. Si la política DLP bloquea una conexión, detener la práctica y solicitar habilitación del entorno al administrador; no reemplazar el conector con uno Premium.

**Resultado esperado:** Sitio accesible, conectores autorizados y correo de prueba identificado.

**Comprobación:** Power Automate muestra las conexiones disponibles y SharePoint permite crear una lista.

---

### Paso 2: Crear la lista de seguimiento de Atlas (10 min)

**Objetivo:** Preparar un conjunto de datos que se pueda modificar y reutilizar en el laboratorio siguiente.

1. Seleccionar en SharePoint **Contenido del sitio > Nuevo > Lista > Lista en blanco**; nombrar la lista `SeguimientoActividades` y crearla en el sitio de prácticas.
2. Conservar la columna predeterminada **Título** (`Title` a nivel técnico). Agregar las columnas siguientes, escribiendo los nombres **sin espacios** para mantener los nombres internos:

   | Columna | Tipo | Configuración |
   | :--- | :--- | :--- |
   | `Proyecto` | Una línea de texto (Texto) | Requerido. |
   | `Estado` | Elección (una opción) | `Pendiente`; `En progreso`; `Bloqueada`; `Completada`. |
   | `FechaCompromiso` | Fecha y hora | Solo fecha. |
   | `Comentarios` | Varias líneas de texto | Texto libre. |

3. Seleccionar **Editar en vista de cuadrícula** o **Nuevo** e introducir los seis elementos. Para todas las filas, escribir `PRJ-ATLAS` en `Proyecto`:

   | Título | Estado | FechaCompromiso | Comentarios |
   | :--- | :--- | :--- | :--- |
   | Actualizar matriz de riesgos | En progreso | 2026-11-04 | Consolidación de riesgos operativos. |
   | Validar accesos del entorno QA | En progreso | 2026-11-05 | Pendiente de autorización TI. |
   | Revisar criterios de entrega | Pendiente | 2026-11-06 | Revisión con responsable funcional. |
   | Documentar pruebas de regresión | En progreso | 2026-11-09 | Preparar casos QA. |
   | Aprobar plan de liberación | Bloqueada | 2026-11-10 | Falta conformidad de coordinación. |
   | Publicar minuta de seguimiento | Completada | 2026-10-23 | Minuta distribuida al equipo. |

4. Salir de la vista de cuadrícula para confirmar el guardado. Comprobar que hay **6 elementos** y que solo **1** comienza como `Bloqueada`.

> **Nota técnica:** Las fechas y nombres corresponden a un caso de práctica. No cambiar el nombre de la columna `Estado` después de crearla: las expresiones dependen de su nombre interno.

**Resultado esperado:** La lista `SeguimientoActividades` contiene seis registros y las cuatro columnas de trabajo.

**Comprobación:** Abrir la vista **Todos los elementos**, verificar el conteo y localizar `Validar accesos del entorno QA` inicialmente en `En progreso`.

---

### Paso 3: Crear la automatización condicional (12 min)

**Objetivo:** Detectar cambios de estado y notificar los bloqueos por Outlook.

1. Seleccionar en Power Automate **Crear > Flujo de nube automatizado**, asignar el nombre `ATLAS_01_Aviso_Bloqueo` y elegir **SharePoint - When an item is created or modified** (cuando se crea o modifica un elemento).
2. Configurar **Site Address** con el sitio de práctica y **List Name** con `SeguimientoActividades`.
3. Agregar **Control > Condition**. Seleccionar como primer valor el contenido dinámico **Estado Value** / **Valor de Estado** del disparador, operador **is equal to**, y como segundo valor el texto exacto `Bloqueada`.
4. Agregar dentro de **If yes / Si verdadero** **Office 365 Outlook > Send an email (V2)**.
5. Completar la acción de correo usando el **selector de contenido dinámico** para cada campo de SharePoint; no escribir manualmente las llaves ilustrativas:

   | Campo | Valor / Configuración |
   | :--- | :--- |
   | **To / Para** | Buzón de prueba autorizado. (Correo electrónico que está usando) |
   | **Subject / Asunto** | `ATLAS, Actividad bloqueada: ` + **Title / Título**. |
   | **Body / Cuerpo** | `Proyecto: ` + **Proyecto**; `Actividad: ` + **Title**; `Estado: ` + **Estado Value**; `Observaciones: ` + **Comentarios**, cada uno en línea propia. |

6. Dejar la rama **If no / Si falso** sin acciones. Guardar el flujo y revisar que las tarjetas no muestren errores.

> **Nota técnica:** Si no aparece el token **Estado Value**, guardar y ejecutar el flujo con un cambio de prueba para inspeccionar la salida del disparador; utilizar la propiedad **Value** únicamente si la salida muestra un objeto de elección. Si devuelve texto plano, comparar ese texto directamente. El nombre del campo puede variar por idioma del diseñador; no comparar un objeto completo contra el texto `Bloqueada`.

**Resultado esperado:** Un flujo guardado con disparador SharePoint, condición de elección y una única acción de Outlook en la rama verdadera.

**Comprobación:** Abrir el resumen del flujo y verificar que se guardó correctamente, que está habilitado y que no solicita un conector Premium.

---

### Paso 4: Ejecutar y observar dos comportamientos (8 min)

**Objetivo:** Demostrar con ejecuciones reales que el aviso se envía solamente al cumplirse la condición.

1. Abrir en SharePoint `Validar accesos del entorno QA` y cambiar **Estado** de `En progreso` a `Bloqueada`; actualizar `Comentarios` a `Acceso QA aún no autorizado por TI.`; guardar.
2. Abrir en Power Automate el **historial de ejecuciones** de `ATLAS_01_Aviso_Bloqueo` y seleccionar la ejecución correspondiente al elemento modificado. Verificar **Condition = true** y **Send an email (V2) = Succeeded**.
3. Localizar en Outlook el mensaje `ATLAS | Actividad bloqueada: Validar accesos del entorno QA` y cotejar los cuatro campos con el registro de SharePoint.
4. Cambiar en SharePoint `Revisar criterios de entrega` de `Pendiente` a `En progreso` y guardar; localizar la ejecución correspondiente en Power Automate.
5. Confirmar **Condition = false** y que el paso Outlook de la rama verdadera no se ejecutó para la segunda modificación.

**Resultado esperado:** La modificación que establece `Bloqueada` genera una notificación y la modificación que deja el elemento `En progreso` no envía correo. El historial permite distinguir ambas ejecuciones.

**Comprobación:** Mostrar el historial de ambas ejecuciones y localizar el correo de la actividad bloqueada; cotejar título, proyecto, estado y comentarios con la lista. No utilizar únicamente la llegada del correo como prueba de que la condición funciona.

---

## Validación y Pruebas

### Ejecución de Prueba Básica

1. Comprobar que la lista tiene seis elementos y que `Validar accesos del entorno QA` termina con estado `Bloqueada`.
2. Verificar que el historial contiene la rama **verdadera** para el cambio de bloqueo y la **falsa** para la actualización a `En progreso`.
3. Contrastar asunto, proyecto y comentarios del correo con el elemento afectado.
4. Confirmar que la lista y el flujo pueden seguir utilizandose en el laboratorio 2.

### Pruebas de Resiliencia y Casos Adversos (Adversarial Testing)

Estas comprobaciones reutilizan los cambios realizados y evitan ampliar el alcance de la práctica.

#### Casos de Prueba Lógica y de Datos

| Escenario | Acción de Prueba | Resultado Esperado en la Validación |
| :--- | :--- | :--- |
| **Estado no crítico** | Cambiar `Revisar criterios de entrega` a `En progreso`. | La condición es falsa y no se ejecuta Outlook. |
| **Texto del comentario modificado** | Revisar el correo del elemento bloqueado tras actualizar el comentario. | El mensaje muestra el comentario **actual** y no uno inventado. |
| **Edición repetida de un registro bloqueado** | Si se prueba una nueva modificación de `Comentarios` manteniendo `Bloqueada`, consultar historial. | Puede llegar **otro correo**: el disparador no implementa deduplicacion. No registrar esto como defecto funcional del temario. |

---

## Solución de Problemas

### Problema 1: No aparece una ejecución tras modificar la lista

- **Síntomas:** El flujo está guardado pero no se observa una nueva ejecución.
- **Causa Raíz:** Dirección de sitio o lista incorrecta, flujo desactivado, permisos insuficientes o retardo del disparador.
- **Solución:**
  1. Abrir detalles del flujo y confirmar estado **Activado**.
  2. Verificar **Site Address**, **List Name** y conexión SharePoint.
  3. Guardar un cambio adicional en un elemento de prueba y revisar nuevamente el historial, sin asumir un tiempo fijo de latencia.

### Problema 2: La condición siempre es falsa

- **Síntomas:** El registro está bloqueado, pero el flujo toma la rama `If no`.
- **Causa Raíz:** Se ha comparado el objeto de elección completo, o el texto no coincide exactamente.
- **Solución:**
  1. Inspeccionar la salida del disparador en la ejecución.
  2. Seleccionar el contenido dinámico **Estado Value** y comparar con `Bloqueada`.
  3. Guardar, volver a modificar el elemento y comprobar la rama correcta.

### Problema 3: Falla `Send an email (V2)`

- **Síntomas:** El flujo llega a la rama verdadera y el envío falla o la bandeja no recibe el mensaje.
- **Causa Raíz:** Conexión de Outlook caducada, destinatario no autorizado o restricciones del tenant.
- **Solución:**
  1. Inspeccionar el detalle del error en el historial y actualizar la conexión Outlook si se solicita.
  2. Confirmar que el destinatario es un buzón real autorizado del entorno, no un dominio de ejemplo.
  3. Ejecutar otra modificación de prueba y revisar Bandeja de entrada y correo no deseado.

---

## Limpieza

Una vez validados los resultados:

1. **Conservar** la lista, sus seis elementos y el estado bloqueado de `Validar accesos del entorno QA`: son el origen obligatorio del laboratorio 2.
2. **Conservar** el flujo para poder consultar su historial; no modificar registros de producción.
3. Eliminar únicamente correos de prueba innecesarios si la política del entorno lo permite. Al finalizar los cuatro laboratorios, el propietario del sitio podrá borrar los recursos de práctica con autorización.

---

## Resumen

En este laboratorio has aprendido a:

1. **Estructurar datos de seguimiento:** Crear una lista SharePoint adecuada para una automatización.
2. **Automatizar avisos condicionados:** Conectar un disparador de cambios con una evaluación de estado y un correo de Outlook.
3. **Verificar resultados:** Interpretar ramas e historial sin sustituir pruebas por suposiciones.


