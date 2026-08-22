# Clase 2 — Django: Usuarios, Autenticación, Autorización y Middleware

## Información de la clase

**Duración:** 2 horas  
**Tema:** Usuarios, autenticación, autorización y middleware en Django  
**Proyecto:** Sistema de inventario

---

# 1. Objetivos de la clase

Al finalizar esta clase, el estudiante será capaz de:

- Comprender qué es un usuario en Django.
- Diferenciar autenticación y autorización.
- Utilizar el sistema de usuarios incorporado de Django.
- Crear usuarios desde el panel de administración.
- Crear un superusuario.
- Implementar inicio y cierre de sesión.
- Utilizar `request.user`.
- Proteger vistas utilizando `login_required`.
- Comprender `is_staff` e `is_superuser`.
- Comprender el funcionamiento básico de grupos y permisos.
- Entender qué es un middleware.
- Comprender el funcionamiento de `AuthenticationMiddleware`.
- Crear un middleware sencillo.
- Integrar autenticación y autorización en el proyecto.

---

# 2. Recordatorio de la clase anterior

En la clase anterior estudiamos:

- Qué es Django.
- Arquitectura MVT.
- Proyecto y aplicaciones.
- `manage.py`.
- Estructura básica de un proyecto.
- URLs.
- Views.
- Templates.
- Request y Response.

La idea principal que debemos recordar es:

```text
NAVEGADOR
    ↓
URL
    ↓
VIEW
    ↓
TEMPLATE
    ↓
RESPONSE
    ↓
NAVEGADOR
```

Hoy vamos a agregar un nuevo componente importante:

```text
USUARIO
    ↓
AUTENTICACIÓN
    ↓
AUTORIZACIÓN
    ↓
VIEW
```

---

# 3. Concepto fundamental: usuario

Un sistema web normalmente necesita saber:

- ¿Quién está utilizando el sistema?
- ¿Está registrado?
- ¿Ha iniciado sesión?
- ¿Qué puede hacer?
- ¿Qué información puede consultar?
- ¿Qué información puede modificar?

Django ya proporciona un sistema de usuarios.

No necesitamos crear desde cero:

- Tabla de usuarios.
- Contraseñas.
- Login.
- Logout.
- Manejo de sesiones.
- Sistema básico de permisos.

Django proporciona todo esto mediante su sistema de autenticación.

---

# 4. El modelo User de Django

Django incluye un modelo de usuario llamado:

```python
User
```

Este modelo pertenece a:

```python
django.contrib.auth
```

Algunos de sus campos importantes son:

```text
id
username
password
first_name
last_name
email
is_staff
is_active
is_superuser
date_joined
last_login
```

La contraseña no se almacena directamente.

Django almacena una versión protegida mediante hashing.

Por ejemplo:

```text
Contraseña:
mi_clave_123
```

No se almacena como:

```text
mi_clave_123
```

Sino como un valor generado mediante un algoritmo de hash.

---

# 5. Autenticación vs autorización

Esta es una de las diferencias más importantes de la clase.

## Autenticación

La autenticación responde:

> ¿Quién eres?

Ejemplo:

```text
Usuario: juan
Contraseña: ********
```

El sistema verifica las credenciales.

Si son correctas:

```text
Usuario autenticado
```

---

## Autorización

La autorización responde:

> ¿Qué puedes hacer?

Por ejemplo:

```text
Juan está autenticado.
¿Puede eliminar productos?
```

La respuesta podría ser:

```text
Sí
```

o:

```text
No
```

---

## Resumen

```text
AUTENTICACIÓN
    ↓
¿Quién eres?

AUTORIZACIÓN
    ↓
¿Qué puedes hacer?
```

Una persona puede estar autenticada pero no tener autorización para realizar determinada acción.

---

# 6. El sistema de autenticación de Django

Django incluye una aplicación llamada:

```python
django.contrib.auth
```

Normalmente ya aparece en:

```python
INSTALLED_APPS
```

Por ejemplo:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Estas aplicaciones forman parte del funcionamiento básico de Django.

---

# 7. Migraciones y usuarios

El sistema de autenticación necesita tablas en la base de datos.

Por eso debemos ejecutar:

```bash
python manage.py migrate
```

Esto crea las tablas necesarias.

Entre ellas están las relacionadas con:

- Usuarios.
- Grupos.
- Permisos.
- Sesiones.

---

# 8. Crear un superusuario

Para administrar usuarios podemos utilizar el panel de administración de Django.

Primero creamos un superusuario:

```bash
python manage.py createsuperuser
```

Django solicitará:

```text
Username:
Email address:
Password:
Password (again):
```

Por ejemplo:

```text
Username: juan
Email address: juan@example.com
Password:
Password (again):
```

Después iniciamos el servidor:

```bash
python manage.py runserver
```

Y accedemos a:

```text
http://127.0.0.1:8000/admin/
```

---

# 9. ¿Qué es un superusuario?

Un superusuario tiene todos los permisos disponibles en Django.

Se identifica mediante:

```python
is_superuser = True
```

Por ejemplo:

```python
request.user.is_superuser
```

devuelve:

```python
True
```

si el usuario es superusuario.

---

# 10. is_staff vs is_superuser

Estos dos conceptos suelen confundirse.

## is_staff

Indica que el usuario puede acceder al sitio de administración de Django, siempre sujeto a los permisos que tenga.

```python
user.is_staff
```

---

## is_superuser

Indica que el usuario tiene todos los permisos.

```python
user.is_superuser
```

---

## Comparación

| Usuario | is_staff | is_superuser |
|---|---:|---:|
| Usuario normal | False | False |
| Personal administrativo | True | False |
| Superusuario | True | True |

Un usuario puede ser:

```text
is_staff = True
is_superuser = False
```

Esto significa que puede entrar al Admin, pero no necesariamente tiene todos los permisos.

---

# 11. Crear usuarios desde el Admin

Desde:

```text
/admin/
```

podemos administrar usuarios.

Django permite crear:

- Usuarios.
- Grupos.
- Permisos.

Al crear un usuario podemos configurar:

```text
Username
Password
First name
Last name
Email
Staff status
Active
Superuser status
Groups
User permissions
```

---

# 12. Crear un usuario normal

Podemos crear, por ejemplo:

```text
Usuario: carlos
```

Este usuario puede tener:

```text
is_staff = False
is_superuser = False
```

Por lo tanto:

```text
Usuario normal
```

---

# 13. ¿Cómo sabe Django quién está conectado?

Django utiliza:

```python
request.user
```

Esta propiedad representa al usuario actual.

Por ejemplo:

```python
def dashboard(request):
    usuario = request.user

    return render(
        request,
        'dashboard.html',
        {
            'usuario': usuario
        }
    )
```

En el template podemos utilizar:

```html
<h1>Bienvenido {{ usuario.username }}</h1>
```

---

# 14. request.user

`request.user` es uno de los conceptos más importantes de esta clase.

Podemos consultar información del usuario:

```python
request.user.username
```

```python
request.user.email
```

```python
request.user.first_name
```

```python
request.user.last_name
```

```python
request.user.is_staff
```

```python
request.user.is_superuser
```

---

# 15. ¿Qué ocurre si no inició sesión?

Django utiliza un usuario especial:

```python
AnonymousUser
```

Por eso podemos comprobar:

```python
request.user.is_authenticated
```

Si el usuario inició sesión:

```python
True
```

Si no inició sesión:

```python
False
```

---

# 16. Comprobar autenticación

Ejemplo:

```python
def dashboard(request):

    if request.user.is_authenticated:
        return render(request, 'dashboard.html')

    return redirect('login')
```

La lógica es:

```text
¿Está autenticado?
       ↓
      Sí
       ↓
Mostrar dashboard

       ↓
      No
       ↓
Enviar al login
```

---

# 17. login_required

Django proporciona un decorador para proteger vistas:

```python
@login_required
```

Primero importamos:

```python
from django.contrib.auth.decorators import login_required
```

Después:

```python
@login_required
def dashboard(request):
    return render(request, 'dashboard.html')
```

Ahora solamente los usuarios autenticados podrán acceder.

---

# 18. ¿Qué sucede si no está autenticado?

Si alguien intenta acceder:

```text
/dashboard/
```

sin haber iniciado sesión, Django lo enviará al login configurado.

Por ejemplo:

```text
/login/?next=/dashboard/
```

El parámetro:

```text
next
```

indica a Django a dónde debe regresar después de iniciar sesión.

---

# 19. Configurar LOGIN_URL

En:

```text
settings.py
```

podemos definir:

```python
LOGIN_URL = '/login/'
```

También podemos configurar:

```python
LOGIN_REDIRECT_URL = '/'
```

Esto indica a Django dónde enviar al usuario después de iniciar sesión.

---

# 20. Login

Django proporciona herramientas para manejar autenticación.

Podemos utilizar:

```python
authenticate()
```

para comprobar las credenciales.

Ejemplo:

```python
from django.contrib.auth import authenticate
```

Después:

```python
usuario = authenticate(
    username=username,
    password=password
)
```

Si las credenciales son correctas:

```python
usuario
```

contendrá el usuario.

Si son incorrectas:

```python
usuario = None
```

---

# 21. Iniciar sesión

Después de autenticar al usuario podemos utilizar:

```python
login(request, usuario)
```

Importamos:

```python
from django.contrib.auth import authenticate, login
```

Ejemplo:

```python
def iniciar_sesion(request):

    if request.method == 'POST':

        username = request.POST['username']
        password = request.POST['password']

        usuario = authenticate(
            username=username,
            password=password
        )

        if usuario is not None:
            login(request, usuario)

            return redirect('dashboard')

    return render(request, 'login.html')
```

---

# 22. Cerrar sesión

Para cerrar sesión utilizamos:

```python
logout(request)
```

Importamos:

```python
from django.contrib.auth import logout
```

Ejemplo:

```python
def cerrar_sesion(request):

    logout(request)

    return redirect('login')
```

---

# 23. Flujo completo del Login

Podemos explicarlo en la pizarra:

```text
Usuario
   ↓
Formulario Login
   ↓
username + password
   ↓
authenticate()
   ↓
¿Credenciales correctas?
   ↓
   Sí
   ↓
login()
   ↓
Sesión
   ↓
request.user
```

---

# 24. Sesiones

Django utiliza sesiones para recordar que un usuario inició sesión.

Conceptualmente:

```text
LOGIN
  ↓
Django crea/actualiza sesión
  ↓
Cookie de sesión
  ↓
Navegador
  ↓
Request posteriores
  ↓
Django identifica al usuario
```

Por eso podemos utilizar:

```python
request.user
```

en diferentes vistas.

---

# 25. Cookies

Una cookie es información almacenada por el navegador.

Django utiliza cookies relacionadas con la sesión.

Podemos observarlas desde las herramientas de desarrollador del navegador.

Conceptualmente:

```text
Servidor
   ↓
Cookie
   ↓
Navegador
```

En las siguientes peticiones:

```text
Navegador
   ↓
Cookie
   ↓
Servidor
```

---

# 26. AuthenticationMiddleware

Aquí aparece un concepto muy importante:

```text
Middleware
```

Django utiliza:

```python
AuthenticationMiddleware
```

para asociar al request el usuario autenticado.

Por eso podemos escribir:

```python
request.user
```

---

# 27. ¿Qué es un Middleware?

Un middleware es una capa que se ejecuta durante el procesamiento de una petición.

Podemos imaginar:

```text
NAVEGADOR
    ↓
Middleware
    ↓
Middleware
    ↓
Middleware
    ↓
VIEW
    ↓
Response
    ↓
Middleware
    ↓
NAVEGADOR
```

El middleware puede realizar tareas antes o después de una vista.

---

# 28. Ejemplos de Middleware

Django utiliza middleware para tareas como:

- Sesiones.
- Autenticación.
- Seguridad.
- Protección CSRF.
- Mensajes.
- Manejo de requests y responses.

En:

```text
settings.py
```

encontramos:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]
```

---

# 29. AuthenticationMiddleware

La línea:

```python
'django.contrib.auth.middleware.AuthenticationMiddleware',
```

es fundamental para trabajar con:

```python
request.user
```

Su función es conectar el sistema de autenticación con el objeto request.

Conceptualmente:

```text
Sesión
  ↓
AuthenticationMiddleware
  ↓
request.user
```

---

# 30. MVT + Middleware

Hasta ahora teníamos:

```text
URL
 ↓
VIEW
 ↓
TEMPLATE
 ↓
RESPONSE
```

Ahora debemos visualizar:

```text
REQUEST
   ↓
MIDDLEWARE
   ↓
URL
   ↓
VIEW
   ↓
TEMPLATE
   ↓
RESPONSE
   ↓
MIDDLEWARE
   ↓
RESPONSE
```

---

# 31. Autorización

Ahora que sabemos quién es el usuario, necesitamos decidir qué puede hacer.

Por ejemplo:

```text
Juan
 ↓
Autenticado
 ↓
¿Puede administrar usuarios?
```

Mientras que:

```text
Carlos
 ↓
Autenticado
 ↓
¿Puede administrar usuarios?
```

La respuesta puede ser diferente.

---

# 32. Autorización mediante is_superuser

Para nuestra primera implementación podemos hacer:

```python
if request.user.is_superuser:
    ...
```

Ejemplo:

```python
@login_required
def administracion(request):

    if not request.user.is_superuser:
        return HttpResponseForbidden(
            'No tienes permiso para acceder'
        )

    return render(
        request,
        'administracion.html'
    )
```

---

# 33. HttpResponseForbidden

Django proporciona:

```python
HttpResponseForbidden
```

para responder con:

```text
HTTP 403 Forbidden
```

Ejemplo:

```python
from django.http import HttpResponseForbidden
```

Después:

```python
return HttpResponseForbidden(
    'No tienes permiso para acceder'
)
```

---

# 34. Autenticación + autorización

Una vista puede necesitar ambas cosas.

Primero:

```python
@login_required
```

Esto comprueba:

```text
¿Está autenticado?
```

Después:

```python
request.user.is_superuser
```

comprueba:

```text
¿Tiene autorización?
```

Por lo tanto:

```text
Autenticación
      ↓
¿Quién eres?

      ↓

Autorización
      ↓
¿Qué puedes hacer?
```

---

# 35. Grupos

Django también proporciona grupos.

Un grupo permite agrupar usuarios que tienen responsabilidades similares.

Ejemplo:

```text
Administradores
Vendedores
Almacén
Supervisores
```

Podríamos tener:

```text
Grupo: Almacén
    ↓
Juan
Carlos
Pedro
```

---

# 36. Permisos

Django también proporciona permisos.

Por ejemplo, para un modelo:

```text
Producto
```

podemos tener permisos como:

```text
add_producto
change_producto
delete_producto
view_producto
```

Esto permite controlar acciones específicas.

---

# 37. Grupos + permisos

La idea es:

```text
USUARIO
   ↓
GRUPO
   ↓
PERMISOS
```

Por ejemplo:

```text
Carlos
   ↓
Almacén
   ↓
view_producto
add_producto
change_producto
```

Mientras que otro usuario podría tener:

```text
Supervisor
   ↓
view_producto
change_producto
delete_producto
```

---

# 38. Nota importante

En esta clase veremos los conceptos de:

- Usuarios.
- Autenticación.
- Autorización.
- Grupos.
- Permisos.

La implementación completa de permisos sobre nuestros modelos del sistema de inventario la desarrollaremos posteriormente.

Por ahora nos enfocaremos principalmente en comprender:

```text
Usuario
    ↓
Autenticación
    ↓
Autorización
```

---

# 39. Práctica de la clase

Vamos a integrar todo lo aprendido.

Crearemos:

```text
Usuario
    ↓
Login
    ↓
Dashboard
    ↓
Administración
```

Tendremos dos usuarios:

```text
juan
carlos
```

Donde:

```text
juan
    ↓
Superusuario
```

y:

```text
carlos
    ↓
Usuario normal
```

---

# 40. Vista Dashboard

En nuestra aplicación podemos crear:

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render


@login_required
def dashboard(request):
    return render(
        request,
        'dashboard.html'
    )
```

---

# 41. Vista de administración

```python
from django.contrib.auth.decorators import login_required
from django.http import HttpResponseForbidden
from django.shortcuts import render


@login_required
def administracion(request):

    if not request.user.is_superuser:
        return HttpResponseForbidden(
            'No tienes permiso para acceder'
        )

    return render(
        request,
        'administracion.html'
    )
```

---

# 42. URLs

En:

```text
urls.py
```

podemos configurar:

```python
from django.urls import path

from . import views


urlpatterns = [
    path(
        'dashboard/',
        views.dashboard,
        name='dashboard'
    ),

    path(
        'administracion/',
        views.administracion,
        name='administracion'
    ),
]
```

---

# 43. Template Dashboard

Archivo:

```text
dashboard.html
```

Contenido:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Dashboard</title>
</head>
<body>

    <h1>Dashboard</h1>

    <p>
        Bienvenido {{ request.user.username }}
    </p>

    <p>
        Tu correo es:
        {{ request.user.email }}
    </p>

    <a href="{% url 'administracion' %}">
        Administración
    </a>

</body>
</html>
```

---

# 44. Template de administración

Archivo:

```text
administracion.html
```

Contenido:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Administración</title>
</head>
<body>

    <h1>Administración</h1>

    <p>
        Bienvenido administrador
        {{ request.user.username }}
    </p>

</body>
</html>
```

---

# 45. Probar la autenticación

Primero iniciamos sesión como:

```text
juan
```

Debemos comprobar:

```text
Login
  ↓
Dashboard
  ↓
Administración
```

Como Juan es superusuario:

```python
request.user.is_superuser
```

debe devolver:

```python
True
```

---

# 46. Probar autorización

Ahora iniciamos sesión como:

```text
carlos
```

Carlos puede acceder a:

```text
/dashboard/
```

porque está autenticado.

Pero cuando intenta acceder a:

```text
/administracion/
```

debe obtener:

```text
403 Forbidden
```

Esto demuestra que:

```text
AUTENTICACIÓN
```

y:

```text
AUTORIZACIÓN
```

son conceptos diferentes.

---

# 47. Ejercicio para los estudiantes

Crear el siguiente flujo:

```text
/login/
    ↓
Dashboard
    ↓
/administracion/
```

Debe cumplir:

### Usuario no autenticado

No puede acceder a:

```text
/dashboard/
```

Debe ser enviado al login.

### Usuario autenticado

Puede acceder a:

```text
/dashboard/
```

### Superusuario

Puede acceder a:

```text
/administracion/
```

### Usuario normal

No puede acceder a:

```text
/administracion/
```

Debe recibir:

```text
403 Forbidden
```

---

# 48. Preguntas para los estudiantes

## Pregunta 1

¿Cuál es la diferencia entre autenticación y autorización?

Respuesta esperada:

```text
Autenticación:
¿Quién eres?

Autorización:
¿Qué puedes hacer?
```

---

## Pregunta 2

¿Qué objeto representa al usuario actual?

Respuesta:

```python
request.user
```

---

## Pregunta 3

¿Cómo sabemos si un usuario inició sesión?

Respuesta:

```python
request.user.is_authenticated
```

---

## Pregunta 4

¿Cómo protegemos una vista para usuarios autenticados?

Respuesta:

```python
@login_required
```

---

## Pregunta 5

¿Qué diferencia existe entre `is_staff` e `is_superuser`?

Respuesta:

```text
is_staff:
permite acceso al Admin según los permisos asignados.

is_superuser:
tiene todos los permisos.
```

---

## Pregunta 6

¿Qué componente permite que Django coloque el usuario autenticado en `request.user`?

Respuesta:

```text
AuthenticationMiddleware
```

---

## Pregunta 7

¿Qué es un middleware?

Respuesta:

```text
Es una capa que procesa las peticiones y respuestas
antes o después de que intervenga la vista.
```

---

# 49. Crear un Middleware sencillo

Ahora podemos mostrar un ejemplo sencillo para comprender el concepto.

Creamos:

```text
middleware.py
```

Por ejemplo:

```python
class MiMiddleware:

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):

        print('Antes de la vista')

        response = self.get_response(request)

        print('Después de la vista')

        return response
```

---

# 50. Registrar el Middleware

En:

```text
settings.py
```

agregamos:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',

    'mi_app.middleware.MiMiddleware',
]
```

Debemos ajustar:

```text
mi_app
```

al nombre real de nuestra aplicación.

---

# 51. Flujo del Middleware

Podemos visualizarlo así:

```text
REQUEST
   ↓
MiMiddleware
   ↓
"Antes de la vista"
   ↓
VIEW
   ↓
"Después de la vista"
   ↓
RESPONSE
```

---

# 52. ¿Para qué sirven realmente los Middleware?

En proyectos reales podemos utilizar middleware para:

- Seguridad.
- Logs.
- Auditoría.
- Autenticación.
- Control de sesiones.
- Modificación de requests.
- Modificación de responses.
- Control de determinadas condiciones.

Por ejemplo:

```text
Request
   ↓
¿Quién es el usuario?
   ↓
¿Está autenticado?
   ↓
¿Cumple determinadas condiciones?
   ↓
View
```

---

# 53. Conceptos que deben quedar claros

Al finalizar la clase debemos tener claras estas relaciones:

```text
USER
 ↓
AUTHENTICATION
 ↓
request.user
 ↓
AUTHORIZATION
 ↓
PERMISSIONS
```

Y:

```text
REQUEST
 ↓
MIDDLEWARE
 ↓
VIEW
 ↓
RESPONSE
```

---

# 54. Resumen general

Django ya proporciona un sistema completo de autenticación.

Tenemos:

```text
User
```

para representar usuarios.

Tenemos:

```python
authenticate()
```

para comprobar credenciales.

Tenemos:

```python
login()
```

para iniciar sesión.

Tenemos:

```python
logout()
```

para cerrar sesión.

Tenemos:

```python
request.user
```

para obtener el usuario actual.

Tenemos:

```python
request.user.is_authenticated
```

para saber si está autenticado.

Tenemos:

```python
@login_required
```

para proteger vistas.

Tenemos:

```python
is_staff
```

para indicar acceso al Admin según permisos.

Tenemos:

```python
is_superuser
```

para indicar que el usuario tiene todos los permisos.

Tenemos:

```text
Groups
```

para agrupar usuarios.

Tenemos:

```text
Permissions
```

para controlar acciones.

Y tenemos:

```text
Middleware
```

para procesar requests y responses.

---

# 55. Escribir en la pizarra

Durante la clase podemos dejar esta estructura visible:

```text
                 DJANGO
                    │
                    ▼
                REQUEST
                    │
                    ▼
               MIDDLEWARE
                    │
                    ▼
                  URL
                    │
                    ▼
                  VIEW
                    │
                    ▼
                TEMPLATE
                    │
                    ▼
                RESPONSE
```

Después agregar:

```text
USUARIO
   │
   ▼
AUTENTICACIÓN
   │
   ▼
request.user
   │
   ▼
AUTORIZACIÓN
   │
   ▼
PERMISOS
```

Y finalmente:

```text
¿Quién eres?
      ↓
AUTENTICACIÓN

¿Qué puedes hacer?
      ↓
AUTORIZACIÓN
```

---

# 56. Distribución sugerida de los 120 minutos

| Tiempo | Tema |
|---|---|
| 0–10 min | Repaso de Clase 1 |
| 10–25 min | Usuarios y sistema de autenticación |
| 25–40 min | Admin y creación de usuarios |
| 40–55 min | Login, logout y sesiones |
| 55–70 min | `request.user` y `login_required` |
| 70–85 min | Autorización, `is_staff` e `is_superuser` |
| 85–100 min | Middleware |
| 100–115 min | Práctica integrada |
| 115–120 min | Preguntas y cierre |

---

# 57. Mensaje final para los estudiantes

La idea principal que debemos llevarnos de esta clase es:

```text
Django no solamente nos ayuda a crear páginas.

También nos proporciona mecanismos para saber:

¿QUIÉN ES EL USUARIO?
        ↓
AUTENTICACIÓN

¿QUÉ PUEDE HACER?
        ↓
AUTORIZACIÓN
```

Y todo esto se integra con:

```text
REQUEST
   ↓
MIDDLEWARE
   ↓
VIEW
   ↓
RESPONSE
```

Estos conceptos serán fundamentales cuando nuestro sistema de inventario comience a manejar diferentes tipos de usuarios y permisos.