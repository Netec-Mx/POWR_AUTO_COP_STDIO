# Consolidar información de una lista de SharePoint y un archivo de Excel, filtrar actividades relevantes, generar un resumen de estado con contenido estructurado para reporte o presentación y distribuirlo automáticamente por correo.

## Metadatos

| Atributo | Detalle |
| :--- | :--- |
| **Duración** | 65 minutos |
| **Complejidad** | Intermedia (*Medium*) |
| **Nivel Taxonómico de Bloom** | Crear (*Create*) |
| **Licencia Requerida** | Microsoft 365 E3/E5 (Trial o Tenancy Educativo) con soporte para Power Automate Cloud Estándar |
| **Plataformas Principales** | Power Automate Cloud (Release 2024.05), SharePoint Online (Release 2024.05), Excel Online Business (Release 2024.05), Office 365 Outlook (Release 2024.05) |

---

## Descripción General

En este laboratorio, el estudiante consolidará datos provenientes de dos fuentes heterogéneas sin utilizar conectores Premium ni servicios de pago: la lista de SharePoint Online `SeguimientoTareas` (creada en la Práctica 1) y un nuevo archivo de Excel Online llamado `RecursosProyecto.xlsx` almacenado en OneDrive for Business / SharePoint. Este último contiene métricas financieras, niveles de riesgo e información simulada proveniente de sistemas de gestión ALM (como Jira o Polarion) exportada en formato estructurado.

El estudiante construirá un flujo de nube automatizado en Power Automate que extrae la información de ambas fuentes, aplica filtros de datos para aislar las actividades críticas o en estado de riesgo (`En progreso`, `Bloqueado` o con `NivelRiesgo` Alto), procesa la información mediante acciones de datos para construir una tabla HTML dinámicamente formateada y envía un reporte ejecutivo automatizado por medio de Office 365 Outlook a la dirección del proyecto.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] **Consolidar información de una lista de SharePoint y un archivo de Excel**, asociando registros mediante identificadores comunes mediante conectores estándar de Microsoft 365.
- [ ] **Utilizar las acciones 'Obtener elementos' de SharePoint y 'Listar las filas presentes en una tabla' de Excel Online (Business)** de forma optimizada y filtrada.
- [ ] **Filtrar actividades relevantes** aplicando condiciones lógicas en Power Automate para aislar tareas con desviaciones o riesgos críticos.
- [ ] **Construir tablas HTML dinámicas y resúmenes ejecutivos** combinando múltiples fuentes de datos estructuradas con formato CSS integrado.
- [ ] **Distribuir automáticamente por correo electrónico** el informe consolidado a los interesados del proyecto manteniendo la integridad visual y de datos.

---

## Prerrequisitos

### Conocimientos Previos
- Comprensión de esquemas de datos estructurados (`ListObject` en Excel y columnas fuertemente tipadas en SharePoint Online).
- Conocimientos básicos sobre la interfaz y el diseñador de flujos en Microsoft Power Automate Cloud.
- Manejo de contenido dinámico (*Dynamic Content*) y expresiones básicas en Power Automate.

### Requisitos de Acceso y Entorno
- Completar satisfactoriamente el **Lab 01-00-01** (donde se creó y pobló la lista `SeguimientoTareas`).
- Cuenta de usuario de Microsoft 365 con licencias activas de SharePoint Online, OneDrive for Business, Excel Online Business y Office 365 Outlook.
- Navegador web moderno (Microsoft Edge v124.0.2478.80 o Google Chrome v124.0.6367.78).
- Acceso al sitio de SharePoint de trabajo: `https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM`.

---

## Entorno de Laboratorio

### Especificaciones de Software y Servicios Cloud

| Herramienta / Servicio | Versión / Release | Enlace Oficial / Origen | Licencia / Permisos |
| :--- | :--- | :--- | :--- |
| **Microsoft Power Automate** | Cloud Release 2024.05 | [https://make.powerautomate.com](https://make.powerautomate.com) | Standard M365 (Sin conectores Premium) |
| **SharePoint Online** | Web App Release 2024.05 | `https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM` | Permiso de Propietario o Integrante de Sitio |
| **Excel Online (Business)** | Web App Release 2024.05 | [https://www.office.com/launch/excel](https://www.office.com/launch/excel) | Estándar M365 |
| **Office 365 Outlook** | Web App Release 2024.05 | [https://outlook.office.com](https://outlook.office.com) | Buzón de correo habilitado para envío interno |

---

## Instrucciones Paso a Paso

### Paso 1: Crear y estructurar el archivo Excel de asignación de recursos en OneDrive for Business

**Objetivo:** Crear el archivo `RecursosProyecto.xlsx` que simula la información de costos, esfuerzo y riesgos exportada de herramientas ALM (Jira/Polarion), delimitando los datos dentro de una Tabla de Excel oficial (`ListObject`).

1. Abre una nueva pestaña en el navegador e ingresa a [Microsoft 365 OneDrive](https://www.office.com/launch/onedrive) o accede a la biblioteca de documentos **DocumentosCompartidos** del sitio `https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM`.
2. Haz clic en **+ Nuevo** (*+ New*) > **Libro de Excel** (*Excel workbook*).
3. En la barra superior de Excel Online, renombra el archivo haciendo clic en el título por defecto y escribe exactamente:
   ```
   RecursosProyecto.xlsx
   ```
4. En la hoja de cálculo (`Hoja1`), introduce la siguiente estructura de datos a partir de la celda **A1**:

   | Celda A1 | Celda B1 | Celda C1 | Celda D1 | Celda E1 |
   | :--- | :--- | :--- | :--- | :--- |
   | `ID_Tarea` | `CostoEstimado` | `HorasAsignadas` | `NivelRiesgo` | `HerramientaOrigen` |
   | 1 | 1500 | 40 | Medio | Jira Export |
   | 2 | 3200 | 85 | Alto | Polarion Export |
   | 3 | 800 | 20 | Bajo | SharePoint Direct |
   | 4 | 2400 | 60 | Alto | Jira Export |
   | 5 | 1100 | 30 | Bajo | Jira Export |

   > **Nota de Integridad:** El campo `ID_Tarea` debe coincidir numéricamente con los identificadores primarios (`ID`) de los elementos presentes en la lista de SharePoint `SeguimientoTareas`.

5. Selecciona todo el rango de datos poblado (desde **A1** hasta **E6**).
6. En la cinta de opciones superior, ve a la pestaña **Insertar** (*Insert*) y haz clic en **Tabla** (*Table*) (o presiona `Ctrl + T`).
7. En el cuadro de diálogo emergente, asegúrate de marcar la casilla **La tabla tiene encabezados** (*My table has headers*) y haz clic en **Aceptar**.
8. Con la tabla seleccionada, dirígete a la pestaña **Diseño de tabla** (*Table Design*) en la cinta superior. En el campo **Nombre de la tabla** (*Table Name*) en el extremo izquierdo, cambia el nombre predeterminado por:
   ```
   tbl_Recursos
   ```
9. Verifica que los cambios se hayan guardado automáticamente en la nube (el indicador en la barra superior debe mostrar "Guardado"). Cierra la pestaña de Excel Online.

**Resultado esperado:** El archivo `RecursosProyecto.xlsx` existe en la biblioteca de documentos o OneDrive, conteniendo la tabla estructurada `tbl_Recursos` con 5 filas de datos técnicos y financieros.

**Comprobación:** Vuelve a abrir el archivo `RecursosProyecto.xlsx`, haz clic sobre cualquier celda con datos y confirma en la pestaña **Diseño de tabla** que el nombre asignado es exactamente `tbl_Recursos`.

---

### Paso 2: Crear el Flujo Automatizado / Instantáneo en Power Automate

**Objetivo:** Inicializar un nuevo flujo de nube en Power Automate para orquestar la extracción y consolidación de datos.

1. Navega a [https://make.powerautomate.com](https://make.powerautomate.com).
2. Asegúrate de estar posicionado en el entorno de trabajo asignado para el curso.
3. En el menú de navegación izquierdo, haz clic en **+ Crear** (*+ Create*).
4. Selecciona la opción **Flujo de nube instantáneo** (*Instant cloud flow*).
5. En el cuadro de diálogo de configuración inicial:
   - **Nombre del flujo:** `Flujo_Consolidado_Reporte_[TuNombre]` (ejemplo: `Flujo_Consolidado_Reporte_JuanPerez`).
   - **Elija cómo desencadenar este flujo:** Selecciona **Desencadenar un flujo manualmente** (*Manually trigger a flow*).
6. Haz clic en **Crear** (*Create*).

**Resultado esperado:** El diseñador de Power Automate se abre mostrando el disparador manual en el lienzo.

**Comprobación:** El nombre del flujo aparece correctamente registrado en la esquina superior izquierda de la pantalla.

---

### Paso 3: Extraer los elementos de la lista de SharePoint

**Objetivo:** Configurar la acción de SharePoint para consultar los registros de la lista `SeguimientoTareas`.

1. En el diseñador de Power Automate, haz clic en el botón **+ Nuevo paso** (*+ New step*) o en el icono **+** debajo del disparador manual.
2. En la barra de búsqueda de acciones, escribe `SharePoint` y selecciona el conector de **SharePoint**.
3. En la lista de acciones, busca y selecciona **Obtener elementos** (*Get items*).
4. Configura los parámetros de la tarjeta de la siguiente manera:
   - **Dirección del sitio** (*Site Address*): Selecciona de la lista desplegable o pega la URL: `https://[tenant].sharepoint.com/sites/CursoAutomatizacionPM`
   - **Nombre de la lista** (*List Name*): Selecciona `SeguimientoTareas`.
5. Haz clic en **Mostrar opciones avanzadas** (*Show advanced options*) para expandir los filtros de consulta:
   - **Consulta de filtro** (*Filter Query*): Escribe la siguiente sintaxis OData para obtener únicamente las actividades que requieren atención inmediata:
     ```text
     Estado eq 'En progreso' or Estado eq 'Bloqueado' or Estado eq 'No iniciado'
     ```
6. Renombra la acción para mantener las buenas prácticas: haz clic en los tres puntos (`...`) de la tarjeta > **Cambiar nombre** (*Rename*) y asigna:
   ```
   Obtener_Tareas_SharePoint
   ```

**Resultado esperado:** La acción queda configurada para recuperar los elementos de SharePoint que cumplan con la condición de estado.

**Comprobación:** Confirma que la sintaxis OData no contenga comillas tipográficas (`“ ”`) sino comillas simples rectas (`' '`).

---

### Paso 4: Extraer la información de recursos del archivo Excel Online

**Objetivo:** Recuperar la lista de filas de la tabla estructurada `tbl_Recursos` almacenada en el archivo `RecursosProyecto.xlsx`.

1. Haz clic en **+ Nuevo paso** (*+ New step*) debajo de la acción `Obtener_Tareas_SharePoint`.
2. En la barra de búsqueda escribe `Excel Online (Business)`.
3. Selecciona la acción **Listar las filas presentes en una tabla** (*List rows present in a table*).
4. Configura los parámetros de la tarjeta de la siguiente manera:
   - **Ubicación** (*Location*): Selecciona `OneDrive for Business` (o el sitio de SharePoint `CursoAutomatizacionPM` según donde guardaste el archivo en el Paso 1).
   - **Biblioteca de documentos** (*Document Library*): Selecciona `OneDrive` (o `DocumentosCompartidos`).
   - **Archivo** (*File*): Haz clic en el icono de carpeta y navega hasta seleccionar `RecursosProyecto.xlsx` (o introduce la ruta `/RecursosProyecto.xlsx`).
   - **Tabla** (*Table*): Despliega la lista y selecciona `tbl_Recursos`.
5. Renombra la acción a:
   ```
   Obtener_Recursos_Excel
   ```

**Resultado esperado:** El conector de Excel se vincula a la tabla `tbl_Recursos` para retornar las métricas de costo y riesgo.

**Comprobación:** La opción **Tabla** debe mostrar la etiqueta `tbl_Recursos` de forma automática tras seleccionar el archivo.

---

### Paso 5: Filtrar y cruzar la información con una Operación de Datos

**Objetivo:** Consolidar en una matriz limpia la información obtenida de ambas fuentes y formatearla dentro de una tabla HTML estructurada.

1. Haz clic en **+ Nuevo paso** (*+ New step*).
2. En el buscador escribe `Operación de datos` (*Data Operation*) y selecciona la acción **Seleccionar** (*Select*).
3. Configura los parámetros del paso de transformación:
   - **Desde** (*From*): Haz clic en el campo e inserta la salida de contenido dinámico `value` perteneciente a la acción `Obtener_Tareas_SharePoint`.
   - **Mapa** (*Map*): En la tabla par Clave-Valor, define las columnas estructuradas de tu reporte:

     | Clave (Izquierda) | Valor (Derecha - Contenido Dinámico / Expresión) |
     | :--- | :--- |
     | `ID Tarea` | Selecciona el contenido dinámico `ID` de `Obtener_Tareas_SharePoint`. |
     | `Actividad` | Selecciona el contenido dinámico `Title` de `Obtener_Tareas_SharePoint`. |
     | `Estado Actual` | Selecciona el contenido dinámico `Estado Valor` (o `Estado`). |
     | `Responsable` | Selecciona el contenido dinámico `Responsable DisplayName` (o `Title` de la persona). |
     | `Descripción` | Selecciona el contenido dinámico `Descripcion`. |

4. Renombra la acción a:
   ```
   Mapear_Datos_Tareas
   ```

5. Haz clic en **+ Nuevo paso** (*+ New step*).
6. En la barra de búsqueda escribe `Operación de datos` (*Data Operation*) y selecciona **Crear tabla HTML** (*Create HTML table*).
7. En el campo **Desde** (*From*), selecciona la salida `Resultado` (*Output*) de la acción anterior `Mapear_Datos_Tareas`.
8. Haz clic en los tres puntos (`...`) de la tarjeta > **Cambiar nombre** (*Rename*) y nombra el paso como:
   ```
   Generar_Tabla_HTML
   ```

**Resultado esperado:** Las colecciones de datos JSON se transforman en una estructura tabular HTML limpia.

**Comprobación:** Asegúrate de que la entrada de `Generar_Tabla_HTML` provenga de la salida de la acción `Seleccionar` (*Mapear_Datos_Tareas*).

---

### Paso 6: Formatear la tabla HTML con un estilo CSS profesional (Componer)

**Objetivo:** Inyectar reglas CSS a la tabla HTML para asegurar una presentación clara y legible en el correo del cliente.

1. Haz clic en **+ Nuevo paso** (*+ New step*).
2. Selecciona la acción **Redactar** (*Compose*) del conector de **Operación de datos**.
3. Renombra la acción a:
   ```
   Estilar_Reporte_HTML
   ```
4. En el campo de texto **Entradas** (*Inputs*), copia y pega exactamente la siguiente plantilla de código HTML/CSS, la cual envuelve la salida dinámica de la tabla:

```html
<div style="font-family: Arial, sans-serif; color: #333333; max-width: 800px; margin: 0 auto;">
  <h2 style="color: #005A9E; border-bottom: 2px solid #005A9E; padding-bottom: 8px;">
    Informe de Estatus Consolidado del Proyecto
  </h2>
  <p>Estimado equipo directivo,</p>
  <p>A continuación se presenta el resumen consolidado de las actividades críticas y el estado de recursos del proyecto registrado en el repositorio oficial de SharePoint y Excel:</p>
  
  <style>
    table {
      border-collapse: collapse;
      width: 100%;
      margin-top: 15px;
      margin-bottom: 15px;
    }
    th {
      background-color: #005A9E;
      color: white;
      text-align: left;
      padding: 10px;
      font-size: 14px;
    }
    td {
      border: 1px solid #dddddd;
      padding: 8px;
      font-size: 13px;
    }
    tr:nth-child(even) {
      background-color: #f9f9f9;
    }
  </style>

  @{outputs('Generar_Tabla_HTML')}

  <div style="margin-top: 20px; padding: 12px; background-color: #F0F4F8; border-left: 4px solid #005A9E;">
    <strong>Nota de Gobernanza:</strong> Este reporte fue generado automáticamente por Power Automate Cloud integrando las fuentes <code>SeguimientoTareas</code> (SharePoint) y <code>RecursosProyecto.xlsx</code> (Excel Online).
  </div>
</div>
```

> **Nota técnica:** En el diseñador moderno de Power Automate, reemplaza la referencia `@{outputs('Generar_Tabla_HTML')}` seleccionando directamente la salida de la acción **Generar_Tabla_HTML** desde el menú de contenido dinámico.

**Resultado esperado:** El bloque `Redactar` une el código CSS de estilo con la tabla HTML dinámica generada previamente.

**Comprobación:** Confirma que no existan etiquetas sin cerrar en el código HTML.

---

### Paso 7: Configurar y enviar el reporte estructurado por correo electrónico en Outlook

**Objetivo:** Distribuir el informe consolidado por medio del conector estándar Office 365 Outlook.

1. Haz clic en **+ Nuevo paso** (*+ New step*).
2. En el buscador escribe `Office 365 Outlook`.
3. Selecciona la acción **Enviar un correo electrónico (V2)** (*Send an email (V2)*).
4. Configura los parámetros del correo electrónico:
   - **A** (*To*): Introduce tu propia dirección de correo institucional/laboratorio (o selecciona tu usuario del panel dinámico).
   - **Asunto** (*Subject*):
     ```text
     [REPORTE ESTATUS] Consolidado de Tareas y Recursos - Proyecto PM
     ```
   - **Cuerpo** (*Body*): Haz clic en la casilla del cuerpo del mensaje. En el menú emergente de contenido dinámico, selecciona la salida **Salida** (*Outputs*) proveniente de la acción `Estilar_Reporte_HTML`.
5. Haz clic en **Mostrar opciones avanzadas** (*Show advanced options*) y verifica que la opción **Es HTML** (*Is HTML*) esté configurada en **Sí** (*Yes*).
6. Renombra la acción a:
   ```
   Enviar_Correo_Reporte
   ```
7. Haz clic en el botón **Guardar** (*Save*) ubicado en la esquina superior derecha del diseñador.

```
[Trigger Manual]
       │
       ▼
[Obtener_Tareas_SharePoint] (SharePoint)
       │
       ▼
[Obtener_Recursos_Excel] (Excel Online Business)
       │
       ▼
[Mapear_Datos_Tareas] (Select Data Operation)
       │
       ▼
[Generar_Tabla_HTML] (Create HTML Table)
       │
       ▼
[Estilar_Reporte_HTML] (Compose with CSS)
       │
       ▼
[Enviar_Correo_Reporte] (Office 365 Outlook V2)
```

**Resultado esperado:** El flujo queda completamente guardado sin advertencias ni errores de sintaxis.

**Comprobación:** Verifica que en la barra superior aparezca la barra verde indicando "El flujo está listo para probarse".

---

## Validación y Pruebas

### Ejecución de Prueba Básica

1. En la barra superior del diseñador de Power Automate, haz clic en **Probar** (*Test*).
2. Selecciona la opción **Manualmente** (*Manually*) y haz clic en **Probar** (*Test*).
3. Si el sistema te pide autorizar las conexiones a SharePoint, Excel Online Business y Office 365 Outlook, haz clic en **Continuar** (*Continue*).
4. Haz clic en **Ejecutar flujo** (*Run flow*) y posteriormente en **Listo** (*Done*).
5. Observa la ejecución paso a paso en el panel de monitoreo. Todas las tarjetas deben mostrar un icono de verificación verde (`✓`).

---

### Pruebas de Resiliencia y Casos Adversos (Adversarial Testing)

Para validar la robustez del flujo frente a inconsistencias de datos reales de proyectos, realiza la siguiente prueba de frontera:

#### Casos de Prueba Lógica y de Datos

| Escenario | Acción de Prueba | Resultado Esperado en la Validación |
| :--- | :--- | :--- |
| **Prueba 1: Filtrado de Lista Vacía** | Cambia temporalmente la sintaxis OData en `Obtener_Tareas_SharePoint` a `Estado eq 'Inexistente'`. Ejecuta el flujo. | El flujo finaliza exitosamente (`200 OK`). El correo se envía mostrando una tabla HTML vacía sin fallar ni romper el motor de ejecución. |
| **Prueba 2: Inyección de caracteres especiales en Excel** | Edita una fila en `tbl_Recursos` e introduce el valor `ID_Tarea` = `<script>alert(1)</script>` o comillas dobles. | La acción `Listar las filas` sanitiza los caracteres y procesa la fila como cadena plano sin interrumpir el flujo. |
| **Prueba 3: Desalineación entre SharePoint y Excel** | Agrega un elemento en la lista `SeguimientoTareas` con un `ID` numérico que no exista en el Excel `tbl_Recursos`. | El flujo procesa el elemento de SharePoint en la tabla HTML manteniendo la integridad de las columnas provenientes del repositorio principal. |

---

## Solución de Problemas

### Problema 1: Error "No table was found with the name 'tbl_Recursos'" en la acción de Excel

- **Sintomas:** El flujo se detiene en la acción `Obtener_Recursos_Excel` con el código de error `404 Not Found` o un mensaje indicando que no existe la tabla especificada.
- **Causa Raíz:** Los datos en `RecursosProyecto.xlsx` fueron formateados visualmente pero no se creó la entidad de Tabla explícita (`ListObject`), o el nombre asignado en Excel Online no coincide exactamente con el valor introducido en el campo del conector.
- **Solución:**
  1. Abre `RecursosProyecto.xlsx` en el navegador.
  2. Selecciona el rango de datos `A1:E6`.
  3. Presiona `Ctrl + T`, marca "La tabla tiene encabezados" y haz clic en Aceptar.
  4. Ve a la pestaña **Diseño de tabla** y confirma que en la esquina izquierda el nombre sea exactamente `tbl_Recursos`.
  5. En Power Automate, vuelve a seleccionar la tabla `tbl_Recursos` desde la lista desplegable de la tarjeta del conector.

---

### Problema 2: El correo electrónico recibido muestra código HTML en texto plano en lugar de la tabla formateada

- **Síntomas:** El usuario recibe el correo pero ve el texto de las etiquetas HTML como `<table><tr><td>...</td></tr></table>` en lugar de la interfaz visual.
- **Causa Raíz:** El conector **Enviar un correo electrónico (V2)** de Office 365 Outlook interpretó el cuerpo del mensaje como texto plano debido a una opción de configuración deshabilitada o al uso de caracteres especiales no escapados.
- **Solución:**
  1. Abre la acción `Enviar_Correo_Reporte` en el diseñador del flujo.
  2. Haz clic en **Mostrar opciones avanzadas** (*Show advanced options*).
  3. Asegúrate de que la propiedad **Es HTML** (*Is HTML*) esté explícitamente establecida en **Sí** (*Yes*).
  4. Verifica que en el campo **Cuerpo** (*Body*) únicamente hayas mapeado la salida `Outputs` del paso `Estilar_Reporte_HTML`.

---

## Limpieza

Una vez validados los resultados de la prueba, limpia los datos temporales del entorno siguiendo estos pasos:

1. Si modificaste el filtro OData durante las pruebas de resiliencia, restaura la consulta en la acción `Obtener_Tareas_SharePoint` a su valor original:
   ```text
   Estado eq 'En progreso' or Estado eq 'Bloqueado'
   ```
2. Elimina cualquier elemento de prueba innecesario creado en la lista de SharePoint `SeguimientoTareas`.
3. Guarda el flujo `Flujo_Consolidado_Reporte_[TuNombre]` y asegúrate de que su estado permanezca en **Activado** (*On*) en el panel principal de Power Automate.

---

## Resumen

En este laboratorio has aprendido a:

1. **Estructurar fuentes no orientadas a base de datos**: Convertiste un rango plano de Excel en una Tabla oficial (`ListObject`) apta para su procesamiento desatendido.
2. **Consolidar información desde múltiples conectores estándar**: Combinaste datos en tiempo real de SharePoint Online y Excel Online Business sin recurrir a licencias Premium o servicios externos pagados.
3. **Optimizar la extracción mediante filtros OData**: Redujiste el volumen de procesamiento filtrando registros desde la fuente en la acción `Obtener elementos`.
4. **Transformar esquemas de datos en artefactos visuales**: Utilizaste las acciones de *Operación de Datos* (`Seleccionar`, `Crear tabla HTML` y `Redactar`) para inyectar estilos CSS y enviar reportes ejecutivos por correo electrónico mediante Outlook.
