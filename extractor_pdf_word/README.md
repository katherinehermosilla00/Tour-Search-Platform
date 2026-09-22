# Extractor de actividades desde PDF/Word

Este módulo permite extraer información de actividades turísticas desde documentos **PDF y Word (DOCX)** y almacenarla en el backend de **Tour Search Platform**.

El extractor procesa las carpetas asociadas a operadores turísticos, identifica información estructurada de las actividades y permite **crear o actualizar registros** en PostgreSQL mediante la API REST del backend.

## 1. Instalar dependencias

Desde la carpeta `extractor_pdf_word`, instalar las dependencias necesarias:

```powershell
pip install pdfplumber python-docx requests
```

## 2. Configurar operadores

El archivo `operadores.json` relaciona el nombre de la carpeta de cada operador turístico con su ID correspondiente en el backend.

Ejemplo:

```json
{
  "Parque Tepuhueico": 1,
  "Gigi's Tours": 2
}
```

Los nombres definidos en este archivo deben coincidir con las carpetas utilizadas para almacenar la información de cada operador.

Los IDs deben corresponder a operadores previamente registrados en Tour Search Platform.

## 3. Probar en modo simulación

Antes de realizar modificaciones en la base de datos, se recomienda ejecutar el extractor utilizando `--dry-run`.

```powershell
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --dry-run
```

Este modo permite revisar la información detectada por el extractor **sin crear ni actualizar registros en el backend**.

Es recomendable utilizar esta opción antes de realizar una importación completa.

## 4. Ejecutar la importación

Con PostgreSQL y el backend iniciados, ejecutar:

```powershell
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --backend "http://localhost:8080/api/actividades" --mapeo operadores.json
```

Por defecto, el backend de Tour Search Platform se ejecuta localmente en:

```text
http://localhost:8080
```

Durante la importación, el extractor comprueba la información existente y crea o actualiza las actividades según corresponda.

## 5. Procesar una actividad específica

Para realizar pruebas sin procesar todas las actividades disponibles, se puede utilizar el parámetro `--solo-actividad`:

```powershell
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --solo-actividad "Nombre de la actividad"
```

También puede combinarse con el modo de simulación:

```powershell
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --solo-actividad "Nombre de la actividad" --dry-run
```

Esto permite validar individualmente la extracción de una actividad antes de ejecutar una importación más amplia.

## 6. Procesar un operador específico

También es posible limitar la ejecución a un operador turístico mediante `--solo-operador`:

```powershell
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --solo-operador "Nombre del operador"
```

Esta opción permite realizar pruebas o actualizaciones de manera controlada sin procesar toda la información disponible.

## Información extraída

Dependiendo de la información disponible en cada documento, el extractor puede identificar los siguientes campos:

* Nombre de la actividad
* Descripción
* Destino
* Duración
* Edad mínima
* Edad máxima
* Idiomas
* Highlights
* Itinerario
* Horarios
* Restricciones

No todos los documentos poseen la misma estructura ni contienen todos los campos. Por este motivo, algunos valores pueden permanecer vacíos cuando la información correspondiente no se encuentra disponible en la fuente original.

## Funcionamiento

El proceso general del extractor es el siguiente:

1. Recorre las carpetas correspondientes a operadores y actividades.
2. Busca documentos compatibles en formato PDF y DOCX.
3. Extrae el contenido textual disponible.
4. Identifica las diferentes secciones presentes en el documento.
5. Estructura la información de acuerdo con el modelo `Actividad` utilizado por el backend.
6. Comprueba si la actividad correspondiente ya se encuentra registrada.
7. Crea una nueva actividad o actualiza el registro existente según corresponda.
8. Mantiene la relación entre cada actividad y su operador turístico.

El extractor utiliza principalmente **encabezados y estructura textual** para reconocer información como highlights, itinerarios, horarios y restricciones.

## Actualización de actividades

Cuando una actividad ya existe en Tour Search Platform, el extractor puede actualizar su información manteniendo la relación con el operador turístico correspondiente.

Esto permite volver a ejecutar el proceso sobre información previamente importada sin necesidad de crear manualmente una nueva actividad.

La funcionalidad facilita la corrección y actualización de los datos cuando existen nuevas versiones de los documentos de origen.

## Archivos soportados

Actualmente el extractor procesa los siguientes formatos:

```text
.pdf
.docx
```

Los documentos antiguos con extensión `.doc` no se procesan directamente.

## Fuente de información

Cuando una actividad proviene de un documento local, el sistema mantiene una referencia que permite identificar el origen de la información importada.

En el frontend, este tipo de fuente puede visualizarse como:

```text
Documento importado localmente
```

Esto permite diferenciar las actividades obtenidas desde documentos de aquellas extraídas desde sitios web.

## Campos fuera del alcance actual

En la versión actual del proyecto, los siguientes elementos **no forman parte de los requisitos de extracción y validación**:

* Imágenes
* Precio o tarifa rack

Estos elementos se consideran posibles mejoras para futuras versiones de Tour Search Platform.

## Recomendaciones de ejecución

Antes de realizar una importación completa se recomienda:

1. Verificar que PostgreSQL se encuentre iniciado.
2. Iniciar el backend de Tour Search Platform.
3. Ejecutar inicialmente el extractor utilizando `--dry-run`.
4. Revisar la información detectada.
5. Probar una actividad o un operador específico cuando sea necesario.
6. Ejecutar finalmente la importación completa.

Este procedimiento permite detectar posibles diferencias en la estructura de los documentos antes de realizar modificaciones en la base de datos.

## Estado actual

El extractor se encuentra **integrado con el backend de Tour Search Platform** y permite procesar, estructurar, crear y actualizar actividades provenientes de documentos PDF y Word.

En las pruebas realizadas durante el desarrollo, el proceso de importación permitió actualizar **161 actividades**, omitir **5 registros por falta de documentos PDF/DOCX disponibles** y finalizar la ejecución con **0 errores**.

El módulo forma parte del flujo de apoyo para la recopilación y estructuración de información turística, reduciendo el trabajo manual necesario para revisar y registrar información proveniente de documentos.
