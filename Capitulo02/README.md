# Práctica 2. Consolidar información de una lista de SharePoint y un archivo de Excel, filtrar actividades relevantes, generar un resumen de estado con contenido estructurado para reporte o presentación y distribuirlo automáticamente por correo.

## Metadatos

| Atributo | Detalle |
| :--- | :--- |
| **Duración** | **65 minutos** (fuentes 10, consultas 15, filtros y tablas 22, distribución 12, pruebas 6). |
| **Complejidad** | Intermedia. |
| **Nivel Taxonómico de Bloom** | **Analizar (nivel 4)**: seleccionar datos relevantes de dos fuentes y construir un reporte verificable. |
| **Licencia Requerida** | Microsoft 365 con Power Automate (conectores estándar), SharePoint Online, OneDrive for Business, Excel Online (Business) y Outlook. |
| **Plataformas Principales** | Power Automate (cloud vigente), SharePoint Online, Excel para la Web, OneDrive for Business y Office 365 Outlook. |

---

## Descripción General

La PMO de **PRJ-ATLAS** recibe avisos sobre actividades bloqueadas desde el laboratorio 1, pero necesita un resumen para comunicar el estado del proyecto. El seguimiento esta repartido entre la lista SharePoint `SeguimientoActividades` y una tabla Excel de hitos, por lo que consultar cada fuente manualmente dificulta priorizar riesgos.

Construirás un flujo de ejecución manual que lea las dos fuentes con conectores estándar, conserve las actividades bloqueadas y los hitos en riesgo, genere un reporte HTML con ambas secciones, guarde una copia en SharePoint y lo envíe por correo. Comprobarás que **los datos de Excel aparecen realmente en el reporte**, sin perder la trazabilidad de cada origen.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] **Recuperar datos de proyecto**, leyendo una lista de SharePoint y una tabla estructurada de Excel Online.
- [ ] **Seleccionar y estructurar información**, aplicando filtros a las dos fuentes y creando secciones HTML con registros y cantidades correctas.
- [ ] **Distribuir y conservar el resultado**, enviando el resumen por Outlook y almacenando el archivo generado en SharePoint.
- [ ] **Validar la exactitud**, contrastando los registros seleccionados con los originales y probando un cambio de datos.

---

## Prerrequisitos

### Conocimientos Previos

- Haber completado las operaciones basicas de Power Automate, disparadores y condiciones del laboratorio 1.
- Manejar tablas de Excel, carpetas de OneDrive y contenido dinámico de los flujos.

### Requisitos de Acceso y Entorno

- Haber completado el laboratorio 1 y conservar la lista `SeguimientoActividades`, sus **6 elementos** y los dos estados `Bloqueada` resultantes de la prueba (no limpiar la lista).
- Disponer del archivo [Hitos_Proyecto_Atlas.xlsx](recursos/Hitos_Proyecto_Atlas.xlsx) incluido en este paquete. Es un recurso de entrada indispensable; no modificar sus encabezados ni el nombre de tabla `tblHitos`.
- Acceder con la misma identidad de Microsoft 365 a [Power Automate](https://make.powerautomate.com/), [OneDrive for Business](https://www.microsoft365.com/launch/onedrive), Excel para la Web, SharePoint y Outlook.
- Poder crear archivos dentro de una biblioteca documental SharePoint y enviar mensajes a un buzón de prueba del tenant; tener conexiones estándar habilitadas por la política DLP.
- Disponer de navegador moderno y conexión a internet.

---

## Entorno de Laboratorio

### Especificaciones de Software y Servicios Cloud

| Herramienta / Servicio | Versión / Release | Enlace Oficial / Origen | Licencia / Permisos |
| :--- | :--- | :--- | :--- |
| Power Automate | Nube vigente; acciones integradas `Filter array`, `Create HTML table` y `Compose`. | [Operaciones de datos](https://learn.microsoft.com/en-us/power-automate/data-operations) | Derecho de crear flujos con conectores estándar. |
| SharePoint Online | Servicio Microsoft 365 vigente; `Get items` y `Create file`. | [Conector oficial](https://learn.microsoft.com/en-us/connectors/sharepointonline/) | Lectura de la lista y escritura en biblioteca de práctica. |
| Excel Online (Business) | Servicio cloud vigente; `List rows present in a table`. | [Conector oficial](https://learn.microsoft.com/en-us/connectors/excelonlinebusiness/) | Cuenta con acceso al archivo en OneDrive for Business. |
| OneDrive for Business | Servicio Microsoft 365 vigente. | [Portal de archivos](https://www.microsoft365.com/launch/onedrive) | Permiso de lectura/escritura en carpeta de práctica. |
| Office 365 Outlook | `Send an email (V2)`. | [Conector oficial](https://learn.microsoft.com/en-us/connectors/office365/) | Buzón de prueba autorizado. |

**Datos de consulta**

| Parámetro / Campo | Valor / Configuración |
| :--- | :--- |
| Proyecto | `PRJ-ATLAS` |
| Lista de origen | `SeguimientoActividades` (SharePoint, laboratorio 1). |
| Excel de entrada | [Hitos_Proyecto_Atlas.xlsx](recursos/Hitos_Proyecto_Atlas.xlsx). |
| Tabla Excel | `tblHitos`, hoja `Hitos`, seis registros. |
| Estados seleccionados | SharePoint: `Bloqueada`; Excel: `En riesgo`. |
| Flujo | `ATLAS_02_Reporte_Seguimiento`. |
| Carpeta SharePoint de resultados | `ProyectoAtlas/Reportes` dentro de la biblioteca **Documentos** del sitio de práctica. |

---

## Instrucciones Paso a Paso

### Paso 1: Preparar el archivo y la carpeta de salida (10 min)

**Objetivo:** Dejar ambas fuentes disponibles en servicios que Power Automate pueda consultar.

1. Abrir la lista SharePoint `SeguimientoActividades` creada en el laboratorio 1. Comprobar los elementos `Validar accesos del entorno QA` y `Aprobar plan de liberación` con `Estado = Bloqueada`.
2. Descargar [Hitos_Proyecto_Atlas.xlsx](recursos/Hitos_Proyecto_Atlas.xlsx). Abrir OneDrive for Business, crear la carpeta `ProyectoAtlas` y cargar el archivo sin cambiarle el nombre.
3. Abrir el archivo cargado con **Excel para la Web**. Seleccionar una celda y verificar que el rango está definido como **Tabla** con nombre `tblHitos` (revisar la pestaña **Tabla / Table Design**, según la interfaz).
4. Confirmar que `H-102` y `H-104` tienen `EstadoHito = En riesgo`; identificar los encabezados exactos `IdHito`, `Hito`, `Responsable`, `EstadoHito`, `FechaObjetivo` e `Impacto`.
5. Crear en la biblioteca **Documentos** del sitio SharePoint `ProyectoAtlas` y dentro de ella `Reportes` si aún no existen.

> **Nota técnica:** Excel Online (Business) requiere una **tabla estructurada**. No basta con tener celdas con encabezados. El flujo leera el archivo de OneDrive for Business y escribira el reporte en SharePoint: son ubicaciones distintas.

**Resultado esperado:** Lista SharePoint, tabla `tblHitos` en OneDrive y carpeta SharePoint de reportes disponibles.

**Comprobación:** El archivo Excel muestra seis filas y la carpeta de reportes está vacia o contiene solo resultados de pruebas previas.

---

### Paso 2: Crear el flujo y consultar las dos fuentes (15 min)

**Objetivo:** Obtener arreglos de registros desde SharePoint y Excel sin utilizar integraciones Premium.

1. Abrir Power Automate y seleccionar **Crear > Flujo de nube instantaneo**. Asignar el nombre `ATLAS_02_Reporte_Seguimiento` y escoger el disparador **Manually trigger a flow / Desencadenar manualmente un flujo**.
2. Agregar **SharePoint > Get items / Obtener elementos**; configurar `Site Address` con el sitio de práctica y `List Name = SeguimientoActividades`. Renombrar la tarjeta como `SP_Obtener_actividades` antes de crear expresiones. En "Parámetros avanzados", seleccionar**Primeros puestos** e insertar `Top` con el valor 20 para limitar la salida a los registros de prueba.
3. Agregar **Excel Online (Business) > List rows present in a table / Enumerar las filas de una tabla** y renombrarla `Excel_Listar_hitos`.
4. Configurar los parámetros de Excel según su ubicación en OneDrive:

   | Campo | Configuración |
   | :--- | :--- |
   | **Location** | `OneDrive for Business`. |
   | **Document Library** | Biblioteca de OneDrive de la cuenta (normalmente `OneDrive`). |
   | **File** | Seleccionar con el explorador `ProyectoAtlas/Hitos_Proyecto_Atlas.xlsx`. |
   | **Table** | `tblHitos`. |

5. Guardar el flujo y ejecutar **Probar > Manualmente** una vez para validar las conexiones. Abrir las salidas de las dos acciones y localizar el arreglo `value` de **6 elementos** en cada una.

> **Nota técnica:** Realizar la prueba en un sitio exclusivo del curso con los seis registros definidos; `Get items` sin filtro devuelve los elementos de la lista elegida. Si la lista contiene información ajena al caso, utilizar otra lista de práctica, no mezclar registros de un proyecto real con los resultados esperados.

**Resultado esperado:** Dos acciones de lectura completadas con seis registros cada una.

**Comprobación:** El historial muestra `SP_Obtener_actividades` y `Excel_Listar_hitos` en estado **Succeeded**, cada una con un arreglo `value`.

---

### Paso 3: Filtrar los registros relevantes y construir dos tablas (22 min)

**Objetivo:** Obtener exclusivamente actividades bloqueadas e hitos en riesgo, y convertir ambas listas en información tabular.

1. Agregar **Data Operations > Filter array / Filtrar matriz**, cambiar el nombre de la tarjeta a `Filtrar_actividades` y configurar **From** desde la pestaña **Expression**:

   ```text
   body('SP_Obtener_actividades')?['value']
   ```

2. Configurar la condición del filtro con el valor **Estado Value** de cada elemento, comparado con el texto `Bloqueada`. Si el editor solicita una expresión en el operando izquierdo, usar:

   ```text
   item()?['Estado']?['Value']
   ```

   El operador debe ser **is equal to** y el valor derecho `Bloqueada`. Si la tarjeta ofrece un modo avanzado que espera la expresión completa, usar `@equals(item()?['Estado']?['Value'], 'Bloqueada')`. En un campo de expresión normal, omitir el `@` externo.

3. Agregar otro **Filter array** llamado `Filtrar_hitos`; configurar **From** con:

   ```text
   body('Excel_Listar_hitos')?['value']
   ```

   Filtrar comparando `item()?['EstadoHito']` con `En riesgo` mediante el operador **is equal to**.

4. Agregar **Data Operations > Create HTML table** y renombrar `Tabla_actividades`. Configurar **From** con `body('Filtrar_actividades')`, seleccionar **Columns = Custom** y agregar estas correspondencias. Escribir los valores como **expresiones** y no como texto literal:

   | Cabecera | Expresión por fila |
   | :--- | :--- |
   | Actividad | `item()?['Title']` |
   | Proyecto | `item()?['Proyecto']` |
   | Estado | `item()?['Estado']?['Value']` |
   | Comentarios | `item()?['Comentarios']` |

5. Agregar una segunda acción **Create HTML table**, renombrarla `Tabla_hitos`, configurar **From** con `body('Filtrar_hitos')`, seleccionar columnas personalizadas e indicar:

   | Cabecera | Expresión por fila |
   | :--- | :--- |
   | ID | `item()?['IdHito']` |
   | Hito | `item()?['Hito']` |
   | Responsable | `item()?['Responsable']` |
   | Estado | `item()?['EstadoHito']` |
   | Impacto | `item()?['Impacto']` |

6. Agregar **Compose / Componer / Redactar** con nombre `Resumen_HTML`. En la pestaña **Expression** pegar, sin `@` inicial, la expresión completa:

   ```text
   concat(
     '<h2>PRJ-ATLAS | Resumen de seguimiento</h2>',
     '<p><strong>Actividades bloqueadas: ',
     string(length(body('Filtrar_actividades'))),
     '</strong></p>',
     body('Tabla_actividades'),
     '<p><strong>Hitos en riesgo: ',
     string(length(body('Filtrar_hitos'))),
     '</strong></p>',
     body('Tabla_hitos'),
     '<p>Origen: SharePoint y Excel Online (Business). Revisión documental por PMO.</p>'
   )
   ```

7. Guardar el flujo. Revisar que los nombres de las acciones coincidan **exactamente** con los utilizados en las expresiones y que no quede ninguna columna de HTML sin valor. En esta composición, `body('Tabla_actividades')` y `body('Tabla_hitos')` recuperan el contenido HTML de cada acción. No utilizar `outputs('Tabla_actividades')` ni `outputs('Tabla_hitos')` directamente en `concat()`, porque esas referencias pueden contener el objeto de salidas completo. En cambio, `outputs('Resumen_HTML')` recupera la cadena producida por una acción `Compose`.

> **Nota técnica:** En una columna de tipo **Elección** de SharePoint, el valor puede exponerse como objeto con propiedad `Value`. Si tu historial muestra una **cadena simple** en `Estado`, adaptar tanto el filtro como la columna HTML a `item()?['Estado']`, tras comprobar la salida real. No dejar dos interpretaciones simultaneas. Las etiquetas visibles del diseñador pueden variar, pero las propiedades del registro deben validarse con los datos de ejecución.

**Resultado esperado:** Dos matrices filtradas, dos tablas HTML y una composición que incorpora efectivamente los datos de SharePoint y Excel, sin afirmar que se haya realizado un cruce por ID entre fuentes diferentes.

**Comprobación:** Ejecutar el flujo hasta `Resumen_HTML` e inspeccionar que `Filtrar_actividades` contiene **2 elementos** y `Filtrar_hitos` otros **2**, con IDs de hito `H-102` y `H-104`.

---

### Paso 4: Conservar el reporte y enviarlo por correo (12 min)

**Objetivo:** Distribuir una salida estructurada y conservar una copia consultable en SharePoint.

1. Agregar después de `Resumen_HTML` **SharePoint > Create file / Crear archivo**. Seleccionar el mismo sitio. En **Folder Path / Ruta de carpeta**, usar el explorador de carpetas y elegir la carpeta `ProyectoAtlas/Reportes` **dentro de la biblioteca Documentos**. El valor técnico de `Folder Path` debe comenzar por la biblioteca existente (por ejemplo, `/Shared Documents/ProyectoAtlas/Reportes`), no por `ProyectoAtlas/Reportes` sin el prefijo de biblioteca; utilizar la ruta que devuelva el selector de tu tenant.
2. Configurar **File Name / Nombre de archivo** con esta expresión para evitar sobrescribir ejecuciones anteriores:

   ```text
   concat(
   '<div style="margin:0;padding:24px;background-color:#F3F6FA;font-family:Segoe UI,Arial,sans-serif;color:#243247;">',
   '<table width="100%" cellpadding="0" cellspacing="0" style="max-width:760px;margin:0 auto;background-color:#FFFFFF;border:1px solid #E2E8F0;">',
   '<tr><td style="padding:28px;">',

   '<p style="margin:0 0 8px;font-size:11px;font-weight:700;letter-spacing:1.5px;color:#64748B;">REPORTE DE SEGUIMIENTO | PMO</p>',
   '<h1 style="margin:0;font-size:26px;line-height:1.3;color:#17324D;">Proyecto PRJ-ATLAS</h1>',
   '<p style="margin:10px 0 24px;font-size:14px;color:#64748B;">Resumen consolidado de actividades y hitos del proyecto.</p>',

   '<table width="100%" cellpadding="0" cellspacing="0" style="border-collapse:separate;border-spacing:8px;">',
   '<tr>',
   '<td width="50%" style="padding:16px;background-color:#FFF5F2;border-left:4px solid #D65B45;">',
   '<p style="margin:0 0 8px;font-size:11px;font-weight:700;color:#9F3B2D;">ACTIVIDADES BLOQUEADAS</p>',
   '<p style="margin:0;font-size:28px;font-weight:700;color:#9F3B2D;">',
   string(length(body('Filtrar_actividades'))),
   '</p></td>',

   '<td width="50%" style="padding:16px;background-color:#FFF8E7;border-left:4px solid #D59A28;">',
   '<p style="margin:0 0 8px;font-size:11px;font-weight:700;color:#986A14;">HITOS EN RIESGO</p>',
   '<p style="margin:0;font-size:28px;font-weight:700;color:#986A14;">',
   string(length(body('Filtrar_hitos'))),
   '</p></td>',
   '</tr></table>',

   '<h2 style="margin:28px 0 12px;font-size:17px;color:#17324D;">01. Actividades bloqueadas</h2>',
   '<p style="margin:0 0 12px;font-size:12px;color:#64748B;">Actividades que requieren seguimiento para resolver impedimentos.</p>',

   replace(
      replace(
         replace(
         body('Tabla_actividades'),
         '<table>',
         '<table width="100%" cellpadding="0" cellspacing="0" style="width:100%;border-collapse:collapse;border:1px solid #CBD5E1;font-family:Segoe UI,Arial,sans-serif;">'
         ),
         '<th>',
         '<th style="padding:12px 10px;background-color:#17324D;color:#FFFFFF;text-align:left;font-size:12px;font-weight:600;border:1px solid #36526B;">'
      ),
      '<td>',
      '<td style="padding:11px 10px;background-color:#FFFFFF;color:#334155;font-size:12px;line-height:1.5;border:1px solid #E2E8F0;vertical-align:top;word-break:break-word;">'
   ),

   '<h2 style="margin:30px 0 12px;font-size:17px;color:#17324D;">02. Hitos en riesgo</h2>',
   '<p style="margin:0 0 12px;font-size:12px;color:#64748B;">Hitos que requieren atención por su estado actual.</p>',

   replace(
      replace(
         replace(
         body('Tabla_hitos'),
         '<table>',
         '<table width="100%" cellpadding="0" cellspacing="0" style="width:100%;border-collapse:collapse;border:1px solid #CBD5E1;font-family:Segoe UI,Arial,sans-serif;">'
         ),
         '<th>',
         '<th style="padding:12px 10px;background-color:#17324D;color:#FFFFFF;text-align:left;font-size:12px;font-weight:600;border:1px solid #36526B;">'
      ),
      '<td>',
      '<td style="padding:11px 10px;background-color:#FFFFFF;color:#334155;font-size:12px;line-height:1.5;border:1px solid #E2E8F0;vertical-align:top;word-break:break-word;">'
   ),

   '<table width="100%" cellpadding="0" cellspacing="0" style="margin-top:28px;border-top:1px solid #E2E8F0;">',
   '<tr><td style="padding-top:16px;">',
   '<p style="margin:0 0 6px;font-size:12px;color:#64748B;"><strong>Fuentes:</strong> SharePoint y Excel Online (Business).</p>',
   '<p style="margin:0;font-size:12px;color:#64748B;"><strong>Responsable de revisión:</strong> PMO.</p>',
   '</td></tr></table>',

   '</td></tr></table></div>'
   )

   ```

3. Configurar **File Content / Contenido del archivo** con la salida dinamica de `Resumen_HTML` (`outputs('Resumen_HTML')`).
4. Agregar **Office 365 Outlook > Send an email (V2)** y completar:

   | Campo | Valor / Configuración |
   | :--- | :--- |
   | Para | Buzón de prueba autorizado. |
   | Asunto | `PRJ-ATLAS, Reporte de actividades e hitos`. |
   | Cuerpo | Salida dinamica de `Resumen_HTML` (contenido HTML). |

5. Guardar y ejecutar el flujo de forma manual. En **Send an email (V2)**, insertar la salida dinámica de `Resumen_HTML` en el campo **Body**; si el editor enriquecido muestra etiquetas como texto, cambiar a **vista de código HTML** (`</>` en el editor que muestre esa opción) y volver a insertar la salida dinámica. Abrir el correo recibido y comprobar que muestra un encabezado, **dos tablas** y contadores de registros.
6. Abrir `ProyectoAtlas/Reportes` en SharePoint, localizar el archivo `Reporte_ATLAS_*.html` y abrirlo o descargarlo para comprobar el contenido. Si SharePoint bloquea la vista previa de HTML, la descarga es suficiente.

> **Nota técnica:** `Send an email (V2)` recibe un cuerpo HTML; no es necesario buscar una opción adicional `Is HTML`. El resultado ofrece texto y tablas reutilizables en informes o presentaciones.

**Resultado esperado:** Correo con resumen consolidado y archivo HTML almacenado en SharePoint.

**Comprobación:** El correo y el archivo incluyen las **2 actividades bloqueadas** y los hitos **H-102 / H-104**, además de sus cantidades.

---

### Paso 5: Probar un cambio real en los datos (6 min)

**Objetivo:** Comprobar que el reporte responde al origen y no contiene valores escritos manualmente.

1. Abrir el libro Excel en OneDrive, localizar `H-104` y cambiar temporalmente `EstadoHito` de `En riesgo` a `En curso`; guardar y cerrar el editor de la hoja.
2. Ejecutar de nuevo el flujo `ATLAS_02_Reporte_Seguimiento` y abrir la salida `Filtrar_hitos`.
3. Verificar que solo figura `H-102` y que el correo nuevo indica **Hitos en riesgo: 1**. Si la modificación no aparece inmediatamente, esperar a que Excel para la Web termine de guardar, cerrar el libro y ejecutar de nuevo: la lectura del conector puede mostrar temporalmente información anterior.
4. Restaurar `H-104` a `En riesgo` y guardar para devolver la fuente al estado inicial.

**Resultado esperado:** La cantidad de hitos del informe cambia de 2 a 1 al cambiar los datos de entrada.

**Comprobación:** Inspeccionar la tabla HTML del segundo correo: no debe incluir `H-104` mientras este en `En curso`.

---

## Validación y Pruebas

### Ejecución de Prueba Básica

1. Mostrar el historial de una ejecución exitosa con **dos** consultas, **dos** filtros, **dos** tablas, un archivo y un envío de correo.
2. Verificar que los filtros contienen `Validar accesos del entorno QA`, `Aprobar plan de liberación`, `H-102` y `H-104` antes de la prueba de cambio de datos.
3. Comparar el correo recibido con SharePoint y el libro Excel; comprobar origen y cantidades.
4. Confirmar que existe una copia HTML nueva en `ProyectoAtlas/Reportes`.

### Pruebas de Resiliencia y Casos Adversos (Adversarial Testing)

Estas pruebas se limitan a verificar la selección de datos y la integridad del resultado, sin agregar integraciones.

#### Casos de Prueba Lógica y de Datos

| Escenario | Acción de Prueba | Resultado Esperado en la Validación |
| :--- | :--- | :--- |
| **Cambio de estado Excel** | Cambiar temporalmente `H-104` a `En curso` y volver a ejecutar. | La tabla de hitos excluye `H-104`, y el conteo pasa de 2 a 1. |
| **Registros no relevantes** | Revisar que `H-101` (`En curso`) y `Publicar minuta de seguimiento` (`Completada`) no aparezcan. | Ambos registros se excluyen de los filtros respectivos. |
| **Tabla no localizada** | Inspeccionar el selector cuando el libro se reubique o se haya cambiado de nombre. | Se identifica un error de origen; corregir la ruta o la tabla, no inventar registros faltantes. |

---

## Solución de Problemas

### Problema 1: `tblHitos` no aparece o Excel no devuelve filas

- **Síntomas:** El selector no ofrece `tblHitos`, o `List rows present in a table` falla.
- **Causa Raíz:** Archivo abierto desde OneDrive personal, tabla convertida en rango o ruta equivocada.
- **Solución:**
  1. Abrir el libro en **OneDrive for Business** y comprobar el nombre `tblHitos` desde Excel para la Web.
  2. Volver al selector de **File** y elegir el archivo desde el explorador, no escribir una URL de comparticion.
  3. Guardar y ejecutar de nuevo la consulta, inspeccionando el arreglo `value`.

### Problema 2: La tabla de actividades aparece vacia

- **Síntomas:** La consulta SharePoint tiene elementos, pero `Filtrar_actividades` devuelve cero registros.
- **Causa Raíz:** La ruta `Estado/Value` no coincide con la estructura de salida o los estados no son `Bloqueada`.
- **Solución:**
  1. Inspeccionar el objeto de un elemento de `SP_Obtener_actividades` en el historial.
  2. Adaptar la expresión a `item()?['Estado']?['Value']` o a `item()?['Estado']`, según el dato realmente devuelto.
  3. Confirmar dos elementos bloqueados y ejecutar otra vez.

### Problema 3: El correo muestra código HTML o solo una fuente

- **Síntomas:** Aparecen etiquetas HTML literales, falta Excel o los contadores no coinciden.
- **Causa Raíz:** El cuerpo esta escrito en modo texto literal o la composición omite `Tabla_hitos`.
- **Solución:**
  1. Inspeccionar la salida de `Resumen_HTML` y comprobar que contiene las cadenas HTML de ambas tablas. Dentro de `concat()`, utilizar **`body('Tabla_actividades')`** y **`body('Tabla_hitos')`** para insertar el marcado de las acciones `Create HTML table` y no el objeto completo de salidas de esas acciones.
  2. En el campo **Body** de Outlook, colocar la **salida dinamica** de `Resumen_HTML` en el editor de contenido HTML, no copiar el código de la expresión como texto.
  3. Volver a ejecutar y comprobar las dos secciones en el correo y en SharePoint.

---

## Limpieza

Una vez validados los resultados:

1. Restaurar `EstadoHito` del hito `H-104` a `En riesgo`, para conservar los datos originales.
2. Conservar la lista y el archivo Excel para futuras comprobaciones; no eliminar archivos o carpetas del sitio compartido sin autorización.
3. Eliminar solo las copias HTML o los correos **de prueba** que no sea necesario conservar, si la política del entorno lo permite.

---

## Resumen

En este laboratorio has aprendido a:

1. **Obtener datos heterogeneos:** Recuperar actividades e hitos con dos conectores estándar.
2. **Aplicar filtros verificables:** Excluir registros no relevantes y controlar cantidades reales.
3. **Producir un reporte reutilizable:** Integrar dos tablas HTML, guardarlas en SharePoint y distribuirlas por Outlook.
4. **Comprobar cambios de origen:** Confirmar que la salida depende de Excel y no de contenido fijo.


