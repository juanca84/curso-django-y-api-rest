# Clase 6 — Práctica Guiada
## Seguridad, Optimización y Buenas Prácticas en Aplicaciones Django

---

# Objetivo de la práctica

Al finalizar esta práctica, el estudiante tendrá una API de productos con:

```text
JWT
├── Login
├── Access Token
└── Refresh Token

Autorización
├── Anónimo → lectura
├── Usuario → crear/modificar
└── Administrador → eliminar

Optimización
├── select_related()
├── prefetch_related()
└── Paginación

Calidad
├── Tests
└── Logging

Seguridad
├── .env
├── DEBUG
├── ALLOWED_HOSTS
└── CORS
```

---

# 1. Punto de partida

Vamos a continuar con el proyecto Django desarrollado en las clases anteriores.

Supongamos la siguiente estructura:

```text
proyecto/
│
├── config/
│   ├── settings.py
│   ├── urls.py
│   └── ...
│
├── productos/
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   ├── urls.py
│   └── ...
│
└── manage.py
```

Antes de comenzar:

```bash
python manage.py runserver
```

Verificamos que la API desarrollada en la Clase 5 continúe funcionando.

---

# 2. Crear usuarios para las pruebas

Necesitamos tres escenarios:

```text
Usuario anónimo
        │
        └── sin login

Usuario normal
        │
        └── juan

Administrador
        │
        └── admin
```

## Crear un usuario normal

Ejecutamos:

```bash
python manage.py shell
```

Dentro del shell:

```python
from django.contrib.auth.models import User

User.objects.create_user(
    username="juan",
    password="123456"
)
```

## Crear un administrador

Ejecutamos:

```bash
python manage.py createsuperuser
```

Por ejemplo:

```text
Username: admin
Password: ********
```

Salir del shell:

```python
exit()
```

---

# 3. Instalar Simple JWT

Instalamos:

```bash
pip install djangorestframework-simplejwt
```

---

# 4. Configurar JWT

En:

```text
config/settings.py
```

configuramos Django REST Framework:

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": (
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ),
}
```

También configuramos la duración de los tokens:

```python
from datetime import timedelta

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=15),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=7),
    "ROTATE_REFRESH_TOKENS": True,
    "BLACKLIST_AFTER_ROTATION": True,
}
```

Tendremos:

```text
Access Token  → 15 minutos
Refresh Token → 7 días
```

> **Configuración de refresh tokens**

> ```python
> SIMPLE_JWT = {
>     "ACCESS_TOKEN_LIFETIME": timedelta(minutes=15),
>     "REFRESH_TOKEN_LIFETIME": timedelta(days=7),
>     "ROTATE_REFRESH_TOKENS": True,
>     "BLACKLIST_AFTER_ROTATION": True,
> }
> ```

> **¿Qué implican estos parámetros?**

> - **ROTATE_REFRESH_TOKENS**: Cuando es `True`, cada vez que se utiliza el refresh token para obtener un nuevo access token, el refresh token actual se invalida y se genera uno nuevo. Esto mejora la seguridad al evitar que un refresh token robado se use indefinidamente.

> - **BLACKLIST_AFTER_ROTATION**: Cuando es `True`, los refresh tokens rotados se agregan automáticamente a una "lista negra" (blacklist). Esto significa que incluso si alguien intercepte un refresh token viejo, este quedará invalidado después de la rotación.

> **¿Se requieren migraciones?**

> Sí, al incluir la app `rest_framework_simplejwt.token_blacklist` en `INSTALLED_APPS`, es necesario crear la tabla de blacklist:

> ```bash
> python manage.py migrate
> ```

> Esto creará la tabla `django_blacklist` en la base de datos, la cual almacena los refresh tokens revocados. La tabla incluye campos como:
> - `id`
> - `token`
> - `created_at`
> - `blacklisted_at`

> Si no se necesita la funcionalidad de blacklist, los settings `ROTATE_REFRESH_TOKENS` y `BLACKLIST_AFTER_ROTATION` seguirán funcionando para la rotación de tokens sin necesidad de esta tabla adicional.

> **Recomendación**: Mantener ambos en `True` en producción para una seguridad óptima, especialmente cuando los refresh tokens tienen una duración mayor que los access tokens.

---

# 5. Configurar endpoints JWT

En:

```text
config/urls.py
```

agregamos:

```python
from django.urls import path
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path(
        "api/token/",
        TokenObtainPairView.as_view(),
        name="token_obtain_pair",
    ),

    path(
        "api/token/refresh/",
        TokenRefreshView.as_view(),
        name="token_refresh",
    ),
]
```

Ahora tenemos:

```text
POST /api/token/
POST /api/token/refresh/
```

> Si ya existen otras rutas en `urlpatterns`, debemos mantenerlas y agregar estas nuevas rutas.

---

# 6. Obtener el Access Token

Utilizamos Postman, Insomnia o Thunder Client.

Realizamos:

```http
POST http://127.0.0.1:8000/api/token/
```

Body:

```json
{
    "username": "juan",
    "password": "123456"
}
```

La respuesta será similar a:

```json
{
    "refresh": "eyJ...",
    "access": "eyJ..."
}
```

Guardamos ambos tokens:

```text
ACCESS TOKEN
REFRESH TOKEN
```

---

# 7. Probar el Access Token

Probamos el endpoint:

```http
GET /api/productos/
```

Primero sin token.

Dependiendo de la configuración actual del ViewSet, el resultado puede ser público o requerir autenticación. Para esta práctica, posteriormente configuraremos el endpoint `GET` como público.

Para enviar el token utilizamos:

```http
Authorization: Bearer <ACCESS_TOKEN>
```

El flujo es:

```text
Cliente
   │
   │ Authorization: Bearer TOKEN
   ▼
Django
   │
   ├── valida token
   ├── identifica usuario
   └── verifica permisos
```

---

# 8. Probar el Refresh Token

Cuando el Access Token expire, realizamos:

```http
POST /api/token/refresh/
```

Body:

```json
{
    "refresh": "<REFRESH_TOKEN>"
}
```

La respuesta:

```json
{
    "access": "eyJ..."
}
```

Ahora utilizamos el nuevo Access Token:

```http
Authorization: Bearer <NUEVO_ACCESS_TOKEN>
```

---

# 9. Definir las reglas de permisos

Implementaremos las siguientes reglas:

## Usuario anónimo

```text
GET       permitido

POST      prohibido
PUT       prohibido
PATCH     prohibido
DELETE    prohibido
```

## Usuario autenticado

```text
GET       permitido
POST      permitido
PUT       permitido
PATCH     permitido
DELETE    prohibido
```

## Administrador

```text
GET       permitido
POST      permitido
PUT       permitido
PATCH     permitido
DELETE    permitido
```

---

# 10. Crear un permiso personalizado

Creamos:

```text
productos/permissions.py
```

Agregamos:

```python
from rest_framework.permissions import BasePermission


class ProductoPermission(BasePermission):

    def has_permission(self, request, view):

        # Lectura pública
        if request.method in ["GET", "HEAD", "OPTIONS"]:
            return True

        # DELETE solamente para administradores
        if request.method == "DELETE":
            return (
                request.user.is_authenticated
                and request.user.is_staff
            )

        # POST, PUT y PATCH requieren autenticación
        return request.user.is_authenticated
```

La lógica será:

```text
GET
  ↓
Público

POST / PUT / PATCH
  ↓
Usuario autenticado

DELETE
  ↓
Administrador
```

---

# 10.1. Método get_permissions en ViewSet

Además de configurar `permission_classes` como atributo de la clase, DRF permite sobrescribir el método `get_permissions()` para definir permisos diferentes según el método HTTP de la petición. Esto es útil cuando se requieren reglas más complejas por acción.

El método `get_permissions()` debe devolver una instancia de cada clase de permiso solicitada:

```python
from rest_framework.views import APIView
from rest_framework.permissions import IsAuthenticated, IsAdminUser

class ProductoViewSetGenerico(APIView):

    def get_permissions(self):
        if self.request.method == 'GET':
            return [IsAuthenticated()]
        if self.request.method == 'POST':
            return [IsAdminUser()]
        return [IsAdminUser()]
```

**Diferencia con `permission_classes`:**

| Enfoque | Cuándo usarlo |
|---|---|
| `permission_classes` | Reglas fijas que aplican a todos los métodos |
| `get_permissions()` | Reglas variables que cambian según el método HTTP |

## Ejemplo aplicado a nuestro ViewSet

Si en lugar de usar el permiso personalizado `ProductoPermission`, quisieras reglas por método, el ViewSet quedaría:

```python
from rest_framework.viewsets import ModelViewSet
from rest_framework.permissions import IsAuthenticated, IsAdminUser
from .models import Producto
from .serializers import ProductoSerializer

class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.select_related(
        "categoria"
    )

    serializer_class = ProductoSerializer

    def get_permissions(self):
        if self.request.method == 'GET':
            return [IsAuthenticated()]
        # POST, PUT, PATCH: requiere autenticación (cualquier usuario)
        if self.request.method in ['POST', 'PUT', 'PATCH']:
            return [IsAuthenticated()]
        # DELETE: solo administradores
        return [IsAdminUser()]
```

En este ejemplo:
- `GET` → cualquier usuario autenticado puede consultar
- `POST`, `PUT`, `PATCH` → requiere estar logueado (usuario normal o admin)
- `DELETE` → solo administradores (`is_staff`)

---

# 11. Aplicar el permiso al ViewSet

En:

```text
productos/views.py
```

importamos:

```python
from .permissions import ProductoPermission
```

Nuestro ViewSet puede quedar así:

```python
from rest_framework.viewsets import ModelViewSet

from .models import Producto
from .serializers import ProductoSerializer
from .permissions import ProductoPermission


class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.all()

    serializer_class = ProductoSerializer

    permission_classes = [ProductoPermission]
```

---

# 12. Probar los permisos manualmente

## Prueba 1 — GET sin autenticación

```http
GET /api/productos/
```

Resultado esperado:

```text
200 OK
```

## Prueba 2 — POST sin autenticación

```http
POST /api/productos/
```

Resultado esperado:

```text
401 Unauthorized
```

## Prueba 3 — POST con usuario normal

Enviar:

```http
Authorization: Bearer <TOKEN_JUAN>
```

Resultado esperado:

```text
201 Created
```

## Prueba 4 — DELETE con usuario normal

```http
DELETE /api/productos/1/
```

Resultado esperado:

```text
403 Forbidden
```

## Prueba 5 — DELETE con administrador

Utilizamos el token del usuario administrador:

```http
DELETE /api/productos/1/
```

Resultado esperado:

```text
204 No Content
```

---

# 13. Optimizar el ViewSet con `select_related()`

Supongamos que nuestro modelo `Producto` tiene una categoría:

```python
categoria = models.ForeignKey(
    Categoria,
    on_delete=models.CASCADE
)
```

Actualmente:

```python
queryset = Producto.objects.all()
```

Lo cambiamos por:

```python
queryset = Producto.objects.select_related(
    "categoria"
)
```

El ViewSet queda:

```python
class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.select_related(
        "categoria"
    )

    serializer_class = ProductoSerializer

    permission_classes = [ProductoPermission]
```

---

# 14. Comprender el problema N+1

Sin optimización:

```text
Consulta productos
        ↓
Producto 1 → consulta categoría
Producto 2 → consulta categoría
Producto 3 → consulta categoría
Producto 4 → consulta categoría
...
```

Con:

```python
select_related("categoria")
```

Django puede recuperar la información relacionada mediante una consulta optimizada.

```text
Producto
   │
   └── JOIN
        │
     Categoria
```

---

# 15. Implementar `prefetch_related()`

Si nuestro modelo tiene una relación ManyToMany, por ejemplo:

```python
proveedores = models.ManyToManyField(
    Proveedor
)
```

podemos utilizar:

```python
queryset = Producto.objects.select_related(
    "categoria"
).prefetch_related(
    "proveedores"
)
```

La regla general:

```text
select_related()
        ↓
ForeignKey / OneToOneField


prefetch_related()
        ↓
ManyToMany / relaciones reversas
```

---

# 16. Añadir paginación

En:

```text
config/settings.py
```

configuramos:

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": (
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ),

    "DEFAULT_PAGINATION_CLASS":
        "rest_framework.pagination.PageNumberPagination",

    "PAGE_SIZE": 20,
}
```

Probamos:

```http
GET /api/productos/
```

La respuesta tendrá una estructura similar a:

```json
{
    "count": 50,
    "next": "http://127.0.0.1:8000/api/productos/?page=2",
    "previous": null,
    "results": [
        ...
    ]
}
```

Probamos la segunda página:

```http
GET /api/productos/?page=2
```

---

# 17. Caché y optimización de consultas

La caché permite almacenar temporalmente resultados que se consultan frecuentemente, evitando repeticiones costosas a la base de datos.

## Caché en Django

Ejemplo utilizando el cache framework de Django:

```python
from django.core.cache import cache

productos = cache.get("productos")

if productos is None:
    productos = list(Producto.objects.all())
    cache.set("productos", productos, 300)
```

Aquí:

```text
300 segundos = 5 minutos
```

## Cuándo utilizar caché

Es útil para:

- Información consultada frecuentemente.
- Datos relativamente estables.
- Operaciones costosas.

No necesariamente es útil para:

- Información que cambia constantemente.
- Datos altamente personalizados.
- Operaciones simples que ya son rápidas.

## Caché y ViewSets

En los ViewSets, la caché puede aplicarse en el método `list`:

```python
class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.select_related(
        "categoria"
    )

    serializer_class = ProductoSerializer

    def list(self, request, *args, **kwargs):
        cache_key = "producto_list"
        productos = cache.get(cache_key)
        
        if productos is None:
            productos = self.get_queryset()
            cache.set(cache_key, productos, 300)
        
        serializer = self.get_serializer(productos, many=True)
        return Response(serializer.data)
```

---

# 18. Configurar variables de entorno

Instalamos:

```bash
pip install python-decouple
```

Creamos el archivo:

```text
.env
```

Contenido:

```env
SECRET_KEY=django-secret-key
DEBUG=True
```

En:

```text
config/settings.py
```

agregamos:

```python
from decouple import config

SECRET_KEY = config("SECRET_KEY")

DEBUG = config(
    "DEBUG",
    default=False,
    cast=bool
)
```

**Alternativa usando `import os`:**

> En lugar de (o junto con) `python-decouple`, Django permite leer variables de entorno directamente usando el módulo `os`:

```python
import os

SECRET_KEY = os.environ.get("SECRET_KEY")

DEBUG = os.environ.get("DEBUG", "False") == "True"
```

> **Diferencias entre enfoques:**

> | Enfoque | Ventajas | Desventajas |
> |---|---|---|
> | `python-decouple` | - Tipado automático (`cast=bool`)<br>- Más legible<br>- `default` values fáciles | - Dependencia extra |
> | `import os` | - Nativo de Python<br>- No requiere instalar paquetes | - Necesita conversión manual de tipos<br>- Menos legible para defaults |

> **Recomendación**: Usar `python-decouple` en proyectos Django ya que maneja el casting de tipos (booleanos, integers, etc.) de forma más robusta.

---

# 18. Proteger el archivo `.env`

En:

```text
.gitignore
```

agregamos:

```gitignore
.env
.venv/
__pycache__/
*.pyc
db.sqlite3
```

Comprobamos:

```bash
git status
```

El archivo `.env` no debería aparecer para ser enviado al repositorio.

---

# 19. Configurar `ALLOWED_HOSTS`

En desarrollo:

```python
ALLOWED_HOSTS = [
    "localhost",
    "127.0.0.1",
]
```

Para producción posteriormente:

```python
ALLOWED_HOSTS = [
    "api.midominio.com",
]
```

---

# 20. Configurar CORS

Instalamos:

```bash
pip install django-cors-headers
```

En:

```text
config/settings.py
```

agregamos:

```python
INSTALLED_APPS = [
    ...
    "corsheaders",
    "rest_framework_simplejwt.token_blacklist",
]
```

En `MIDDLEWARE`:

```python
MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",
    ...
]
```

Configuramos:

```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
]
```

La arquitectura será:

```text
Frontend
localhost:3000
       │
       │ HTTP
       ▼
API Django
localhost:8000
```

---

# 21. Implementar logging

En:

```text
productos/views.py
```

agregamos:

```python
import logging

logger = logging.getLogger(__name__)
```

Podemos registrar eventos:

```python
logger.info("Solicitud recibida para productos")
```

Y errores:

```python
logger.error("Error al procesar producto")
```

Nunca debemos registrar:

```text
password
access_token
refresh_token
SECRET_KEY
```

---

# 22. Configurar logging básico

En:

```text
config/settings.py
```

agregamos:

```python
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,

    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
        },
    },

    "loggers": {
        "django": {
            "handlers": ["console"],
            "level": "INFO",
        },
    },
}
```

Ejecutamos:

```bash
python manage.py runserver
```

y observamos los mensajes en la consola.

---

# 23. Crear tests de la API

En:

```text
productos/tests.py
```

creamos:

```python
from django.contrib.auth.models import User
from rest_framework.test import APITestCase
from rest_framework import status

from .models import Producto


class ProductoAPITest(APITestCase):

    def setUp(self):

        self.usuario = User.objects.create_user(
            username="juan",
            password="123456"
        )

        self.admin = User.objects.create_user(
            username="admin",
            password="123456",
            is_staff=True
        )

        self.producto = Producto.objects.create(
            nombre="Laptop",
            precio=5000
        )
```

> Si el modelo `Producto` requiere otros campos obligatorios, debemos agregarlos en la creación del objeto.

---

# 24. Test de GET público

```python
def test_listar_productos_publicamente(self):

    response = self.client.get(
        "/api/productos/"
    )

    self.assertEqual(
        response.status_code,
        status.HTTP_200_OK
    )
```

---

# 25. Test de POST sin autenticación

```python
def test_crear_producto_sin_autenticacion(self):

    data = {
        "nombre": "Mouse",
        "precio": 100
    }

    response = self.client.post(
        "/api/productos/",
        data,
        format="json"
    )

    self.assertEqual(
        response.status_code,
        status.HTTP_401_UNAUTHORIZED
    )
```

---

# 26. Test de POST autenticado

Autenticamos al usuario:

```python
self.client.force_authenticate(
    user=self.usuario
)
```

Después realizamos la prueba:

```python
def test_crear_producto_autenticado(self):

    self.client.force_authenticate(
        user=self.usuario
    )

    data = {
        "nombre": "Mouse",
        "precio": 100
    }

    response = self.client.post(
        "/api/productos/",
        data,
        format="json"
    )

    self.assertEqual(
        response.status_code,
        status.HTTP_201_CREATED
    )
```

---

# 27. Test de DELETE como usuario normal

```python
def test_eliminar_producto_usuario_normal(self):

    self.client.force_authenticate(
        user=self.usuario
    )

    response = self.client.delete(
        f"/api/productos/{self.producto.id}/"
    )

    self.assertEqual(
        response.status_code,
        status.HTTP_403_FORBIDDEN
    )
```

---

# 28. Test de DELETE como administrador

```python
def test_eliminar_producto_admin(self):

    self.client.force_authenticate(
        user=self.admin
    )

    response = self.client.delete(
        f"/api/productos/{self.producto.id}/"
    )

    self.assertEqual(
        response.status_code,
        status.HTTP_204_NO_CONTENT
    )
```

---

# 29. Ejecutar los tests

Ejecutamos:

```bash
python manage.py test
```

Esperamos una salida similar a:

```text
Ran 5 tests

OK
```

La matriz de pruebas es:

```text
GET público
       ↓
200

POST anónimo
       ↓
401

POST autenticado
       ↓
201

DELETE usuario normal
       ↓
403

DELETE administrador
       ↓
204
```

---

# 30. Ejecutar la comprobación de despliegue

Finalmente ejecutamos:

```bash
python manage.py check --deploy
```

Django mostrará advertencias relacionadas con configuraciones que deben revisarse antes de producción.

Revisamos especialmente:

```text
DEBUG
SECRET_KEY
ALLOWED_HOSTS
HTTPS
Cookies
Configuraciones de seguridad
```

---

# 31. Generar `requirements.txt`

Al finalizar la práctica:

```bash
pip freeze > requirements.txt
```

Entre las dependencias tendremos:

```text
Django
djangorestframework
djangorestframework-simplejwt
django-cors-headers
python-decouple
```

---

# 32. Documentación automática con Swagger

Para generar documentación interactiva de la API, podemos utilizar **DRF Spectacular** o **drf-yasg**. En esta práctica utilizaremos `drf-spectacular` por ser más moderno y mantenerse mejor con las nuevas versiones de DRF.

## Paso 1 — Instalar la dependencia

```bash
pip install drf-spectacular
```

## Paso 2 — Agregar a `INSTALLED_APPS`

En `config/settings.py`, añadimos:

```python
INSTALLED_APPS = [
    ...
    "rest_framework.schemas",  # opcional, para schemas nativos
    "drf_spectacular",
]
```

## Paso 3 — Configurar `SPECTACULAR_SETTINGS`

```python
SPECTACULAR_SETTINGS = {
    "TITLE": "API de Productos",
    "VERSION": "1.0.0",
    "DESCRIPTION": "Documentación de la API REST para gestión de productos",
    "TERMS_OF_SERVICE": "",
    "CONTACT": {
        "name": "Equipo de desarrollo",
        "email": "contacto@midominio.com",
    },
    "LICENSE": {
        "name": "BSD License",
    },
    "SERVERS": [
        {
            "url": "http://127.0.0.1:8000",
            "description": "Entorno de desarrollo"
        }
    ],
}
```

## Paso 4 — Configurar URLs de documentación

En `config/urls.py`, añadimos las rutas de Swagger:

```python
from django.contrib import admin
from django.urls import path, include
from drf_spectacular.views import (
    SpectacularAPIView,
    SpectacularSwaggerView,
    SpectacularRedocView,
)

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("productos.urls")),
    # Rutas de documentación:
    path(
        "api/schema/",
        SpectacularAPIView.as_view(),
        name="schema"
    ),
    path(
        "api/docs/",
        SpectacularSwaggerView.as_view(),
        name="swagger-ui"
    ),
    path(
        "api/redoc/",
        SpectacularRedocView.as_view(),
        name="redoc"
    ),
]
```

## Paso 5 — Verificar la documentación

Ejecutar el servidor:

```bash
python manage.py runserver
```

Visitar en el navegador:

- **Swagger UI**: `http://127.0.0.1:8000/api/docs/`
- **ReDoc**: `http://127.0.0.1:8000/api/redoc/`

Al acceder a estas URLs, se mostrarán todos los endpoints disponibles con sus métodos, parámetros y modelos de respuesta.

## Beneficios de usar Swagger:

- ✅ Documentación siempre actualizada con los cambios del código
- ✅ Interfaz interactiva para probar endpoints directamente
- ✅ Tipos de datos y validación mostrados visualmente
- ✅ Facilita el onboarding de nuevos desarrolladores
- ✅ Integración con herramientas de testing y generación de clientes API

---

# 33. Resultado final de la práctica

El proyecto debe quedar conceptualmente así:

```text
                    API PRODUCTOS

                         │
                         ▼

                       JWT
                   ┌──────┴──────┐
                   │             │
                Access        Refresh
                   │
                   ▼
              Autenticación
                   │
                   ▼
               Permisos
           ┌───────┼────────┐
           │       │        │
         GET      CRUD    DELETE
           │       │        │
        Público   User     Admin
                   │
                   ▼
               Optimización
           ┌───────┼────────┐
           │       │        │
         select  prefetch  Paginación
         related  related
                   │
                   ▼
                 Tests
                   │
                   ▼
                 Logs
                   │
                   ▼
              Producción
```

---

# 34. Checklist final

- [ ] Crear usuario normal.
- [ ] Crear administrador.
- [ ] Instalar Simple JWT.
- [ ] Configurar JWT.
- [ ] Configurar endpoint `/api/token/`.
- [ ] Configurar endpoint `/api/token/refresh/`.
- [ ] Obtener Access Token.
- [ ] Obtener Refresh Token.
- [ ] Probar endpoints autenticados.
- [ ] Crear permiso personalizado.
- [ ] Permitir GET público.
- [ ] Proteger POST.
- [ ] Proteger PUT.
- [ ] Proteger PATCH.
- [ ] Restringir DELETE al administrador.
- [ ] Aplicar `select_related()`.
- [ ] Aplicar `prefetch_related()` si existe una relación adecuada.
- [ ] Configurar paginación.
- [ ] Crear `.env`.
- [ ] Configurar `SECRET_KEY`.
- [ ] Configurar `DEBUG`.
- [ ] Configurar `ALLOWED_HOSTS`.
- [ ] Agregar `.env` al `.gitignore`.
- [ ] Configurar CORS.
- [ ] Implementar logging.
- [ ] Crear tests de autenticación y permisos.
- [ ] Ejecutar `python manage.py test`.
- [ ] Ejecutar `python manage.py check --deploy`.
- [ ] Generar `requirements.txt`.

---
