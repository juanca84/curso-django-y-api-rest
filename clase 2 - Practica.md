# Clase 2 — Usuarios, Autenticación, Autorización y Middleware

## Práctica completa

### Objetivo

Al finalizar la práctica podremos:

- Crear usuarios y superusuarios.
- Acceder al sistema de administración.
- Crear un login y logout.
- Utilizar `request.user`.
- Proteger vistas con `@login_required`.
- Diferenciar autenticación y autorización.
- Trabajar con permisos de Django.
- Utilizar el permiso `auth.add_user`.
- Crear grupos y asignar permisos.
- Crear un Middleware propio.
- Registrar y ejecutar nuestro Middleware.
- Comprender el flujo Request → Middleware → View → Response.

---

# 1. Estructura inicial

Partimos de nuestro proyecto Django:

```text
mi-proyecto/
├── manage.py
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── usuarios/
    ├── migrations/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    └── views.py
```

Si todavía no tenemos la aplicación:

```bash
python manage.py startapp usuarios
```

---

# 2. Registrar la aplicación

Abrimos:

```text
config/settings.py
```

Buscamos:

```python
INSTALLED_APPS = [
```

Y agregamos:

```python
INSTALLED_APPS = [
    # Aplicaciones de Django
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Aplicaciones propias
    'usuarios',
]
```

---

# 3. Ejecutar las migraciones

Antes de trabajar con usuarios, verificamos que la base de datos esté preparada:

```bash
python manage.py migrate
```

Podemos comprobar que Django creó las tablas relacionadas con autenticación.

---

# 4. Crear un superusuario

Ejecutamos:

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
Username: admin
Email: admin@example.com
Password: ********
```

---

# 5. Ejecutar el servidor

```bash
python manage.py runserver
```

Entramos a:

```text
http://127.0.0.1:8000/admin/
```

Iniciamos sesión con el superusuario.

---

# 6. Conocer el modelo User

Django proporciona un modelo de usuario:

```python
from django.contrib.auth.models import User
```

Podemos utilizar el shell:

```bash
python manage.py shell
```

Consultar todos los usuarios:

```python
from django.contrib.auth.models import User

User.objects.all()
```

Podemos obtener un usuario:

```python
usuario = User.objects.get(username='admin')
```

Consultar su nombre:

```python
usuario.username
```

Consultar su correo:

```python
usuario.email
```

---

# 7. Crear un usuario normal desde el shell

Utilizamos:

```python
usuario = User.objects.create_user(
    username='juan',
    password='123456'
)
```

Comprobamos:

```python
usuario.username
```

```text
'juan'
```

### Importante

No debemos crear usuarios utilizando:

```python
User.objects.create(
    username='juan',
    password='123456'
)
```

Para crear usuarios debemos utilizar:

```python
User.objects.create_user(...)
```

porque Django se encarga de realizar correctamente el hash de la contraseña.

---

# 8. Comprobar usuarios

Podemos ejecutar:

```python
User.objects.all()
```

También:

```python
for usuario in User.objects.all():
    print(usuario.username)
```

---

# 9. Crear las URLs de la aplicación

Creamos:

```text
usuarios/urls.py
```

Contenido:

```python
from django.urls import path
from . import views

urlpatterns = [
]
```

---

# 10. Incluir las URLs en el proyecto

Abrimos:

```text
config/urls.py
```

Configuramos:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('usuarios/', include('usuarios.urls')),
]
```

---

# 11. Crear nuestra primera vista

Abrimos:

```text
usuarios/views.py
```

Agregamos:

```python
from django.http import HttpResponse


def perfil(request):
    return HttpResponse(
        f'Hola {request.user.username}'
    )
```

---

# 12. Crear la URL de perfil

En:

```text
usuarios/urls.py
```

agregamos:

```python
from django.urls import path
from . import views

urlpatterns = [
    path(
        'perfil/',
        views.perfil,
        name='perfil'
    ),
]
```

Ahora podemos acceder a:

```text
http://127.0.0.1:8000/usuarios/perfil/
```

---

# 13. Comprender request.user

En nuestra vista:

```python
def perfil(request):
    return HttpResponse(
        f'Hola {request.user.username}'
    )
```

Django proporciona:

```python
request.user
```

Este objeto representa al usuario asociado a la petición.

Podemos comprobar:

```python
request.user.username
```

También:

```python
request.user.email
```

Y:

```python
request.user.is_authenticated
```

---

# 14. Proteger nuestra vista

Importamos:

```python
from django.contrib.auth.decorators import login_required
```

Y utilizamos:

```python
@login_required
def perfil(request):
    return HttpResponse(
        f'Hola {request.user.username}'
    )
```

Ahora:

```text
Usuario autenticado
        ↓
    Puede acceder
        ↓
       perfil
```

Mientras que:

```text
Usuario no autenticado
        ↓
   No puede acceder
        ↓
       login
```

---

# 15. Configurar LOGIN_URL

En:

```text
config/settings.py
```

agregamos:

```python
LOGIN_URL = '/usuarios/login/'
```

Esto le indica a Django dónde se encuentra nuestra página de login.

---

# 16. Crear el login

En:

```text
usuarios/urls.py
```

importamos:

```python
from django.contrib.auth.views import LoginView
```

Y agregamos:

```python
path(
    'login/',
    LoginView.as_view(
        template_name='registration/login.html'
    ),
    name='login'
),
```

Nuestro archivo completo queda:

```python
from django.urls import path
from django.contrib.auth.views import LoginView
from . import views

urlpatterns = [
    path(
        'login/',
        LoginView.as_view(
            template_name='registration/login.html'
        ),
        name='login'
    ),

    path(
        'perfil/',
        views.perfil,
        name='perfil'
    ),
]
```

---

# 17. Crear el template del login

Creamos:

```text
usuarios/
└── templates/
    └── registration/
        └── login.html
```

Contenido:

```html
<h1>Iniciar sesión</h1>

<form method="post">
    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit">
        Iniciar sesión
    </button>
</form>
```

---

# 18. Probar el login

Entramos:

```text
http://127.0.0.1:8000/usuarios/login/
```

Ingresamos:

```text
Usuario: juan
Contraseña: 123456
```

Después accedemos:

```text
http://127.0.0.1:8000/usuarios/perfil/
```

Deberíamos obtener:

```text
Hola juan
```

---

# 19. Crear el logout

En:

```text
usuarios/urls.py
```

importamos:

```python
from django.contrib.auth.views import (
    LoginView,
    LogoutView,
)
```

Agregamos:

```python
path(
    'logout/',
    LogoutView.as_view(),
    name='logout'
),
```

Las URLs quedan:

```python
urlpatterns = [
    path(
        'login/',
        LoginView.as_view(
            template_name='registration/login.html'
        ),
        name='login'
    ),

    path(
        'logout/',
        LogoutView.as_view(),
        name='logout'
    ),

    path(
        'perfil/',
        views.perfil,
        name='perfil'
    ),
]
```

---

# 20. Autenticación

Autenticación responde:

```text
¿Quién eres?
```

Podemos comprobarlo mediante:

```python
request.user.is_authenticated
```

Ejemplo:

```python
if request.user.is_authenticated:
    print('Usuario autenticado')
else:
    print('Usuario no autenticado')
```

---

# 21. Autorización

Autorización responde:

```text
¿Qué puedes hacer?
```

Por ejemplo:

```python
request.user.has_perm('auth.add_user')
```

Estamos preguntando:

```text
¿Este usuario tiene permiso para agregar usuarios?
```

---

# 22. Los permisos de User

El modelo `User` pertenece a la aplicación:

```text
auth
```

Django crea permisos automáticamente.

Los permisos principales son:

```text
auth.add_user
auth.change_user
auth.delete_user
auth.view_user
```

Significado:

```text
auth.add_user
    ↓
Crear usuarios

auth.change_user
    ↓
Modificar usuarios

auth.delete_user
    ↓
Eliminar usuarios

auth.view_user
    ↓
Consultar usuarios
```

---

# 23. Comprobar permisos desde el shell

Entramos:

```bash
python manage.py shell
```

Importamos:

```python
from django.contrib.auth.models import User
```

Obtenemos nuestro usuario:

```python
usuario = User.objects.get(
    username='juan'
)
```

Comprobamos:

```python
usuario.has_perm('auth.add_user')
```

Inicialmente probablemente obtendremos:

```text
False
```

---

# 24. Comprobar el permiso desde una vista

En:

```text
usuarios/views.py
```

podemos crear:

```python
from django.http import HttpResponse


def crear_usuario(request):

    if request.user.has_perm('auth.add_user'):
        return HttpResponse(
            'Puedes crear usuarios'
        )

    return HttpResponse(
        'No tienes permiso para crear usuarios'
    )
```

---

# 25. Crear la URL

En:

```text
usuarios/urls.py
```

agregamos:

```python
path(
    'crear-usuario/',
    views.crear_usuario,
    name='crear_usuario'
),
```

Ahora:

```text
/usuarios/crear-usuario/
```

---

# 26. Utilizar permission_required

Django también proporciona un decorador para permisos.

Importamos:

```python
from django.contrib.auth.decorators import permission_required
```

Podemos proteger directamente la vista:

```python
@permission_required('auth.add_user')
def crear_usuario(request):
    return HttpResponse(
        'Puedes crear usuarios'
    )
```

Ahora Django comprueba automáticamente:

```text
¿Está autorizado?
       ↓
auth.add_user
       ↓
   ┌───┴───┐
   Sí      No
   ↓        ↓
Accede    403
```

---

# 27. Diferencia entre is_staff y permisos

No debemos confundir:

```python
request.user.is_staff
```

con:

```python
request.user.has_perm('auth.add_user')
```

`is_staff` indica que el usuario puede acceder al sitio de administración de Django.

Mientras que:

```python
request.user.has_perm('auth.add_user')
```

comprueba un permiso específico.

También tenemos:

```python
request.user.is_superuser
```

Un superusuario tiene todos los permisos.

---

# 28. Crear un grupo

Django permite agrupar usuarios.

Desde el shell:

```bash
python manage.py shell
```

Importamos:

```python
from django.contrib.auth.models import (
    Group,
    Permission,
)
```

Buscamos el permiso:

```python
permiso = Permission.objects.get(
    codename='add_user'
)
```

También podemos comprobar:

```python
permiso.content_type.app_label
```

Debería indicar:

```text
auth
```

---

# 29. Crear un grupo

```python
grupo = Group.objects.create(
    name='Gestores de usuarios'
)
```

---

# 30. Asignar el permiso al grupo

```python
grupo.permissions.add(permiso)
```

Ahora el grupo posee:

```text
auth.add_user
```

---

# 31. Asignar el grupo al usuario

Obtenemos el usuario:

```python
usuario = User.objects.get(
    username='juan'
)
```

Agregamos el grupo:

```python
usuario.groups.add(grupo)
```

---

# 32. Comprobar nuevamente el permiso

```python
usuario.has_perm('auth.add_user')
```

Ahora debería devolver:

```text
True
```

El flujo es:

```text
Usuario
   ↓
Grupo
   ↓
Gestores de usuarios
   ↓
auth.add_user
```

---

# 33. Crear nuestro propio Middleware

Ahora pasamos a Middleware.

Creamos:

```text
usuarios/middleware.py
```

Contenido:

```python
class RequestLogMiddleware:

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):

        print(
            f'Petición: '
            f'{request.method} '
            f'{request.path}'
        )

        response = self.get_response(request)

        print(
            f'Respuesta: '
            f'{response.status_code}'
        )

        return response
```

---

# 34. Comprender el Middleware

Tenemos:

```python
def __init__(self, get_response):
```

Django ejecuta esto cuando crea el Middleware.

`get_response` representa el siguiente elemento del ciclo de procesamiento.

Después:

```python
def __call__(self, request):
```

Se ejecuta para cada petición.

Aquí podemos inspeccionar:

```python
request.method
request.path
request.user
request.headers
request.GET
request.POST
```

Finalmente:

```python
response = self.get_response(request)
```

permite que la petición continúe.

Y:

```python
return response
```

devuelve la respuesta.

---

# 35. Registrar nuestro Middleware

Abrimos:

```text
config/settings.py
```

Buscamos:

```python
MIDDLEWARE = [
```

Agregamos:

```python
'usuarios.middleware.RequestLogMiddleware',
```

Por ejemplo:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',

    'django.contrib.sessions.middleware.SessionMiddleware',

    'django.middleware.common.CommonMiddleware',

    'django.middleware.csrf.CsrfViewMiddleware',

    'django.contrib.auth.middleware.AuthenticationMiddleware',

    'django.contrib.messages.middleware.MessageMiddleware',

    'django.middleware.clickjacking.XFrameOptionsMiddleware',

    'usuarios.middleware.RequestLogMiddleware',
]
```

---

# 36. Ejecutar el Middleware

Reiniciamos el servidor:

```bash
python manage.py runserver
```

Entramos a:

```text
http://127.0.0.1:8000/usuarios/perfil/
```

En la terminal podremos observar:

```text
Petición: GET /usuarios/perfil/
Respuesta: 200
```

Si accedemos a:

```text
/usuarios/login/
```

veremos algo parecido a:

```text
Petición: GET /usuarios/login/
Respuesta: 200
```

---

# 37. Middleware y request.user

Podemos crear un Middleware relacionado con autenticación.

En:

```text
usuarios/middleware.py
```

podemos utilizar:

```python
class UserInfoMiddleware:

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):

        if request.user.is_authenticated:
            print(
                f'Usuario autenticado: '
                f'{request.user.username}'
            )
        else:
            print(
                'Usuario no autenticado'
            )

        response = self.get_response(request)

        return response
```

---

# 38. Importante: orden de Middleware

Nuestro Middleware utiliza:

```python
request.user
```

Por eso:

```python
'django.contrib.auth.middleware.AuthenticationMiddleware',
```

debe ejecutarse antes que nuestro Middleware.

Conceptualmente:

```text
Request
   ↓
SessionMiddleware
   ↓
AuthenticationMiddleware
   ↓
request.user
   ↓
Nuestro Middleware
   ↓
View
```

---

# 39. Middleware que agrega información al Request

Podemos crear otro ejemplo:

```python
class RequestInfoMiddleware:

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):

        request.mensaje = (
            'Información agregada por Middleware'
        )

        response = self.get_response(request)

        return response
```

El Middleware agrega:

```python
request.mensaje
```

---

# 40. Utilizar la información desde una View

En:

```text
usuarios/views.py
```

podemos hacer:

```python
def informacion(request):

    return HttpResponse(
        request.mensaje
    )
```

Creamos la URL:

```python
path(
    'informacion/',
    views.informacion,
    name='informacion'
),
```

Ahora tenemos:

```text
Request
   ↓
Middleware
   ↓
request.mensaje = "..."
   ↓
View
   ↓
request.mensaje
   ↓
Response
```

---

# 41. Flujo completo de Django

La idea fundamental que debemos llevarnos de esta clase es:

```text
                    Django
                      │
                      ▼
                  HTTP Request
                      │
                      ▼
                ┌─────────────┐
                │ Middleware  │
                └─────────────┘
                      │
                      ▼
                     URL
                      │
                      ▼
                    View
                      │
                      ▼
                  Response
                      │
                      ▼
                ┌─────────────┐
                │ Middleware  │
                └─────────────┘
                      │
                      ▼
                 HTTP Response
                      │
                      ▼
                  Navegador
```

---

# 42. Resumen de autenticación y autorización

## Autenticación

Pregunta:

```text
¿Quién eres?
```

Ejemplo:

```python
request.user.is_authenticated
```

---

## Autorización

Pregunta:

```text
¿Qué puedes hacer?
```

Ejemplo:

```python
request.user.has_perm(
    'auth.add_user'
)
```

---

## Roles / Staff

```python
request.user.is_staff
```

Indica si el usuario puede acceder al sitio de administración.

---

## Superusuario

```python
request.user.is_superuser
```

Un superusuario tiene todos los permisos.

---

# 43. Ejercicio final

Al finalizar la práctica, los estudiantes deben implementar:

### Parte 1 — Usuarios

- Crear un superusuario.
- Crear un usuario normal.
- Iniciar sesión con el usuario normal.
- Mostrar su nombre utilizando `request.user.username`.

### Parte 2 — Autenticación

Crear:

```text
/usuarios/login/
/usuarios/logout/
/usuarios/perfil/
```

La vista `/perfil/` debe estar protegida con:

```python
@login_required
```

### Parte 3 — Autorización

Crear:

```text
/usuarios/crear-usuario/
```

Y protegerla utilizando:

```python
@permission_required('auth.add_user')
```

### Parte 4 — Permisos

Crear un grupo:

```text
Gestores de usuarios
```

Asignarle:

```text
auth.add_user
```

Agregar un usuario al grupo y comprobar:

```python
usuario.has_perm('auth.add_user')
```

### Parte 5 — Middleware

Crear:

```text
usuarios/middleware.py
```

Implementar un Middleware que muestre en la terminal:

```text
Usuario: juan
Método: GET
URL: /usuarios/perfil/
```

---

# 44. Conceptos que debemos poder explicar

Al terminar la clase, el estudiante debería poder responder:

```text
¿Qué es autenticación?
¿Qué es autorización?
¿Qué es request.user?
¿Qué hace @login_required?
¿Qué hace @permission_required?
¿Qué significa auth.add_user?
¿Qué diferencia existe entre is_staff e is_superuser?
¿Qué es un grupo?
¿Qué es un permiso?
¿Qué es un Middleware?
¿Qué hace __init__() en un Middleware?
¿Qué hace __call__()?
¿Qué hace get_response()?
¿Por qué importa el orden de los Middleware?
```

---

# 45. Flujo conceptual final

```text
                    USUARIO
                       │
                       ▼
                  LOGIN
                       │
                       ▼
              AUTENTICACIÓN
                       │
                       ▼
                 request.user
                       │
                       ▼
              ¿ESTÁ AUTENTICADO?
                 │           │
                NO           SÍ
                 │           │
                 ▼           ▼
               LOGIN      AUTORIZACIÓN
                              │
                              ▼
                    ¿Tiene permiso?
                              │
                       ┌──────┴──────┐
                      NO             SÍ
                       │              │
                       ▼              ▼
                    DENEGADO         VIEW
                                      │
                                      ▼
                                  RESPONSE
```

**Idea central de la Clase 2:**

> Django proporciona todo un sistema integrado para saber quién es el usuario, mantener su sesión, determinar qué puede hacer y ejecutar lógica antes y después de las vistas mediante Middleware.