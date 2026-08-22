# CLASE 1 — FUNDAMENTOS, MVT E INTRODUCCIÓN A DJANGO

**Curso:** Desarrollo Web con Django  
**Duración:** 2 horas  
**Clase:** 1  
**Proyecto del curso:** Sistema de Inventario

---

# 1. Objetivos de la clase

Al finalizar la clase, el estudiante será capaz de:

- Comprender cómo funciona una aplicación web.
- Diferenciar frontend y backend.
- Comprender qué son HTTP, Request y Response.
- Entender el patrón MVT.
- Identificar Model, View y Template.
- Comprender el flujo de una solicitud dentro de una aplicación.
- Explicar qué es Django.
- Entender por qué Django utiliza MVT.
- Instalar Python y Django.
- Crear un entorno virtual.
- Crear un proyecto Django.
- Identificar la estructura básica de un proyecto.
- Comprender el propósito de `manage.py`, `settings.py` y `urls.py`.
- Diferenciar un proyecto Django de una aplicación Django.
- Crear una primera aplicación.
- Ejecutar el servidor de desarrollo.

---

# PARTE I — ENTENDIENDO LAS APLICACIONES WEB

## 2. ¿Qué es una aplicación web?

Antes de aprender Django debemos entender qué estamos construyendo.

Una aplicación web es un software al que normalmente accedemos mediante un navegador y que se comunica utilizando protocolos web, principalmente HTTP.

Ejemplos:

- Sistemas de inventario.
- Bancos en línea.
- Tiendas electrónicas.
- Sistemas educativos.
- Redes sociales.
- Sistemas empresariales.

Cuando un usuario utiliza una aplicación web normalmente existe una comunicación entre:

```text
USUARIO
   │
   │
   ▼
NAVEGADOR
   │
   │ HTTP
   ▼
SERVIDOR
   │
   ▼
APLICACIÓN
   │
   ▼
BASE DE DATOS
```

---

# 3. Frontend y Backend

Una aplicación web normalmente puede dividirse en dos grandes partes.

## Frontend

Es la parte con la que interactúa directamente el usuario.

Incluye:

- HTML.
- CSS.
- JavaScript.
- React.
- Angular.
- Vue.

Ejemplo:

```text
┌──────────────────────────────┐
│       SISTEMA INVENTARIO     │
├──────────────────────────────┤
│                              │
│ Productos                    │
│                              │
│ [ Buscar producto... ]       │
│                              │
│ Código  Producto      Stock  │
│ 001     Teclado       20     │
│ 002     Mouse         15     │
│                              │
└──────────────────────────────┘
```

---

## Backend

Es la parte que funciona en el servidor.

Se encarga de:

- Procesar solicitudes.
- Ejecutar reglas de negocio.
- Validar información.
- Consultar la base de datos.
- Gestionar usuarios.
- Aplicar permisos.
- Generar respuestas.
- Proporcionar APIs.

Django pertenece principalmente al backend.

---

# 4. ¿Qué es HTTP?

HTTP significa:

> **HyperText Transfer Protocol**

Es uno de los protocolos utilizados para la comunicación entre clientes y servidores web.

Por ejemplo:

```text
Navegador
    │
    │ HTTP Request
    ▼
Servidor
    │
    │ HTTP Response
    ▼
Navegador
```

Cuando visitamos:

```text
http://localhost:8000/productos/
```

el navegador realiza una solicitud al servidor.

---

# 5. Request y Response

Dos conceptos fundamentales:

## Request

Es la solicitud que realiza el cliente.

Por ejemplo:

```text
GET /productos/
```

Significa:

> "Quiero obtener los productos."

## Response

Es la respuesta que devuelve el servidor.

Puede contener:

- HTML.
- JSON.
- Archivos.
- Imágenes.
- Información de error.

Conceptualmente:

```text
CLIENTE
   │
   │ Request
   ▼
SERVIDOR
   │
   │ Response
   ▼
CLIENTE
```

Este concepto será fundamental para entender Django.

---

# PARTE II — MVT

# 6. ¿Qué es MVT?

MVT significa:

```text
M → Model
V → View
T → Template
```

Es el patrón arquitectónico utilizado por Django para organizar la aplicación.

Su objetivo es separar responsabilidades.

```text
MODEL
Datos

VIEW
Lógica

TEMPLATE
Presentación
```

---

# 7. ¿Por qué necesitamos separar responsabilidades?

Imaginemos que tenemos todo nuestro código en un único archivo:

```text
aplicacion.py
```

Dentro tendríamos:

```text
HTML
SQL
Lógica
Validaciones
Usuarios
Productos
Reportes
```

Conforme crece el sistema, sería muy difícil mantenerlo.

La separación permite:

```text
Datos
   ↓
Model

Lógica
   ↓
View

Presentación
   ↓
Template
```

Cada componente tiene una responsabilidad diferente.

---

# 8. M — Model

El Model representa los datos de nuestra aplicación.

En nuestro sistema de inventario podríamos tener:

```text
Producto
──────────────
id
nombre
descripcion
precio
stock
```

También podríamos tener:

```text
Categoría
Proveedor
Usuario
Movimiento
```

Conceptualmente:

```text
MODEL
  │
  ↓
DATOS
  │
  ↓
BASE DE DATOS
```

En Django, posteriormente definiremos estos modelos mediante clases Python.

Por ejemplo, conceptualmente:

```python
class Producto:
    nombre
    precio
    stock
```

Más adelante aprenderemos la implementación real utilizando Django ORM.

---

# 9. V — View

La View contiene la lógica que procesa las solicitudes.

Por ejemplo, el usuario solicita:

```text
GET /productos/
```

La View puede:

1. Recibir la solicitud.
2. Consultar los productos.
3. Procesar la información.
4. Enviar los datos al Template.
5. Devolver una Response.

Conceptualmente:

```text
Request
   ↓
View
   ↓
Procesamiento
   ↓
Response
```

---

# 10. T — Template

El Template se encarga de la presentación.

Normalmente contiene HTML.

Por ejemplo:

```html
<h1>Productos</h1>

<ul>
    <li>Teclado - Bs 150</li>
    <li>Mouse - Bs 80</li>
</ul>
```

El Template recibe datos y los presenta al usuario.

Conceptualmente:

```text
Model
  ↓
Datos
  ↓
View
  ↓
Template
  ↓
HTML
  ↓
Usuario
```

---

# 11. Flujo completo de MVT

Supongamos que el usuario solicita:

```text
GET /productos/
```

El flujo conceptual es:

```text
                 USUARIO
                    │
                    │ Request
                    ▼
                   URL
                    │
                    ▼
                  VIEW
                    │
              ┌─────┴─────┐
              │           │
              ▼           ▼
            MODEL      TEMPLATE
              │           │
              ▼           │
          DATABASE        │
              │           │
              └─────┬─────┘
                    │
                    ▼
                 RESPONSE
                    │
                    ▼
                 USUARIO
```

---

# 12. Ejemplo completo

Supongamos:

```text
GET /productos/
```

### Paso 1 — Request

El navegador solicita:

```text
GET /productos/
```

### Paso 2 — URL

Django determina qué View debe procesar esa dirección.

### Paso 3 — View

La View recibe la solicitud.

### Paso 4 — Model

La View solicita los productos.

```text
View
 ↓
Model
 ↓
Database
```

### Paso 5 — Datos

La base de datos devuelve:

```text
Teclado
Mouse
Monitor
```

### Paso 6 — Template

La View entrega los datos al Template.

### Paso 7 — HTML

El Template genera la página.

### Paso 8 — Response

Django devuelve la respuesta al navegador.

```text
Navegador
    ↓
Request
    ↓
Django
    ↓
URL
    ↓
View
    ↓
Model
    ↓
Database
    ↓
Model
    ↓
View
    ↓
Template
    ↓
Response
    ↓
Navegador
```

---

# 13. MVT y MVC

Es posible que los estudiantes ya conozcan MVC.

MVC significa:

```text
Model
View
Controller
```

Django utiliza:

```text
Model
View
Template
```

Una comparación simplificada es:

| MVC | Django |
|---|---|
| Model | Model |
| View | Template |
| Controller | View |

Sin embargo, no debemos considerar que sean exactamente equivalentes.

En Django, la View asume buena parte de la responsabilidad que normalmente asociamos al Controller.

El Template se encarga principalmente de la presentación.

Lo importante para nosotros es comprender la separación:

```text
Datos
 ↓
Model

Lógica
 ↓
View

Presentación
 ↓
Template
```

---

# PARTE III — ¿QUÉ ES DJANGO?

# 14. ¿Qué es Django?

Ahora que entendemos MVT podemos introducir Django.

Django es un:

> **Framework web de alto nivel desarrollado en Python.**

Permite construir aplicaciones web de forma estructurada y proporciona herramientas para resolver problemas comunes del desarrollo web.

---

# 15. ¿Qué es un framework?

Un framework proporciona una estructura y herramientas para desarrollar software.

Sin framework tendríamos que implementar muchas funcionalidades por nuestra cuenta.

Por ejemplo:

```text
Sin framework
    ↓
Routing
HTTP
Seguridad
Autenticación
Base de datos
Validaciones
Sesiones
etc.
```

Con Django:

```text
             DJANGO
                │
      ┌─────────┼─────────┐
      ↓         ↓         ↓
    URLs      Views     Models
      ↓         ↓         ↓
   Routing    Lógica    ORM
```

Django proporciona la infraestructura para que podamos concentrarnos en la lógica de nuestra aplicación.

---

# 16. ¿Qué problema resuelve Django?

Supongamos que queremos construir nuestro sistema de inventario.

Necesitamos:

```text
Productos
Categorías
Proveedores
Usuarios
Inventario
Entradas
Salidas
Reportes
```

Django nos proporciona herramientas para:

```text
Django
│
├── MVT
├── ORM
├── URLs
├── Views
├── Templates
├── Admin
├── Authentication
├── Security
├── Forms
└── Testing
```

Esto nos permite construir aplicaciones completas sin tener que desarrollar toda la infraestructura desde cero.

---

# 17. ¿Django es frontend o backend?

Django es principalmente un framework backend.

```text
┌───────────────────────┐
│       FRONTEND        │
│                       │
│ HTML / CSS / JS       │
│ React / Angular / Vue │
└───────────┬───────────┘
            │
           HTTP
            │
            ▼
┌───────────────────────┐
│        BACKEND        │
│                       │
│        DJANGO         │
│        Python         │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│       DATABASE        │
│                       │
│ SQLite / PostgreSQL   │
│ MySQL                 │
└───────────────────────┘
```

Django también puede generar HTML utilizando Templates.

Además, posteriormente podemos utilizar Django REST Framework para construir APIs.

---

# PARTE IV — PREPARACIÓN DEL ENTORNO

# 18. ¿Qué necesitamos?

Nuestro entorno será:

```text
Python
   ↓
pip
   ↓
venv
   ↓
Django
   ↓
VS Code
   ↓
Git
```

---

# 19. ¿Qué es pip?

`pip` es el administrador de paquetes de Python.

Permite instalar librerías.

Por ejemplo:

```bash
pip install Django
```

También permite actualizar:

```bash
pip install --upgrade Django
```

Y posteriormente podremos generar nuestras dependencias:

```bash
pip freeze > requirements.txt
```

---

# 20. ¿Qué es venv?

`venv` es una herramienta incluida con Python para crear entornos virtuales.

Ejemplo:

```text
Proyecto A
└── .venv
    └── Django 6

Proyecto B
└── .venv
    └── Django 5
```

Cada proyecto puede tener sus propias dependencias.

---

# PARTE V — INSTALACIÓN

# 21. Windows

Comprobar Python:

```powershell
py --version
```

Crear entorno:

```powershell
py -m venv .venv
```

Activarlo:

```powershell
.venv\Scripts\Activate.ps1
```

Actualizar pip:

```powershell
py -m pip install --upgrade pip
```

Instalar Django:

```powershell
py -m pip install Django
```

Verificar:

```powershell
py -m django --version
```

---

# 22. macOS / Linux

Comprobar Python:

```bash
python3 --version
```

Crear entorno:

```bash
python3 -m venv .venv
```

Activar:

```bash
source .venv/bin/activate
```

Actualizar pip:

```bash
python -m pip install --upgrade pip
```

Instalar Django:

```bash
python -m pip install Django
```

Verificar:

```bash
python -m django --version
```

---

# 23. ¿Cómo sabemos que el entorno virtual está activo?

En macOS/Linux normalmente veremos:

```text
(.venv)
```

al principio de la terminal.

También podemos comprobar:

```bash
echo $VIRTUAL_ENV
```

Y:

```bash
which python
```

La ruta debería apuntar a:

```text
.../.venv/bin/python
```

En Windows:

```powershell
where python
```

debería mostrar la ruta del Python dentro del entorno virtual.

---

# PARTE VI — CREANDO NUESTRO PRIMER PROYECTO

# 24. Crear la carpeta

```bash
mkdir curso-django
cd curso-django
```

---

# 25. Crear el proyecto

```bash
django-admin startproject configuracion .
```

El punto final significa:

> Crear el proyecto dentro del directorio actual.

---

# 26. Estructura inicial

Obtendremos:

```text
curso-django/
│
├── .venv/
│
├── manage.py
│
└── configuracion/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

---

# 27. ¿Qué es manage.py?

`manage.py` es nuestra herramienta de línea de comandos para administrar el proyecto.

Ejemplos:

```bash
python manage.py runserver
```

Inicia el servidor.

```bash
python manage.py startapp productos
```

Crea una aplicación.

```bash
python manage.py migrate
```

Ejecuta migraciones.

```bash
python manage.py createsuperuser
```

Crea un administrador.

Por eso podemos decir:

> `manage.py` es la puerta de entrada al CLI de nuestro proyecto Django.

---

# 28. ¿Qué es settings.py?

Contiene la configuración principal del proyecto.

Encontraremos configuraciones como:

```text
SECRET_KEY
DEBUG
ALLOWED_HOSTS
INSTALLED_APPS
MIDDLEWARE
DATABASES
LANGUAGE_CODE
TIME_ZONE
STATIC_URL
```

Una de las configuraciones más importantes es:

```python
INSTALLED_APPS = [
    ...
]
```

Aquí registraremos nuestras aplicaciones.

También encontraremos:

```python
DATABASES = {
    ...
}
```

que contiene la configuración de la base de datos.

Por defecto Django utiliza SQLite.

---

# 29. ¿Qué es urls.py?

Define las rutas de nuestro proyecto.

Por ejemplo:

```text
/
/productos/
/clientes/
/ventas/
```

Conceptualmente:

```text
Request
   ↓
URL
   ↓
¿Qué View debe ejecutarse?
```

Más adelante conectaremos nuestras URLs con las Views.

---

# 30. ¿Qué son asgi.py y wsgi.py?

Son puntos de entrada utilizados para ejecutar Django mediante servidores compatibles con ASGI y WSGI.

Por ahora solo debemos conocer su función general.

No profundizaremos en ellos durante esta clase.

---

# 31. ¿Qué es __init__.py?

Es un archivo relacionado con los paquetes de Python.

Permite que el directorio sea tratado como un paquete Python.

Por ejemplo:

```text
configuracion/
├── __init__.py
├── settings.py
└── urls.py
```

---

# PARTE VII — EJECUTANDO DJANGO

# 32. Iniciar el servidor

Ejecutamos:

```bash
python manage.py runserver
```

En Windows podemos utilizar:

```powershell
py manage.py runserver
```

Django mostrará:

```text
Starting development server at
http://127.0.0.1:8000/
```

Abrimos:

```text
http://127.0.0.1:8000/
```

---

# 33. ¿Qué significa 127.0.0.1:8000?

`127.0.0.1` representa nuestra propia computadora.

También podemos utilizar:

```text
localhost
```

Por lo tanto:

```text
127.0.0.1
```

significa:

> Esta computadora.

Y:

```text
8000
```

es el puerto donde está escuchando nuestro servidor de desarrollo.

Entonces:

```text
http://127.0.0.1:8000/
```

significa que estamos accediendo al servidor Django que está ejecutándose localmente.

---

# PARTE VIII — PROYECTO VS APP

# 34. ¿Qué es un proyecto Django?

El proyecto representa la aplicación o sistema completo.

Por ejemplo:

```text
Sistema de Inventario
```

Puede contener diferentes funcionalidades.

---

# 35. ¿Qué es una app?

Una app es un módulo que implementa una funcionalidad concreta.

Nuestro sistema podría tener:

```text
Sistema de Inventario
│
├── productos
├── categorias
├── proveedores
├── inventario
├── usuarios
└── reportes
```

Podemos pensar:

```text
Proyecto
   ↓
Sistema completo

App
   ↓
Módulo funcional
```

---

# 36. Crear nuestra primera app

Ejecutamos:

```bash
python manage.py startapp productos
```

Obtendremos:

```text
curso-django/
│
├── .venv/
│
├── manage.py
│
├── configuracion/
│
└── productos/
    ├── migrations/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    └── views.py
```

---

# 37. ¿Qué contiene una app?

## models.py

Representará nuestros datos.

```text
Producto
Categoría
Proveedor
```

Aquí implementaremos el **M de MVT**.

---

## views.py

Contendrá las Views.

Aquí implementaremos el **V de MVT**.

---

## templates/

Aquí colocaremos nuestros Templates.

Aquí implementaremos el **T de MVT**.

---

## admin.py

Permite configurar modelos para el administrador de Django.

---

## apps.py

Contiene la configuración de la aplicación.

---

## tests.py

Contendrá nuestras pruebas automatizadas.

---

## migrations/

Contendrá las migraciones relacionadas con los modelos.

---

# 38. Relacionando MVT con Django

Ahora podemos conectar todo lo aprendido:

```text
             DJANGO
                │
                ↓
               MVT
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
     MODEL     VIEW   TEMPLATE
       │        │        │
       ↓        ↓        ↓
 models.py  views.py  templates/
       │
       ↓
  DATABASE
```

Esto es fundamental:

> No queremos que los estudiantes memoricen archivos. Queremos que comprendan qué responsabilidad representa cada archivo.

---

# PARTE IX — PRIMER EJERCICIO

# 39. Ejercicio práctico

Cada alumno deberá crear un proyecto llamado:

```text
inventario
```

Dentro deberá crear una app:

```text
productos
```

La estructura esperada será aproximadamente:

```text
inventario/
│
├── .venv/
│
├── manage.py
│
├── configuracion/
│
└── productos/
    ├── migrations/
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    └── views.py
```

Luego deberá ejecutar:

```bash
python manage.py runserver
```

Y comprobar:

```text
http://127.0.0.1:8000/
```

---

# 40. Preguntas para el estudiante

## Pregunta 1

### ¿Qué es Python?

Python es un lenguaje de programación de propósito general utilizado, entre otras cosas, para desarrollar aplicaciones web.

---

## Pregunta 2

### ¿Qué es un framework?

Es una estructura de trabajo que proporciona herramientas, componentes y convenciones para facilitar el desarrollo de aplicaciones.

---

## Pregunta 3

### ¿Qué es Django?

Django es un framework web de alto nivel desarrollado en Python para construir aplicaciones web.

---

## Pregunta 4

### ¿Django es un lenguaje de programación?

No.

```text
Python → lenguaje de programación
Django → framework
```

---

## Pregunta 5

### ¿Qué significa MVT?

```text
M → Model
V → View
T → Template
```

Es un patrón utilizado por Django para separar responsabilidades dentro de una aplicación.

---

## Pregunta 6

### ¿Qué hace el Model?

Representa los datos y la lógica relacionada con ellos.

Normalmente está relacionado con la base de datos.

---

## Pregunta 7

### ¿Qué hace la View?

Procesa las solicitudes y contiene la lógica necesaria para generar una respuesta.

---

## Pregunta 8

### ¿Qué hace el Template?

Se encarga principalmente de presentar los datos al usuario, normalmente mediante HTML.

---

## Pregunta 9

### ¿Cuál es el flujo de MVT?

```text
Request
   ↓
URL
   ↓
View
   ↓
Model
   ↓
Database
   ↓
View
   ↓
Template
   ↓
Response
```

---

## Pregunta 10

### ¿Qué es pip?

Es el administrador de paquetes de Python.

---

## Pregunta 11

### ¿Qué es un entorno virtual?

Es un entorno aislado donde instalamos las dependencias de un proyecto sin afectar otros proyectos.

---

## Pregunta 12

### ¿Para qué sirve manage.py?

Permite administrar el proyecto Django desde la línea de comandos.

---

## Pregunta 13

### ¿Para qué sirve settings.py?

Contiene la configuración principal del proyecto.

---

## Pregunta 14

### ¿Para qué sirve urls.py?

Define las rutas que Django debe atender.

---

## Pregunta 15

### ¿Qué diferencia hay entre proyecto y app?

El proyecto representa el sistema completo.

Una app representa una funcionalidad o módulo del sistema.

---

# PARTE X — RESUMEN VISUAL

La idea principal que debe quedar de esta clase es:

```text
                    APLICACIÓN WEB
                          │
             ┌────────────┴────────────┐
             │                         │
          FRONTEND                  BACKEND
             │                         │
             │                       DJANGO
             │                         │
             │                        MVT
             │                         │
             │            ┌────────────┼────────────┐
             │            │            │            │
             │          Model         View       Template
             │            │            │            │
             │            ↓            ↓            ↓
             │         Datos        Lógica     Presentación
             │            │
             │            ↓
             │       Base de datos
             │
             └────────── HTTP ───────────┘
```

---

# 41. Flujo completo que debemos recordar

```text
USUARIO
   │
   │ HTTP Request
   ↓
DJANGO
   │
   ↓
URL
   │
   ↓
VIEW
   │
   ├──────────────→ MODEL
   │                   │
   │                   ↓
   │               DATABASE
   │                   │
   │                   ↓
   │←──────────────────┘
   │
   ↓
TEMPLATE
   │
   ↓
HTML
   │
   ↓
HTTP Response
   │
   ↓
USUARIO
```

---

# 42. Comandos fundamentales de la clase

## Crear entorno virtual

### Windows

```powershell
py -m venv .venv
```

### macOS/Linux

```bash
python3 -m venv .venv
```

---

## Activar entorno

### Windows

```powershell
.venv\Scripts\Activate.ps1
```

### macOS/Linux

```bash
source .venv/bin/activate
```

---

## Instalar Django

```bash
python -m pip install Django
```

---

## Comprobar Django

```bash
python -m django --version
```

---

## Crear proyecto

```bash
django-admin startproject configuracion .
```

---

## Ejecutar servidor

```bash
python manage.py runserver
```

---

## Crear app

```bash
python manage.py startapp productos
```

---

# 43. Tarea para la siguiente clase

Crear un proyecto llamado:

```text
inventario
```

Crear una app llamada:

```text
productos
```

Y responder:

1. ¿Qué es un modelo?
2. ¿Qué es una base de datos?
3. ¿Qué es una migración?
4. ¿Qué es un ORM?
5. ¿Qué relación existe entre un modelo Django y una tabla de base de datos?
6. ¿Qué ventajas tiene utilizar un ORM?
7. ¿Qué ocurre cuando modificamos un modelo?

En la siguiente clase utilizaremos estas respuestas para comenzar a implementar el **Model de nuestro sistema de inventario**.

---

# 44. Cierre de la clase

La primera clase no busca que el estudiante memorice Django.

Busca que comprenda esta idea:

> **Django es una herramienta que nos proporciona una estructura para construir aplicaciones web en Python.**

Y que entienda el flujo:

```text
Web
 ↓
HTTP
 ↓
Request
 ↓
Django
 ↓
MVT
 ├── Model
 ├── View
 └── Template
 ↓
Response
 ↓
Usuario
```

A partir de aquí, las siguientes clases dejarán de ser una colección de comandos y comenzarán a construir progresivamente nuestro **Sistema de Inventario**.