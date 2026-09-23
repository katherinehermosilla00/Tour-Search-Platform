# Tour Search Platform

Plataforma para automatizar la extracción, validación, almacenamiento y consulta de información de actividades y operadores turísticos.

El sistema obtiene información desde sitios web y documentos PDF/DOCX, la estructura y almacena en PostgreSQL mediante una API desarrollada con Spring Boot, y posteriormente permite consultarla a través de una interfaz web construida con React.

> **Estado:** en desarrollo y validación. Los componentes de backend, frontend, base de datos, scraper web y extractor documental están integrados y funcionales como MVP.

---

## Contexto y propósito

Tour Search Platform fue desarrollado por **Katherine Hermosilla Ortega, estudiante de Ingeniería Informática**, durante su práctica profesional y a partir de su experiencia trabajando junto al área de **Customer Success** de una organización del sector turístico.

El proyecto surge de una necesidad observada directamente durante el trabajo cotidiano del área. La revisión, optimización y corrección de fichas turísticas requiere localizar información distribuida entre sitios web de operadores y documentos PDF/DOCX. Frente a un aumento de solicitudes, repetir estas búsquedas manualmente implica destinar una parte importante del tiempo a localizar, revisar y organizar antecedentes antes de poder trabajar sobre cada ficha.

Como respuesta a esta necesidad se desarrolló Tour Search Platform como una **herramienta informática de apoyo**, capaz de extraer, estructurar, almacenar y posteriormente consultar información turística de forma centralizada.

El objetivo de la plataforma es **facilitar el trabajo de las personas del área de Customer Success, disminuir tareas manuales repetitivas y contribuir al ahorro de tiempo**, evitando que cada nueva solicitud obligue a comenzar desde cero la búsqueda de información que puede haber sido localizada o procesada previamente.

La plataforma **no reemplaza a las personas del área, no automatiza la decisión final sobre las fichas y no sustituye los sistemas, herramientas ni procesos oficiales de la organización**. La información obtenida mediante los procesos automáticos continúa sujeta a revisión y validación humana.

El proyecto aplica conocimientos propios de **Ingeniería Informática** mediante el desarrollo e integración de frontend, backend, base de datos, APIs REST, extracción automatizada de información, seguridad, persistencia y pruebas de software.

---

## Contenido

* [Contexto y propósito](#contexto-y-propósito)
* [Tecnologías utilizadas](#tecnologías-utilizadas)
* [Arquitectura general](#arquitectura-general)
* [Puesta en marcha](#puesta-en-marcha)

  * [1. Base de datos PostgreSQL](#1-base-de-datos-postgresql)
  * [2. Backend Java](#2-backend-java)
  * [3. Scraper web FastAPI](#3-scraper-web-fastapi)
  * [4. Extracción desde PDF y Word](#4-extracción-desde-pdf-y-word)
  * [5. Frontend](#5-frontend)
* [Modelo de información](#modelo-de-información)
* [API — Endpoints principales](#api--endpoints-principales)
* [Funcionalidades de la plataforma](#funcionalidades-de-la-plataforma)
* [Seguridad](#seguridad)
* [Normas y buenas prácticas de referencia](#normas-y-buenas-prácticas-de-referencia)
* [Flujo de extracción y consulta](#flujo-de-extracción-y-consulta)
* [Pruebas y validación](#pruebas-y-validación)
* [Alcance actual y mejoras futuras](#alcance-actual-y-mejoras-futuras)
* [Estado del proyecto](#estado-del-proyecto)
* [Autoría](#autoría)

---

## Tecnologías utilizadas

| Capa                      | Tecnologías                                                                                  |
| ------------------------- | -------------------------------------------------------------------------------------------- |
| **Frontend**              | React, Vite, Mantine, JavaScript                                                             |
| **Backend**               | Java 21, Spring Boot, Spring Data JPA / Hibernate, Spring Security, Maven, Swagger / OpenAPI |
| **Base de datos**         | PostgreSQL 17, Docker                                                                        |
| **Extracción web**        | Python, FastAPI, BeautifulSoup, Requests, Playwright                                         |
| **Extracción documental** | Python, pdfplumber, python-docx, Requests                                                    |

---

## Arquitectura general

La solución utiliza una arquitectura modular en la que cada componente posee una responsabilidad definida.

```text
tour-search-platform/
│
├── backend/                 API REST desarrollada con Spring Boot
├── frontend/                Aplicación web React + Vite
├── scraper_v2/              Servicio FastAPI para extracción desde sitios web
├── extractor_pdf_word/      Extracción e importación desde documentos PDF/DOCX
│
├── docker-compose.yml
├── .gitignore
└── README.md
```

Flujo principal de información:

```text
Sitio web / PDF / DOCX
          │
          ▼
  Scraper / Extractor
          │
          ▼
      Validación
          │
          ▼
 Backend (Spring Boot)
          │
          ▼
      PostgreSQL
          │
          ▼
   Frontend (React)
```

El scraper y el extractor documental recopilan y estructuran la información disponible. Posteriormente, el backend gestiona su persistencia en PostgreSQL y el frontend permite consultar y administrar los datos almacenados.

---

# Puesta en marcha

## 1. Base de datos PostgreSQL

Desde la raíz del proyecto:

```bash
docker compose up -d postgres
docker compose ps
```

Puerto por defecto:

```text
5432
```

---

## 2. Backend Java

**Requisitos:**

* Java 21
* Maven
* PostgreSQL iniciado
* Variables de entorno requeridas configuradas

Ejecutar:

```bash
cd backend
mvn spring-boot:run
```

Servicios principales:

| Servicio   | Dirección                               |
| ---------- | --------------------------------------- |
| API        | `http://localhost:8080`                 |
| Swagger UI | `http://localhost:8080/swagger-ui.html` |

---

## 3. Scraper web FastAPI

Desde la carpeta `scraper_v2`:

```bash
cd scraper_v2

python -m venv .venv
.venv\Scripts\activate

pip install -r requirements.txt

python -m uvicorn main:app --reload --port 8000
```

Servicios:

| Servicio   | Dirección                    |
| ---------- | ---------------------------- |
| API        | `http://localhost:8000`      |
| Swagger UI | `http://localhost:8000/docs` |

El scraper analiza páginas de actividades turísticas y extrae información estructurada mediante reglas genéricas y análisis contextual.

Entre los campos considerados se encuentran:

* nombre;
* descripción;
* ubicación;
* destino;
* duración;
* edad mínima y máxima cuando corresponde;
* idiomas;
* highlights;
* itinerario;
* horarios;
* restricciones.

La lógica de extracción busca evitar una dependencia directa de valores específicos de un único operador.

También se incorpora detección de fuentes protegidas por CAPTCHA o mecanismos anti-bot. Cuando se identifica un bloqueo, el proceso se detiene y reporta la situación en lugar de intentar superar la protección.

---

## 4. Extracción desde PDF y Word

El módulo:

```text
extractor_pdf_word/
```

procesa documentos `.pdf` y `.docx` proporcionados como fuentes de información.

El extractor identifica y estructura, cuando se encuentran disponibles, datos como:

* nombre de la actividad;
* descripción;
* ubicación;
* destino;
* duración;
* edades;
* idiomas;
* highlights;
* itinerario;
* horarios;
* restricciones.

Permite realizar pruebas sin persistencia mediante `dry-run` y limitar la ejecución a una actividad u operador específico.

Ejemplos:

```bash
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --dry-run
```

```bash
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --solo-actividad "Nombre"
```

```bash
python extraer_actividades.py --raiz "C:\ruta\a\la\carpeta" --solo-operador "Operador"
```

Los datos procesados pueden ser enviados al backend para su creación o actualización en PostgreSQL.

---

## 5. Frontend

Ejecutar:

```bash
cd frontend
npm install
npm run dev
```

Normalmente se encuentra disponible en:

```text
http://localhost:5173
```

Si ese puerto está ocupado, Vite puede utilizar:

```text
http://localhost:5174
```

La interfaz permite realizar consultas, visualizar información turística, autenticarse y acceder a funcionalidades administrativas según el rol correspondiente.

---

## Modelo de información

El modelo principal utilizado por el flujo actual es **Actividad**.

Una actividad puede almacenar:

* nombre;
* descripción;
* ubicación;
* destino;
* duración;
* edad mínima y máxima;
* idiomas;
* highlights;
* itinerario;
* horarios;
* restricciones;
* URL o fuente de origen;
* operador turístico asociado;
* estado activo/inactivo.

Las actividades importadas desde documentos locales pueden identificarse como:

```text
Documento importado localmente
```

evitando tratar una ruta local como si fuera una dirección web pública.

Relación conceptual principal:

```text
País
  │
  ▼
Operador turístico
  │
  ▼
Actividad
  │
  ├── Idiomas
  ├── Highlights
  ├── Itinerario
  ├── Horarios
  └── Restricciones
```

Los usuarios corresponden al acceso y administración de la plataforma y no constituyen una entidad dependiente de las actividades.

---

# API — Endpoints principales

## Actividades

| Método | Endpoint                                          | Descripción              |
| ------ | ------------------------------------------------- | ------------------------ |
| GET    | `/api/actividades`                                | Listado de actividades   |
| GET    | `/api/actividades/{id}`                           | Consultar una actividad  |
| GET    | `/api/actividades/operador/{operadorTuristicoId}` | Actividades por operador |
| GET    | `/api/actividades/buscar?texto={texto}`           | Búsqueda de actividades  |
| POST   | `/api/actividades`                                | Crear una actividad      |
| PUT    | `/api/actividades/{id}`                           | Actualizar una actividad |
| PATCH  | `/api/actividades/{id}/desactivar`                | Desactivar una actividad |
| DELETE | `/api/actividades/{id}`                           | Eliminar una actividad   |

## Búsqueda de tours

El repositorio conserva un módulo complementario/histórico para búsquedas mediante:

```text
GET /api/tours/search
```

Este módulo no representa el flujo principal de extracción de actividades del MVP actual.

## Otros módulos

La API incluye además rutas relacionadas con:

```text
/api/operadores
/api/paises
/api/auth
/api/admin
/api/registros
```

Estas permiten gestionar operadores turísticos, países, autenticación, administración y registros asociados al funcionamiento de la plataforma.

---

## Funcionalidades de la plataforma

* Inicio de sesión.
* Registro y gestión de usuarios.
* Recuperación de contraseña.
* Segundo factor de autenticación.
* Consulta y búsqueda de actividades.
* Visualización del detalle de actividades.
* Consulta de operadores turísticos y países.
* Administración de actividades.
* Administración de operadores y países.
* Administración de usuarios y roles.
* Consulta de registros.
* Edición y desactivación de información según las capacidades disponibles.
* Extracción desde fuentes web.
* Extracción e importación desde documentos PDF/DOCX.
* Persistencia estructurada en PostgreSQL.
* Documentación de la API mediante Swagger/OpenAPI.

---

## Seguridad

La plataforma incorpora controles de seguridad orientados al funcionamiento del MVP.

* Autenticación mediante **JWT** con política `stateless`.
* Contraseñas protegidas mediante **BCrypt**.
* **Segundo factor de autenticación (2FA/TOTP)**.
* Recuperación y restablecimiento de contraseña.
* Control de acceso mediante **roles**.
* Rutas `/api/admin/**` restringidas al rol `ADMIN`.
* Validación de URLs antes de su procesamiento.
* Variables de entorno para secretos y credenciales.
* Detección de fuentes web bloqueadas.
* No se intenta superar CAPTCHA, autenticación, paywalls u otros mecanismos de protección.

Ejemplo de configuración local:

```powershell
$env:JWT_SECRET="TU_CLAVE_CONFIGURADA_LOCALMENTE"
```

> **Importante:** nunca se deben subir contraseñas, claves JWT, tokens, secretos 2FA ni otras credenciales al repositorio.

### Pendientes de seguridad para producción

Antes de un eventual despliegue productivo se deben considerar, entre otras medidas:

* restringir las operaciones de escritura de las APIs según autenticación y rol;
* parametrizar CORS según el ambiente;
* utilizar HTTPS;
* fortalecer la gestión centralizada de secretos;
* incorporar auditoría y monitoreo.

---

# Normas y buenas prácticas de referencia

Durante el desarrollo de Tour Search Platform se consideraron normas internacionales como referencia para orientar aspectos relacionados con seguridad de la información, controles técnicos, calidad y mejora continua.

Su utilización corresponde a la aplicación de principios y buenas prácticas y **no implica que Tour Search Platform o la organización estén certificados bajo estas normas ni constituye una declaración de cumplimiento integral**.

## ISO/IEC 27001 — Gestión de la seguridad de la información

ISO/IEC 27001 se consideró como referencia para orientar la protección de la información y la gestión de riesgos asociados al acceso y utilización de la plataforma.

Dentro del proyecto, este enfoque se relaciona con medidas como:

* autenticación de usuarios;
* control de acceso mediante roles;
* protección de contraseñas con BCrypt;
* utilización de JWT;
* segundo factor de autenticación;
* recuperación de contraseña;
* administración de secretos mediante variables de entorno;
* identificación de riesgos y controles pendientes.

## ISO/IEC 27002 — Controles de seguridad de la información

ISO/IEC 27002 se consideró como referencia complementaria para buenas prácticas relacionadas con controles de seguridad.

En Tour Search Platform estas prácticas se relacionan con:

* control de acceso;
* autenticación;
* protección de credenciales;
* validación de entradas y URLs;
* separación de responsabilidades entre componentes;
* configuración segura;
* protección de secretos;
* detección y respeto de mecanismos de protección presentes en fuentes externas.

## ISO 9001 — Gestión de la calidad

ISO 9001 se consideró como referencia para orientar aspectos relacionados con calidad, control de procesos y mejora continua durante el desarrollo del proyecto.

Este enfoque se relaciona con:

* definición de requerimientos;
* documentación del funcionamiento;
* establecimiento de procesos de extracción y validación;
* ejecución de pruebas;
* registro de resultados;
* identificación de errores y limitaciones;
* corrección de problemas detectados;
* definición de mejoras futuras.

La aplicación de estas buenas prácticas permite mantener evidencia del comportamiento observado y continuar mejorando progresivamente la solución.

> ISO/IEC 27001, ISO/IEC 27002 e ISO 9001 se presentan exclusivamente como **marcos de referencia utilizados durante el desarrollo**. No se declara certificación formal ni conformidad integral con estas normas.

---

## Flujo de extracción y consulta

El scraping no necesita ejecutarse cada vez que una persona realiza una consulta.

El flujo general es:

```text
Fuente externa
      │
      ▼
Scraper / Extractor
      │
      ▼
Validación y estructuración
      │
      ▼
Backend
      │
      ▼
PostgreSQL
      │
      ▼
Consulta desde Frontend
      │
      ▼
Revisión humana
```

Este enfoque permite reutilizar información previamente procesada y disminuye la necesidad de comenzar nuevamente una búsqueda manual desde cero.

La información obtenida automáticamente debe considerarse un **insumo para el trabajo del área**. La revisión final continúa siendo responsabilidad de las personas que utilizan la plataforma.

---

## Pruebas y validación

### Extractor documental

Durante la ejecución completa utilizada para validar el extractor se obtuvieron:

| Resultado                | Cantidad |
| ------------------------ | -------: |
| Actividades actualizadas |      161 |
| Actividades omitidas     |        5 |
| Errores                  |        0 |

Los casos que habían presentado problemas durante etapas anteriores fueron reprocesados antes de la ejecución final.

### Scraper web

Se validó utilizando una ficha real de actividad para comprobar la extracción contextual de información.

También se probó el comportamiento frente a una fuente protegida mediante mecanismos anti-bot. El sistema detectó el bloqueo y detuvo la extracción sin intentar evadir la protección.

### Persistencia e interfaz

Se verificaron operaciones relacionadas con:

* persistencia de actividades;
* actualización de información;
* visualización de campos extraídos;
* administración de operadores y países;
* desactivación de actividades;
* consulta desde el frontend.

### Rendimiento

Se realizaron pruebas mediante **Apache JMeter 5.6.3** sobre:

```text
GET /api/actividades
```

Resultados del escenario local probado:

| Usuarios concurrentes | Solicitudes | Promedio |  Error |
| --------------------: | ----------: | -------: | -----: |
|                    10 |         100 |   969 ms | 0,00 % |
|                    30 |         300 | 2.363 ms | 0,00 % |
|                    45 |         450 | 3.784 ms | 0,00 % |
|                    60 |         600 | 8.617 ms | 8,33 % |

En el escenario probado, **45 usuarios concurrentes realizaron 450 solicitudes sin errores**.

Con 60 usuarios concurrentes se observó degradación del rendimiento y un 8,33 % de errores.

Estos resultados corresponden exclusivamente al endpoint, configuración y entorno local utilizados durante la prueba y **no representan un límite absoluto de capacidad de toda la plataforma**.

---

## Alcance actual y mejoras futuras

### Alcance actual

La versión actual se concentra en:

* extracción automática de información turística desde sitios web;
* extracción desde PDF y DOCX;
* almacenamiento estructurado;
* gestión de países y operadores;
* gestión de actividades;
* consulta y búsqueda de información;
* administración de usuarios;
* autenticación y roles;
* validación de campos relevantes;
* trazabilidad de la fuente;
* apoyo a la revisión humana de las fichas.

### Fuera del alcance obligatorio actual

En esta versión no se consideran requisitos obligatorios:

* extracción o validación automática de imágenes;
* precio o tarifa rack;
* elusión de CAPTCHA;
* elusión de autenticación o paywalls;
* acceso a contenido privado o restringido;
* procesamiento de documentos antiguos `.doc`;
* extracción perfecta desde cualquier sitio web.

### Mejoras futuras

Entre las mejoras consideradas se encuentran:

* gestión y validación de imágenes;
* incorporación de precio/tarifa rack cuando sea necesario;
* ampliación de reglas de extracción;
* soporte para nuevas fuentes;
* monitoreo de información desactualizada;
* completar el ciclo activar/desactivar actividades desde administración;
* pruebas automatizadas;
* auditoría de modificaciones;
* métricas de calidad de extracción;
* parametrización de CORS;
* HTTPS;
* gestión centralizada de secretos;
* monitoreo para un eventual ambiente productivo.

---

## Estado del proyecto

| Componente                           | Estado                        |
| ------------------------------------ | ----------------------------- |
| Backend Spring Boot                  | ✅ Funcional                   |
| Frontend React                       | ✅ Funcional                   |
| PostgreSQL                           | ✅ Funcional                   |
| Scraper web FastAPI                  | ✅ Funcional                   |
| Extractor PDF/DOCX                   | ✅ Funcional                   |
| Autenticación y roles                | ✅ Funcional                   |
| Pruebas de carga iniciales           | ✅ Realizadas                  |
| Seguridad para producción            | ⏳ Pendiente de endurecimiento |
| Reactivación completa de actividades | ⏳ Mejora pendiente            |
| Imágenes                             | 🔜 Mejora futura              |
| Precio / tarifa rack                 | 🔜 Mejora futura              |
| Despliegue productivo                | ⏳ Pendiente                   |

---

## Autoría

**Tour Search Platform** fue desarrollado por **Katherine Hermosilla Ortega**, estudiante de **Ingeniería Informática**, en el marco de su práctica profesional y como respuesta a una necesidad identificada durante su participación en el área de **Customer Success**.

El desarrollo de la solución —incluyendo frontend, backend, base de datos, scraper web, extractor documental e integración de sus componentes— corresponde al trabajo realizado durante esta experiencia.

La plataforma fue creada como una herramienta informática de apoyo para **facilitar el trabajo de las personas del área, reducir tareas manuales repetitivas y contribuir al ahorro de tiempo en la búsqueda y organización de información turística**.

Su propósito es apoyar a las personas que realizan estas tareas y permitirles disponer de información previamente procesada de manera más rápida. La plataforma mantiene la **revisión y validación humana** y no reemplaza los procesos, herramientas ni sistemas oficiales de la organización.

---

**Tour Search Platform — Proyecto de práctica profesional de Ingeniería Informática — 2026**
