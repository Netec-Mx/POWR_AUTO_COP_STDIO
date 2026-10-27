# Crear un flujo básico de seguimiento que detecte un cambio en una lista de SharePoint, evalúe el estado de una actividad y envíe una notificación automática por correo.

## Metadatos

| Parámetro | Detalle |
| :--- | :--- |
| **Duración** | 35 minutos |
| **Nivel de Complejidad** | Fácil (Easy) |
| **Nivel de Taxonomía de Bloom** | Aplicar |
| **Tecnologías Principal** | Power Automate Cloud (Conectores estándar M365), SharePoint Online |
| **Perfil de Rol Objetivo** | Project Manager (PM), Business Analyst (BA), Quality Assurance (QA) |

## Descripción General

En este laboratorio práctico, aprenderás a automatizar un proceso clave de seguimiento de actividades dentro del ciclo de vida de un proyecto de software o gestión operativa. Partiendo de las necesidades de gobernanza sin costo adicional de licencias Premium, construirás un flujo automatizado en la nube mediante Microsoft Power Automate. 

El flujo monitoreará la lista de SharePoint Online `SeguimientoTareas`, detectará modificaciones en los registros de actividades de proyecto, evaluará condicionalmente el valor de la columna de estado (identificando estados críticos como `Bloqueado` o `Completado`) y enviará una notificación por correo electrónico estructurada a través de Microsoft 365 Outlook dirigida al responsable de la tarea.

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] Configurar el disparador automatizado `Cuando un elemento se crea o se modifica` (*When an item is created or modified*) en SharePoint Online dentro de Power Automate.
- [ ] Implementar estructuras condicionales avanzadas (bloques *Condition*) para evaluar atributos de estado sin recurrir a conectores Premium ni código personalizado.
- [ ] Mapear contenido dinámico (*Dynamic Content*) desde la entidad de SharePoint hacia el conector estándar de Office 365 Outlook para construir correos informativos estructurados.
- [ ] Validar la ejecución del flujo mediante pruebas de integración en vivo y diagnósticos en el historial de ejecuciones de Power Automate.

## Prerrequisitos

### Conocimientos Previos
- Comprensión básica de la interfaz de Microsoft 365 y navegación en sitios de SharePoint Online.
- Nivel conceptual sobre columnas y listas personalizadas en SharePoint.
- Familiaridad con el uso de correo electrónico corporativo (Outlook para la Web).

### Acceso y Permisos
- Cuenta de Microsoft 365 con licencia activa que incluya Power Automate para M365 y SharePoint Online.
- Permisos de edición o administración en el sitio de SharePoint del curso: `https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM`.
- Acceso al entorno de prueba de Power Automate (Power Automate Cloud Release 2024.05).

## Entorno de Laboratorio

### Software y Servicios Requeridos

| Software / Servicio | Versión / Release | Enlace Oficial de Referencia |
| :--- | :--- | :--- |
| **Microsoft Power Automate (Cloud)** | Cloud Release 2024.05 | [https://learn.microsoft.com/es-es/power-automate/](https://learn.microsoft.com/es-es/power-automate/) |
| **SharePoint Online** | Cloud Release 2024.05 | [https://www.microsoft.com/es-es/microsoft-365/sharepoint/](https://www.microsoft.com/es-es/microsoft-365/sharepoint/) |
| **Microsoft 365 Outlook (Web App)** | Web App Release 2024.05 | [https://outlook.office.com/](https://outlook.office.com/) |
| **Navegador Web** | Microsoft Edge v124.0+ / Chrome v124.0+ | [https://www.microsoft.com/edge](https://www.microsoft.com/edge) |

### Constantes de Entorno Globales
- **Sitio SharePoint:** `https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM`
- **Lista de Tareas:** `SeguimientoTareas`
- **Conectores Utilizados:** Estándar M365 exclusivamente (`SharePoint`, `Office 365 Outlook`).

---

## Instrucciones Paso a Paso

### Paso 1: Verificar la Estructura de la Lista en SharePoint Online

**Objetivo:** Validar que la lista `SeguimientoTareas` contenga las columnas requeridas para registrar actividades y estados de proyecto.

1. Abre tu navegador web e ingresa a la URL del sitio de SharePoint del laboratorio:
   ```text
   https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM
   ```
2. En el menú de navegación izquierdo o en **Contenido del sitio**, selecciona la lista **SeguimientoTareas**.
3. Verifica que la lista disponga de las siguientes columnas con su respectivo tipo de dato:

   | Nombre de Columna | Tipo de Dato | Opciones / Valores requeridos |
   | :--- | :--- | :--- |
   | `Title` (Nombre de Actividad) | Línea de texto única | (Nombre por defecto) |
   | `Estado` | Elección (Choice) | `No iniciado`, `En progreso`, `Bloqueado`, `Completado` |
   | `Responsable` | Persona o Grupo (Person) | Usuario de la organización |
   | `Descripcion` | Varias líneas de texto | Texto libre |

4. Si la lista no existe o carece de la columna `Estado`, créala haciendo clic en **+ Agregar columna** > **Elección**, asigna el nombre `Estado` e introduce las opciones indicadas.

```text
Configuración de columna Estado:
- Nombre: Estado
- Tipo: Elección
- Opciones:
  * No iniciado
  * En progreso
  * Bloqueado
  * Completado
```

**Resultado esperado:** La lista `SeguimientoTareas` está correctamente estructurada y lista para recibir modificaciones de datos.

**Comprobación:** Asegúrate de que existe al menos un elemento de prueba en la lista. Si no existe ninguno, crea un elemento con el título `Revisión de Arquitectura Base`, Estado `En progreso` y asígnate a ti mismo en el campo `Responsable`.

---

### Paso 2: Crear el Flujo Automatizado de Nube en Power Automate

**Objetivo:** Crear un nuevo flujo en Power Automate en blanco iniciado por el disparador nativo de SharePoint.

1. Abre una nueva pestaña en el navegador e ingresa a [https://make.powerautomate.com](https://make.powerautomate.com).
2. En la barra lateral izquierda, verifica que estás en el entorno de trabajo asignado para el curso.
3. Haz clic en **+ Crear** (*+ Create*) y selecciona la opción **Flujo de nube automatizado** (*Automated cloud flow*).
4. En la ventana emergente de configuración inicial:
   - **Nombre del flujo:** `Flujo_Seguimiento_Tareas_[TuNombre]` (ejemplo: `Flujo_Seguimiento_Tareas_JuanPerez`).
   - **Elija el desencadenador del flujo:** En la barra de búsqueda escribe `SharePoint`.
   - Selecciona el disparador: **Cuando se crea o se modifica un elemento** (*When an item is created or modified*).
5. Haz clic en **Crear** (*Create*).

```text
Nombre del flujo: Flujo_Seguimiento_Tareas_[TuNombre]
Desencadenador: SharePoint - Cuando se crea o se modifica un elemento
```

**Resultado esperado:** El diseñador de Power Automate se abre mostrando el bloque desencadenador listo para configurar.

**Comprobación:** Confirma que el título del flujo aparece en la esquina superior izquierda de la pantalla.

---

### Paso 3: Configurar el Desencadenador de SharePoint

**Objetivo:** Conectar el disparador del flujo a la lista de SharePoint del proyecto.

1. Haz clic sobre la tarjeta del desencadenador **Cuando se crea o se modifica un elemento**.
2. En el panel de parámetros a la izquierda (o sobre la tarjeta):
   - **Dirección del sitio** (*Site Address*): Despliega la lista y selecciona `https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM` (o pega la URL exacta si no aparece en la lista desplegable).
   - **Nombre de la lista** (*List Name*): Selecciona `SeguimientoTareas`.

```text
Dirección del sitio: https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM
Nombre de la lista: SeguimientoTareas
```

3. Haz clic en el lienzo del diseñador para fijar los valores.

**Resultado esperado:** El desencadenador queda configurado adecuadamente con el sitio y lista correspondientes sin mostrar alertas rojas de validación.

**Comprobación:** Verifica que no aparezca el ícono de advertencia o error en la esquina del bloque desencadenador.

---

### Paso 4: Agregar y Configurar la Condición de Evaluación de Estado

**Objetivo:** Incorporar un bloque de control condicional para identificar si la actividad ha entrado en estado `Bloqueado` o `Completado`.

1. Haz clic en el botón **+ Nuevo paso** (*+ New step*) o en el ícono **+** debajo del desencadenador.
2. En la barra de búsqueda de acciones, escribe `Condición` (*Condition*) y selecciona la herramienta de control **Condición**.
3. En el bloque de lógica condicional:
   - Haz clic en el campo izquierdo **Elegir un valor** (*Choose a value*).
   - En la ventana emergente de **Contenido dinámico** (*Dynamic content*), busca y selecciona la propiedad `Estado Valor` (*Estado Value*) proveniente del paso de SharePoint.
   - En el operador central, deja la opción por defecto: **es igual a** (*is equal to*).
   - En el campo derecho **Elegir un valor** (*Choose a value*), escribe exactamente: `Bloqueado`.

```text
[Contenido Dinámico: Estado Valor]  |  es igual a  |  Bloqueado
```

4. *(Opcional - Estructura Adicional)*: Si deseas validar también el estado `Completado`, puedes cambiar el selector de la condición a **O** (*OR*) agregando una fila:
   - Fila 2: `[Estado Valor]` -> `es igual a` -> `Completado`.
   - Por simplicidad de esta práctica inicial, mantendremos la condición enfocada en detectar el estado crítico `Bloqueado`.

**Resultado esperado:** El diagrama ahora muestra una bifurcación lógica con dos ramas: **En caso afirmativo** (*If yes*) y **En caso negativo** (*If no*).

**Comprobación:** Asegúrate de haber seleccionado `Estado Valor` y no únicamente la propiedad `Estado` (que entrega un objeto en listas de elección de SharePoint).

---

### Paso 5: Configurar el Envío de Notificación por Correo (Rama Afirmativa)

**Objetivo:** Enviar un correo electrónico corporativo al responsable de la tarea cuando el estado sea `Bloqueado`, empleando el conector nativo de Office 365 Outlook.

1. Dentro de la caja de la rama **En caso afirmativo** (*If yes*), haz clic en **Agregar una acción** (*Add an action*).
2. En el buscador de acciones, escribe `Enviar correo electronico` y selecciona la acción **Enviar un correo electrónico (V2)** (*Send an email (V2)*) del conector **Office 365 Outlook**.
3. Si el sistema te pide autorizar la conexión, confirma con las credenciales activas de Microsoft 365.
4. Configura los parámetros del correo electrónico utilizando Contenido Dinámico:
   - **Para** (*To*): Haz clic en el campo, abre la pestaña de *Contenido dinámico* y selecciona **Responsable Email** (*Responsable Correo* o *Responsable Email*).
   - **Asunto** (*Subject*): Escribe exactamente:
     ```text
     [ALERTA TAREA BLOQUEADA] - Actividad: 
     ```
     A continuación del texto, inserta el contenido dinámico **Título** (*Title*).
   - **Cuerpo** (*Body*): Escribe el siguiente texto formateado en HTML/Texto estructurado e inserta los datos dinámicos correspondientes:

```text
Hola,

La siguiente actividad asignada a tu responsabilidad ha cambiado su estado a BLOQUEADO y requiere atención inmediata.

Detalles de la Actividad:
--------------------------------------------------
- Nombre de la Tarea: [Título]
- Estado Actual: [Estado Valor]
- Modificado por: [Modificado por DisplayName]
- Descripción: [Descripcion]

Por favor, ingresa a la lista de SharePoint del proyecto para resolver los bloqueos registrados.

Atentamente,
Oficina de Gestión de Proyectos (PMO)
```

*(Nota: Reemplaza cada etiqueta entre corchetes `[...]` seleccionando la variable correspondiente de la ventana de Contenido Dinámico).*

```text
Parámetros finales de la acción Enviar correo electrónico (V2):
- To: Responsable Email
- Subject: [ALERTA TAREA BLOQUEADA] - Actividad: [Title]
- Body: Incluye el texto de plantilla formateado con contenido dinámico.
```

5. En la rama **En caso negativo** (*If no*), no agregaremos ninguna acción, ya que no se requiere notificar cambios a estados normales en esta práctica.

**Resultado esperado:** La rama afirmativa contiene la acción de envío de correo electrónico completamente mapeada con variables del elemento de SharePoint.

**Comprobación:** Revisa que el campo `Para` (*To*) contenga una dirección válida proveniente del objeto de dynamic content (`Responsable Email`).

---

### Paso 6: Guardar y Probar el Flujo Automatizado

**Objetivo:** Ejecutar la verificación lógica e iniciar una prueba funcional en vivo interactuando con SharePoint.

1. En la parte superior derecha del diseñador, haz clic en **Guardar** (*Save*).
2. Espera a que aparezca la barra verde indicando: *"El flujo está listo para usarse"*.
3. Haz clic en **Comprobador de flujos** (*Flow Checker*) y confirma que no haya errores ni advertencias de rendimiento.
4. Haz clic en el botón **Probar** (*Test*) en la esquina superior derecha.
5. Selecciona la opción **Manualmente** (*Manually*) y haz clic en **Probar** (*Test*). El flujo entrará en estado de espera del disparador.

```text
Estado de la prueba: Esperando que se produzca el evento en SharePoint.
```

6. Vuelve a la pestaña de SharePoint Online donde tienes abierta la lista **SeguimientoTareas**.
7. Selecciona el elemento de prueba `Revisión de Arquitectura Base` (o crea uno nuevo) y edita sus propiedades:
   - Cambia la columna **Estado** al valor `Bloqueado`.
   - Asegúrate de que el campo **Responsable** tenga asignada tu propia cuenta de correo para poder recibir la notificación.
   - Haz clic en **Guardar**.

8. Regresa a la pestaña de Power Automate y observa el seguimiento de ejecución.

**Resultado esperado:** En un tiempo de 10 a 30 segundos, la pantalla de Power Automate se actualizará mostrando verde en todos los bloques del flujo (*"Su flujo se ejecutó correctamente"*).

**Comprobación:** Revisa tu bandeja de entrada en Microsoft Outlook Web App. Debes haber recibido un correo con el asunto `[ALERTA TAREA BLOQUEADA] - Actividad: Revisión de Arquitectura Base` y los detalles extraídos dinámicamente.

---

## Validación y Pruebas

Para validar la solidez del flujo y descartar comportamientos no deseados o fallos de ejecución, realiza las siguientes pruebas de verificación:

### Caso de Prueba 1: Notificación Exitosa ante Estado Bloqueado (Camino Feliz)
- **Acción:** Modificar un elemento existente en SharePoint cambiando su estado de `En progreso` a `Bloqueado`.
- **Resultado Esperado en Power Automate:** El flujo se dispara, evalúa la condición como `True` (Verdadero) y ejecuta la rama de envío de correo.
- **Evidencia requerida:** Detalle de ejecución con marcas de verificación verdes y recepción del correo en la bandeja de entrada de Outlook.

### Caso de Prueba 2: Filtrado Correcto ante Estados No Críticos (Prueba Negativa)
- **Acción:** Crear un nuevo elemento en la lista con el título `Documentación de Requisitos` y estado `En progreso`.
- **Resultado Esperado en Power Automate:** El flujo se dispara correctamente, pero en el bloque condicional toma la ruta `False` (En caso negativo), finalizando la ejecución sin enviar correos electrónicos.
- **Evidencia requerida:** Historial de ejecución mostrando la rama *If no* ejecutada y ausencia de nuevos correos.

### Caso de Prueba 3 (Adversarial / Casos Limite): Elemento Modificado Sin Campo Responsable Asignado
- **Acción:** Crear o modificar una tarea en la lista, cambiar el Estado a `Bloqueado`, pero dejar el campo `Responsable` completamente vacío (Nulo).
- **Resultado Esperado en Power Automate:** El flujo se ejecuta, la condición es evaluada como `True`, pero la acción de Outlook fallará con un error `400 Bad Request` indicando que el parámetro `To` es requerido o nulo (`The input parameter 'To' cannot be null or empty`).
- **Análisis del Aprendizaje:** Este caso demuestra la necesidad de incorporar validaciones preventivas antes del envío de notificaciones (por ejemplo, verificar si el responsable no está nulo) en diseños de producción.

---

## Solución de Problemas

### Problema 1: El flujo no se dispara tras modificar un registro en SharePoint

- **Sintoma:** Transcurren varios minutos tras guardar un cambio en la lista `SeguimientoTareas`, pero no aparece ninguna nueva ejecución en el historial de Power Automate.
- **Causa Posible:** 
  1. El disparador `Cuando un elemento se crea o se modifica` funciona mediante inspección periódica (*polling*), lo que en licencias de prueba o estándar puede tardar entre 1 y 5 minutos.
  2. La URL del sitio o el nombre de la lista fueron seleccionados manualmente con errores tipográficos.
- **Solución:**
  1. Aguarda hasta 5 minutos o fuerza la activación haciendo una modificación clara en una columna de texto.
  2. Vuelve al diseñador del flujo, edita la tarjeta del desencadenador y reselecciona el sitio y la lista utilizando directamente el menú desplegable oficial de SharePoint en lugar de escribir la dirección de texto manualmente.

---

### Problema 2: El correo no se envía y la condición evalúa como Falsa a pesar de seleccionar "Bloqueado"

- **Símoma:** En el historial de ejecución, la condición entra por la rama **En caso negativo** (*If no*), aunque la lista de SharePoint muestra visiblemente el estado `Bloqueado`.
- **Causa Posible:** Se seleccionó la variable de contenido dinámico incorrecta. En SharePoint, las columnas de tipo Elección (Choice) exponen dos propiedades: `Estado` (que entrega el objeto o arreglo de metadatos) y `Estado Valor` (que entrega la cadena de texto con la opción elegida). Si se utiliza `Estado`, la comparación de cadenas falla.
- **Solución:**
  1. Edita el flujo y elimina la variable asignada a la izquierda del bloque condicional.
  2. En el panel de contenido dinámico, busca específicamente la propiedad **Estado Valor** (*Estado Value*).
  3. Asegúrate de que no existan espacios en blanco adicionales al inicio o final de la palabra `Bloqueado` en el campo de comparación.

---

## Limpieza

Una vez concluidas y validadas las pruebas del laboratorio, realiza los siguientes pasos de ordenamiento en tu entorno:

1. **Desactivar o Eliminar el Flujo de Prueba:**
   - Navega a **Mis flujos** (*My flows*) en Power Automate.
   - Ubica `Flujo_Seguimiento_Tareas_[TuNombre]`.
   - Haz clic en el ícono de los tres puntos (**...**) y selecciona **Desactivar** (*Turn off*) para evitar que continúe ejecutándose durante otros laboratorios del curso.
2. **Restaurar Datos en SharePoint:**
   - Ingresa a la lista **SeguimientoTareas**.
   - Elimina los elementos de prueba creados durante las actividades de validación funcional para mantener limpia la lista de trabajo.

---

## Resumen

En esta práctica completaste la construcción de un flujo de trabajo automatizado de nube orientado al seguimiento de proyectos sin costo de licencias Premium:

- Conectaste exitosamente **Power Automate Cloud** con **SharePoint Online** mediante el disparador nativo de modificación de elementos.
- Implementaste una regla de negocio estructurada mediante el control de **Condición** para filtrar eventos según el estado operativo de una tarea (`Bloqueado`).
- Mapeaste variables de **Contenido Dinámico** hacia el conector estándar de **Office 365 Outlook**, notificando de forma automática y personalizada al responsable del entregable.
- Ejecutaste pruebas funcionales en vivo y analizaste casos límite de datos faltantes en la orquestación.

### Recursos Adicionales
- [Documentación oficial del conector de SharePoint en Power Automate](https://learn.microsoft.com/es-es/connectors/sharepointonline/)
- [Uso de condiciones en flujos de nube de Power Automate](https://learn.microsoft.com/es-es/power-automate/add-condition)
- [Guía de límites y configuraciones en conectores estándar de M365](https://learn.microsoft.com/es-es/power-automate/limits-and-config)
