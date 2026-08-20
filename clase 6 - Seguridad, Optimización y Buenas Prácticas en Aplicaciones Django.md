# Clase 6 — Seguridad, Optimización y Buenas Prácticas en Aplicaciones Django

**Curso:** Desarrollo Backend con Django y Django REST Framework  
**Duración:** 2 horas  
**Nivel:** Intermedio  
**Prerequisito:** Clases 1–5  
**Proyecto:** API de inventario con Django REST Framework

---

# 1. Objetivos de la clase

Al finalizar esta clase, el estudiante podrá:

- Comprender autenticación y autorización en Django.
- Diferenciar autenticación de autorización.
- Comprender el funcionamiento de JWT.
- Implementar `Access Token` y `Refresh Token`.
- Proteger endpoints mediante permisos.
- Crear permisos personalizados.
- Comprender las principales configuraciones de seguridad de Django.
- Comprender la importancia de `DEBUG=False`.
- Configurar `ALLOWED_HOSTS`.
- Proteger secretos mediante variables de entorno.
- Comprender CORS y CSRF.
- Aplicar HTTPS en producción.
- Identificar problemas de rendimiento.
- Comprender el problema N+1.
- Utilizar `select_related()` y `prefetch_related()`.
- Aplicar paginación.
- Comprender el uso de caché.
- Crear pruebas automatizadas.
- Utilizar logging.
- Preparar una aplicación Django para producción.

---

# 2. Introducción: seguridad en una aplicación web

Una aplicación backend no solamente debe funcionar.

También debe:

- Proteger información.
- Verificar quién realiza una petición.
- Determinar qué puede hacer cada usuario.
- Evitar accesos no autorizados.
- Proteger credenciales.
- Evitar exposición de información sensible.
- Manejar correctamente los errores.
- Ser eficiente.
- Poder ser monitoreada y mantenida.

Podemos dividir la seguridad en varias áreas:

```text
                    SEGURIDAD
                        │
        ┌───────────────┼────────────────┐
        │               │                │
 Autenticación     Autorización      Configuración
        │               │                │
   ¿Quién eres?    ¿Qué puedes       ¿Cómo está
                     hacer?           configurado?
```

---

# 3. Autenticación vs autorización

## 3.1 Autenticación

La autenticación responde:

> ¿Quién eres?

Ejemplo:

```text
Usuario:
juan

Contraseña:
********
```

El servidor verifica las credenciales.

Si son correctas:

```text
Usuario autenticado
```

---

## 3.2 Autorización

La autorización responde:

> ¿Qué puedes hacer?

Por ejemplo:

```text
Juan
 ├── puede consultar productos
 ├── puede crear productos
 └── no puede eliminar productos
```

Otro usuario:

```text
Administrador
 ├── puede consultar productos
 ├── puede crear productos
 ├── puede modificar productos
 └── puede eliminar productos
```

---

## 3.3 Diferencia

```text
AUTENTICACIÓN
     ↓
¿Quién eres?
     ↓
Identidad

AUTORIZACIÓN
     ↓
¿Qué puedes hacer?
     ↓
Permisos
```

Una aplicación segura necesita ambas.

---

# 4. Autenticación en Django

Django proporciona un sistema de autenticación integrado.

Incluye:

- Usuarios.
- Contraseñas.
- Sesiones.
- Grupos.
- Permisos.
- `is_staff`.
- `is_superuser`.

En aplicaciones web tradicionales, Django suele trabajar con:

```text
Cookie + Session
```

En APIs REST es común utilizar:

```text
Token
```

Uno de los mecanismos más utilizados es:

```text
JWT
```

---

# 5. ¿Qué es JWT?

JWT significa:

> JSON Web Token

Es un estándar utilizado para transportar información sobre una identidad de manera compacta.

Un JWT normalmente tiene tres partes:

```text
HEADER.PAYLOAD.SIGNATURE
```

Ejemplo conceptual:

```text
eyJhbGciOiJIUzI1NiJ9.
eyJ1c2VyX2lkIjoxLCJleHAiOjE3...
.
firma
```

---

# 6. Partes de un JWT

## 6.1 Header

Indica información sobre el token.

Ejemplo:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

## 6.2 Payload

Contiene los claims.

Ejemplo:

```json
{
  "user_id": 10,
  "username": "juan",
  "exp": 1780000000
}
```

Puede contener información como:

- Identificación del usuario.
- Fecha de expiración.
- Fecha de emisión.
- Identificadores adicionales.

### Importante

No debemos guardar información sensible en el payload.

---

## 6.3 Signature

La firma permite verificar que el token no fue alterado.

Conceptualmente:

```text
header + payload + secret
                ↓
             firma
```

Si alguien modifica el payload:

```text
user_id = 10
```

por:

```text
user_id = 999
```

la firma ya no coincidirá.

---

# 7. JWT no significa que el payload esté cifrado

Este punto es muy importante.

JWT normalmente está:

```text
Codificado
```

pero no necesariamente:

```text
Cifrado
```

Por eso no debemos colocar:

```json
{
  "password": "123456"
}
```

ni:

```json
{
  "credit_card": "..."
}
```

El payload puede ser leído por quien tenga el token.

La firma protege la **integridad**, pero no convierte automáticamente el contenido en secreto.

---

# 8. Access Token y Refresh Token

En una arquitectura JWT normalmente encontramos:

```text
Access Token
+
Refresh Token
```

---

## 8.1 Access Token

Es el token utilizado para acceder a los endpoints protegidos.

Ejemplo:

```http
Authorization: Bearer <access_token>
```

Normalmente tiene una duración relativamente corta.

Ejemplo:

```text
Access Token
     │
     └── válido durante 5–30 minutos
```

---

## 8.2 Refresh Token

Se utiliza para obtener un nuevo access token sin solicitar nuevamente usuario y contraseña.

Ejemplo:

```text
Refresh Token
     │
     └── duración mayor
```

Puede durar:

```text
días
```

dependiendo de la política de seguridad.

---

# 9. ¿Por qué utilizar Refresh Token?

Supongamos:

```text
Access Token = 15 minutos
```

El usuario inicia sesión:

```text
Login
 ↓
Access Token
 ↓
15 minutos
 ↓
Expira
```

Sin refresh token tendríamos que pedir nuevamente:

```text
username
password
```

Con refresh token:

```text
Access Token expira
        ↓
Cliente utiliza Refresh Token
        ↓
Servidor valida Refresh Token
        ↓
Nuevo Access Token
        ↓
Usuario continúa trabajando
```

Esto permite tener access tokens de corta duración sin obligar al usuario a iniciar sesión constantemente.

---

# 10. Ejemplo práctico del flujo JWT

```text
CLIENTE
   │
   │ POST /api/token/
   │ username + password
   ↓
DJANGO
   │
   │ verifica credenciales
   ↓
JWT
   │
   ├── access
   └── refresh
   ↓
CLIENTE
```

Después:

```text
CLIENTE
   │
   │ GET /api/productos/
   │ Authorization: Bearer ACCESS_TOKEN
   ↓
DJANGO
   │
   ├── valida token
   ├── identifica usuario
   └── verifica permisos
   ↓
RESPUESTA
```

---

# 11. Instalación de Simple JWT

Para implementar JWT con Django REST Framework:

```bash
pip install djangorestframework-simplejwt
```

En `settings.py`:

```python
INSTALLED_APPS = [
    ...
    "rest_framework",
]
```

Configuración:

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": (
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ),
}
```

---

# 12. Configurar URLs para JWT

En `urls.py`:

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
        name="token_obtain_pair"
    ),

    path(
        "api/token/refresh/",
        TokenRefreshView.as_view(),
        name="token_refresh"
    ),
]
```

---

# 13. Obtener un token

Realizamos:

```http
POST /api/token/
```

Body:

```json
{
    "username": "juan",
    "password": "123456"
}
```

Respuesta:

```json
{
    "refresh": "eyJ...",
    "access": "eyJ..."
}
```

Tenemos:

```text
refresh
access
```

---

# 14. Utilizar el Access Token

Para acceder a un endpoint protegido:

```http
GET /api/productos/
Authorization: Bearer eyJ...
```

El servidor:

```text
Recibe token
    ↓
Valida firma
    ↓
Verifica expiración
    ↓
Identifica usuario
    ↓
Verifica permisos
    ↓
Permite o rechaza
```

---

# 15. Refresh del Access Token

Cuando el access token expira:

```http
POST /api/token/refresh/
```

Body:

```json
{
    "refresh": "eyJ..."
}
```

Respuesta:

```json
{
    "access": "eyJ..."
}
```

No es necesario enviar nuevamente:

```text
username
password
```

---

# 16. Configuración del tiempo de vida

En `settings.py`:

```python
from datetime import timedelta

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=15),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=7),
}
```

Ejemplo:

```text
Access Token  → 15 minutos
Refresh Token → 7 días
```

La duración debe definirse según las necesidades de seguridad de la aplicación.

---

# 17. Autorización con DRF

Autenticar al usuario no significa permitirle hacer todo.

DRF permite utilizar permisos.

Ejemplo:

```python
from rest_framework.permissions import IsAuthenticated
```

En una vista:

```python
class ProductoListCreateView(generics.ListCreateAPIView):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    permission_classes = [IsAuthenticated]
```

Ahora solamente usuarios autenticados pueden acceder.

---

# 18. Permisos comunes

DRF proporciona permisos como:

```python
AllowAny
IsAuthenticated
IsAdminUser
IsAuthenticatedOrReadOnly
```

---

## 18.1 AllowAny

Permite acceso a cualquier usuario.

```python
permission_classes = [AllowAny]
```

---

## 18.2 IsAuthenticated

Requiere autenticación.

```python
permission_classes = [IsAuthenticated]
```

---

## 18.3 IsAdminUser

Permite acceso a usuarios administrativos.

```python
permission_classes = [IsAdminUser]
```

---

## 18.4 IsAuthenticatedOrReadOnly

Permite:

```text
GET → público
```

pero requiere autenticación para:

```text
POST
PUT
PATCH
DELETE
```

Ejemplo:

```python
permission_classes = [IsAuthenticatedOrReadOnly]
```

---

# 19. Permisos personalizados

Podemos crear reglas propias.

Ejemplo:

```python
from rest_framework.permissions import BasePermission

class EsAdministrador(BasePermission):

    def has_permission(self, request, view):
        return request.user.is_staff
```

Después:

```python
permission_classes = [EsAdministrador]
```

---

# 20. Seguridad: contraseñas

Nunca debemos guardar contraseñas directamente.

Incorrecto:

```python
usuario.password = "123456"
```

Django utiliza hashing de contraseñas.

Ejemplo:

```python
user.set_password("123456")
user.save()
```

Para crear un usuario:

```python
User.objects.create_user(
    username="juan",
    password="123456"
)
```

Django almacena un hash, no la contraseña original.

---

# 21. SECRET_KEY

Django utiliza:

```python
SECRET_KEY
```

para diferentes mecanismos de seguridad.

Nunca debemos publicar la clave en GitHub.

Incorrecto:

```python
SECRET_KEY = "mi-clave-super-secreta"
```

Mejor:

```python
import os

SECRET_KEY = os.environ["SECRET_KEY"]
```

---

# 22. Variables de entorno

Las variables de entorno permiten separar configuración y código.

Ejemplo:

```text
SECRET_KEY
DATABASE_URL
DEBUG
ALLOWED_HOSTS
```

Podemos utilizar:

```bash
pip install python-decouple
```

Ejemplo:

```python
from decouple import config

SECRET_KEY = config("SECRET_KEY")
DEBUG = config("DEBUG", default=False, cast=bool)
```

---

# 23. DEBUG

Durante desarrollo:

```python
DEBUG = True
```

En producción:

```python
DEBUG = False
```

Nunca debemos dejar:

```python
DEBUG = True
```

en producción.

Una aplicación con errores puede exponer información como:

- Rutas internas.
- Código.
- Configuración.
- Consultas.
- Variables.
- Stack traces.
- Información del servidor.

---

# 24. ALLOWED_HOSTS

En producción:

```python
ALLOWED_HOSTS = [
    "api.midominio.com",
]
```

Durante desarrollo:

```python
ALLOWED_HOSTS = [
    "localhost",
    "127.0.0.1",
]
```

Evitar utilizar:

```python
ALLOWED_HOSTS = ["*"]
```

sin entender las implicaciones.

---

# 25. CSRF

CSRF significa:

> Cross-Site Request Forgery

Es un ataque en el cual un sitio malicioso intenta realizar acciones utilizando la sesión de un usuario autenticado.

Django incorpora protección CSRF.

Para formularios Django:

```html
<form method="POST">
    {% csrf_token %}
    ...
</form>
```

En APIs REST, el mecanismo depende del tipo de autenticación utilizado.

Con JWT enviado mediante:

```http
Authorization: Bearer <token>
```

el flujo es diferente al de autenticación basada en cookies/sesiones.

---

# 26. CORS

CORS significa:

> Cross-Origin Resource Sharing

Es relevante cuando tenemos:

```text
Frontend
   │
   │ HTTP
   ↓
API Django
```

Por ejemplo:

```text
Frontend:
http://localhost:3000

Backend:
http://localhost:8000
```

Son diferentes orígenes.

Instalación:

```bash
pip install django-cors-headers
```

Configuración:

```python
INSTALLED_APPS = [
    ...
    "corsheaders",
]
```

Middleware:

```python
MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",
    ...
]
```

Ejemplo:

```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
]
```

No debemos permitir cualquier origen sin analizar el riesgo.

---

# 27. HTTPS

En producción debemos utilizar:

```text
HTTPS
```

y no:

```text
HTTP
```

Especialmente cuando se transmiten:

- Credenciales.
- Tokens.
- Información personal.
- Datos sensibles.

Arquitectura típica:

```text
Cliente
   │
 HTTPS
   ↓
Nginx / Load Balancer
   │
   ↓
Gunicorn
   │
   ↓
Django
```

---

# 28. Cookies y seguridad

Si utilizamos cookies para autenticación, debemos considerar:

```python
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
```

También pueden utilizarse políticas `SameSite`.

La configuración exacta depende de la arquitectura de la aplicación.

---

# 29. SQL Injection

Django ORM ayuda a evitar SQL Injection cuando utilizamos correctamente los QuerySets.

Correcto:

```python
Producto.objects.filter(nombre=nombre)
```

Debemos evitar construir SQL concatenando strings.

Incorrecto:

```python
query = f"SELECT * FROM productos WHERE nombre = '{nombre}'"
```

La regla es:

> Utilizar el ORM y consultas parametrizadas siempre que sea posible.

---

# 30. Validación de datos

Nunca debemos confiar en los datos enviados por el cliente.

El cliente puede enviar:

```json
{
    "precio": -100
}
```

o:

```json
{
    "stock": "abc"
}
```

Los serializers de DRF permiten validar.

Ejemplo:

```python
class ProductoSerializer(serializers.ModelSerializer):

    class Meta:
        model = Producto
        fields = "__all__"

    def validate_precio(self, value):
        if value < 0:
            raise serializers.ValidationError(
                "El precio no puede ser negativo."
            )

        return value
```

---

# 31. Principio de mínimo privilegio

Un usuario debe tener únicamente los permisos que necesita.

Ejemplo:

```text
Empleado
 ├── consultar productos
 └── crear productos

Administrador
 ├── consultar
 ├── crear
 ├── modificar
 └── eliminar
```

No debemos hacer que todos sean:

```text
superuser
```

---

# 32. Optimización

Una aplicación puede funcionar correctamente y aun así ser lenta.

Ejemplo:

```text
100 productos
```

puede funcionar perfectamente.

Pero:

```text
1.000.000 productos
```

requiere pensar en:

- Consultas.
- Índices.
- Paginación.
- Caché.
- Serialización.
- Cantidad de datos.
- Relaciones.
- Arquitectura.

---

# 33. El problema N+1

Supongamos:

```python
productos = Producto.objects.all()
```

y cada producto tiene:

```text
categoria
```

Después:

```python
for producto in productos:
    print(producto.categoria.nombre)
```

Podríamos terminar realizando:

```text
1 consulta → productos

+ 100 consultas → categorías
```

Total:

```text
101 consultas
```

Esto se conoce como:

> Problema N+1

---

# 34. select_related()

Para relaciones:

```text
ForeignKey
OneToOneField
```

podemos utilizar:

```python
Producto.objects.select_related("categoria")
```

Ejemplo:

```python
productos = Producto.objects.select_related(
    "categoria"
)
```

Diferencia conceptual:

```text
Sin optimización:

1 consulta productos
+
N consultas categorías


Con select_related():

consulta optimizada
        ↓
JOIN
        ↓
productos + categoría
```

---

# 35. prefetch_related()

Para relaciones como:

```text
ManyToMany
Reverse ForeignKey
```

podemos utilizar:

```python
Producto.objects.prefetch_related("proveedores")
```

Ejemplo:

```python
productos = Producto.objects.prefetch_related(
    "proveedores"
)
```

Diferencia general:

```text
select_related
    ↓
JOIN
    ↓
ForeignKey / OneToOne

prefetch_related
    ↓
consultas separadas optimizadas
    ↓
ManyToMany / relaciones reversas
```

---

# 36. only()

Podemos limitar los campos recuperados:

```python
Producto.objects.only(
    "id",
    "nombre",
    "precio"
)
```

Esto puede ser útil cuando tenemos modelos grandes y solamente necesitamos algunos campos.

---

# 37. values()

Podemos obtener diccionarios en lugar de instancias completas:

```python
Producto.objects.values(
    "id",
    "nombre",
    "precio"
)
```

Resultado conceptual:

```python
[
    {
        "id": 1,
        "nombre": "Laptop",
        "precio": 5000
    }
]
```

---

# 38. exists()

Si solamente necesitamos saber si existe un registro:

```python
if Producto.objects.filter(
    nombre="Laptop"
).exists():
    ...
```

Esto es mejor que cargar todos los registros solamente para verificar existencia.

---

# 39. count()

Si necesitamos conocer la cantidad:

```python
Producto.objects.count()
```

No debemos cargar todos los objetos solamente para contarlos.

---

# 40. Índices de base de datos

Si buscamos frecuentemente:

```python
Producto.objects.filter(codigo="ABC123")
```

podemos considerar un índice.

En el modelo:

```python
codigo = models.CharField(
    max_length=50,
    db_index=True
)
```

O:

```python
class Meta:
    indexes = [
        models.Index(fields=["codigo"]),
    ]
```

Los índices mejoran determinadas búsquedas, pero tienen un costo en almacenamiento y operaciones de escritura.

---

# 41. Paginación

No debemos devolver miles de registros en una sola respuesta.

Mala práctica:

```http
GET /api/productos/
```

devolviendo:

```text
500.000 registros
```

Mejor:

```text
Página 1 → 20 registros
Página 2 → 20 registros
Página 3 → 20 registros
```

DRF permite configurar paginación.

Ejemplo:

```python
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS":
        "rest_framework.pagination.PageNumberPagination",

    "PAGE_SIZE": 20,
}
```

---

# 42. Filtrado y búsqueda

En APIs grandes debemos permitir consultar solamente los datos necesarios.

Ejemplo:

```http
GET /api/productos/?categoria=1
```

o:

```http
GET /api/productos/?search=laptop
```

Esto reduce:

- Datos enviados.
- Procesamiento.
- Memoria.
- Tiempo de respuesta.

---

# 43. Caché

La caché permite almacenar temporalmente resultados que se consultan frecuentemente.

Ejemplo:

```text
Primera petición
      ↓
Base de datos
      ↓
Resultado
      ↓
Cache

Segunda petición
      ↓
Cache
      ↓
Resultado
```

Ejemplo:

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

---

# 44. No utilizar caché para todo

La caché debe utilizarse estratégicamente.

Es útil para:

- Información consultada frecuentemente.
- Datos relativamente estables.
- Operaciones costosas.

No necesariamente es útil para:

- Información que cambia constantemente.
- Datos altamente personalizados.
- Operaciones simples que ya son rápidas.

---

# 45. Optimización del Serializer

Los serializers también pueden convertirse en un punto costoso.

Debemos evitar:

```text
Serializer
    ↓
consulta adicional
    ↓
consulta adicional
    ↓
consulta adicional
```

Ejemplo:

```python
def get_categoria_nombre(self, obj):
    return obj.categoria.nombre
```

Si no utilizamos:

```python
select_related("categoria")
```

podemos generar consultas adicionales.

---

# 46. Optimización de ViewSets

Ejemplo:

```python
class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.select_related(
        "categoria"
    )

    serializer_class = ProductoSerializer
```

La optimización debe hacerse considerando cómo se utilizará el QuerySet.

---

# 47. Testing

Las pruebas automatizadas permiten verificar que la aplicación funciona correctamente.

Podemos probar:

```text
Modelos
Views
Serializers
Endpoints
Autenticación
Permisos
```

---

# 48. ¿Por qué hacer tests?

Sin tests:

```text
Cambiar código
     ↓
¿Se rompió algo?
     ↓
No sabemos
```

Con tests:

```text
Cambiar código
     ↓
Ejecutar tests
     ↓
PASS / FAIL
```

---

# 49. Test básico en Django

Django incluye herramientas de testing.

Ejemplo:

```python
from django.test import TestCase

class ProductoTestCase(TestCase):

    def test_producto(self):
        producto = Producto.objects.create(
            nombre="Laptop",
            precio=5000
        )

        self.assertEqual(
            producto.nombre,
            "Laptop"
        )
```

Ejecutamos:

```bash
python manage.py test
```

---

# 50. Testing de API con DRF

DRF proporciona:

```python
from rest_framework.test import APITestCase
```

Ejemplo:

```python
class ProductoAPITest(APITestCase):

    def test_listar_productos(self):
        response = self.client.get(
            "/api/productos/"
        )

        self.assertEqual(
            response.status_code,
            200
        )
```

---

# 51. Probar autenticación

Podemos probar que un endpoint protegido rechace usuarios no autenticados.

Ejemplo:

```python
def test_endpoint_protegido(self):
    response = self.client.get(
        "/api/productos/"
    )

    self.assertEqual(
        response.status_code,
        401
    )
```

Después podemos probar un usuario autenticado.

---

# 52. Probar permisos

Debemos verificar:

```text
Usuario autenticado
        ↓
¿Tiene permiso?
        ↓
200 / 403
```

Ejemplo:

```text
Empleado
    ↓
DELETE /api/productos/1/
    ↓
403 Forbidden
```

Mientras:

```text
Administrador
    ↓
DELETE /api/productos/1/
    ↓
204 No Content
```

---

# 53. Logging

Los logs permiten saber qué ocurre en la aplicación.

Podemos registrar:

```text
INFO
WARNING
ERROR
CRITICAL
```

---

# 54. Ejemplo de logging

```python
import logging

logger = logging.getLogger(__name__)

logger.info("Producto creado")
```

Error:

```python
logger.error(
    "Error al crear producto"
)
```

No debemos registrar información sensible.

Evitar:

```python
logger.info(password)
```

o:

```python
logger.info(access_token)
```

---

# 55. Configuración básica de logging

Ejemplo:

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

En producción podemos enviar logs a sistemas especializados.

---

# 56. Manejo de errores

No debemos exponer errores internos al cliente.

Mala respuesta:

```json
{
    "error": "Traceback ... SECRET_KEY ... database password ..."
}
```

Mejor:

```json
{
    "detail": "Ocurrió un error interno."
}
```

El detalle técnico debe quedar en los logs.

---

# 57. Códigos HTTP importantes

Debemos conocer:

```text
200 OK
201 Created
204 No Content

400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity

500 Internal Server Error
```

Especial atención:

```text
401
```

Generalmente significa:

> El usuario no está autenticado o las credenciales no son válidas.

Mientras:

```text
403
```

significa:

> El usuario está autenticado, pero no tiene permisos suficientes.

---

# 58. Buenas prácticas de estructura

Una estructura posible:

```text
project/
│
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   │
│   ├── urls.py
│   └── wsgi.py
│
├── productos/
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   ├── permissions.py
│   ├── urls.py
│   ├── tests.py
│   └── admin.py
│
├── manage.py
├── requirements.txt
└── .env
```

La estructura exacta depende del tamaño del proyecto.

---

# 59. requirements.txt

Debemos registrar dependencias.

Ejemplo:

```text
Django
djangorestframework
djangorestframework-simplejwt
psycopg
django-cors-headers
```

Generar:

```bash
pip freeze > requirements.txt
```

Instalar:

```bash
pip install -r requirements.txt
```

---

# 60. .gitignore

Nunca debemos subir:

```text
.env
.venv/
__pycache__/
*.pyc
db.sqlite3
```

Ejemplo:

```gitignore
.env
.venv/
__pycache__/
*.pyc
db.sqlite3
```

---

# 61. Variables de entorno en producción

Ejemplo:

```text
SECRET_KEY=...
DEBUG=False
DATABASE_URL=...
ALLOWED_HOSTS=api.midominio.com
```

El código fuente no debe contener estos secretos.

---

# 62. Deployment de Django

En producción no debemos utilizar:

```bash
python manage.py runserver
```

`runserver` está diseñado para desarrollo.

Una arquitectura común es:

```text
                    INTERNET
                        │
                        ▼
                     HTTPS
                        │
                        ▼
                     NGINX
                        │
                        ▼
                   GUNICORN
                        │
                        ▼
                     DJANGO
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
         PostgreSQL             Redis
```

---

# 63. Gunicorn

Gunicorn es un servidor WSGI utilizado para ejecutar aplicaciones Python/Django.

Instalación:

```bash
pip install gunicorn
```

Ejemplo:

```bash
gunicorn config.wsgi:application
```

El nombre `config` depende del nombre del proyecto.

---

# 64. Nginx

Nginx puede actuar como:

- Reverse proxy.
- Servidor de archivos estáticos.
- Terminador HTTPS.
- Balanceador de carga.

Flujo:

```text
Cliente
   ↓
HTTPS
   ↓
Nginx
   ↓
Gunicorn
   ↓
Django
```

---

# 65. Archivos estáticos

En producción:

```python
STATIC_URL = "static/"
STATIC_ROOT = BASE_DIR / "staticfiles"
```

Después:

```bash
python manage.py collectstatic
```

Esto recopila los archivos estáticos para servirlos correctamente.

---

# 66. Migraciones en producción

Antes de ejecutar la aplicación:

```bash
python manage.py migrate
```

En desarrollo:

```bash
python manage.py makemigrations
```

Las migraciones deben mantenerse versionadas.

---

# 67. Crear superusuario

Para administración:

```bash
python manage.py createsuperuser
```

No debemos compartir las credenciales.

---

# 68. Comando de comprobación de despliegue

Django proporciona:

```bash
python manage.py check --deploy
```

Este comando ayuda a detectar configuraciones de seguridad que deben revisarse antes de producción.

---

# 69. Checklist de producción

Antes de desplegar:

```text
DEBUG = False
        ↓
SECRET_KEY protegida
        ↓
ALLOWED_HOSTS configurado
        ↓
HTTPS habilitado
        ↓
Base de datos de producción
        ↓
Migraciones ejecutadas
        ↓
collectstatic
        ↓
Gunicorn / servidor WSGI
        ↓
Nginx / reverse proxy
        ↓
Logs
        ↓
Backups
        ↓
Tests
```

---

# 70. Buenas prácticas de seguridad para JWT

Recomendaciones:

- Access tokens de duración corta.
- Refresh tokens con duración controlada.
- No guardar información sensible en el payload.
- Utilizar HTTPS.
- Proteger secretos.
- No registrar tokens en logs.
- Definir correctamente permisos.
- Revocar o rotar refresh tokens cuando la arquitectura lo requiera.
- Proteger el almacenamiento del token en el cliente.
- Validar correctamente expiración y firma.

---

# 71. Flujo completo de seguridad de nuestra API

```text
                 CLIENTE
                    │
                    │ Login
                    ▼
             /api/token/
                    │
                    ▼
             AUTENTICACIÓN
                    │
                    ▼
            Access + Refresh
                    │
                    ▼
        ┌──────────────────────┐
        │ Request con Access   │
        │ Token                │
        └──────────────────────┘
                    │
                    ▼
             ¿Token válido?
               │       │
              NO      SÍ
               │       │
              401      ▼
                  ¿Tiene permiso?
                    │       │
                   NO      SÍ
                    │       │
                   403     ▼
                       Ejecutar vista
                           │
                           ▼
                         200
```

---

# 72. Ejemplo práctico de la clase

## Objetivo

Proteger nuestra API de productos.

Tenemos:

```text
/api/productos/
```

Queremos:

```text
GET
    público

POST
    usuario autenticado

PUT
    usuario autenticado

PATCH
    usuario autenticado

DELETE
    administrador
```

---

# 73. Implementación del permiso

Podemos crear:

```python
from rest_framework.permissions import BasePermission

class EsAdministradorParaEliminar(BasePermission):

    def has_permission(self, request, view):
        if request.method == "DELETE":
            return (
                request.user.is_authenticated
                and request.user.is_staff
            )

        return True
```

Dependiendo del diseño, también podemos separar permisos por acción.

---

# 74. Ejemplo con ViewSet

```python
from rest_framework.viewsets import ModelViewSet
from rest_framework.permissions import IsAuthenticated

class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.select_related(
        "categoria"
    )

    serializer_class = ProductoSerializer
    permission_classes = [IsAuthenticated]
```

Después podemos refinar permisos según:

```text
list
retrieve
create
update
partial_update
destroy
```

---

# 75. Optimización del ViewSet

En lugar de:

```python
queryset = Producto.objects.all()
```

podemos utilizar:

```python
queryset = Producto.objects.select_related(
    "categoria"
)
```

Si además tenemos proveedores:

```python
queryset = Producto.objects.select_related(
    "categoria"
).prefetch_related(
    "proveedores"
)
```

Esto reduce consultas innecesarias.

---

# 76. Paginación del ViewSet

Configuración global:

```python
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS":
        "rest_framework.pagination.PageNumberPagination",

    "PAGE_SIZE": 20,
}
```

Respuesta:

```json
{
    "count": 100,
    "next": "...",
    "previous": null,
    "results": [
        ...
    ]
}
```

---

# 77. Testing final de la API

Debemos probar:

```text
1. Usuario no autenticado
2. Usuario autenticado
3. Usuario sin permisos
4. Administrador
5. Crear producto
6. Actualizar producto
7. Eliminar producto
8. Token expirado
9. Refresh token
10. Validación de datos
```

---

# 78. Práctica guiada

## Paso 1 — Instalar JWT

```bash
pip install djangorestframework-simplejwt
```

## Paso 2 — Configurar autenticación

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": (
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ),
}
```

## Paso 3 — Configurar endpoints

```python
path(
    "api/token/",
    TokenObtainPairView.as_view()
)

path(
    "api/token/refresh/",
    TokenRefreshView.as_view()
)
```

## Paso 4 — Obtener token

```http
POST /api/token/
```

```json
{
    "username": "juan",
    "password": "123456"
}
```

## Paso 5 — Proteger productos

```python
permission_classes = [IsAuthenticated]
```

## Paso 6 — Probar Access Token

```http
Authorization: Bearer <TOKEN>
```

## Paso 7 — Obtener un nuevo Access Token

```http
POST /api/token/refresh/
```

Body:

```json
{
    "refresh": "<REFRESH_TOKEN>"
}
```

## Paso 8 — Crear permisos personalizados

Separar:

```text
lectura
escritura
eliminación
```

## Paso 9 — Optimizar QuerySet

Aplicar:

```python
select_related()
prefetch_related()
```

## Paso 10 — Añadir paginación

```python
PAGE_SIZE = 20
```

## Paso 11 — Añadir tests

Probar:

```text
401
403
200
201
204
```

## Paso 12 — Añadir logging

Registrar eventos importantes sin exponer información sensible.

## Paso 13 — Revisar seguridad

Verificar:

```text
DEBUG
SECRET_KEY
ALLOWED_HOSTS
CORS
HTTPS
```

---

# 79. Ejercicio para estudiantes

Implementar las siguientes reglas.

## Usuario anónimo

Puede:

```text
GET /api/productos/
GET /api/productos/{id}/
```

No puede:

```text
POST
PUT
PATCH
DELETE
```

## Usuario autenticado

Puede:

```text
GET
POST
PUT
PATCH
```

No puede:

```text
DELETE
```

## Administrador

Puede:

```text
GET
POST
PUT
PATCH
DELETE
```

---

# 80. Segundo ejercicio: optimización

Partir de:

```python
productos = Producto.objects.all()
```

Detectar si el serializer genera consultas adicionales.

Después implementar:

```python
select_related()
```

y:

```python
prefetch_related()
```

Comparar el número de consultas.

---

# 81. Tercer ejercicio: testing

Crear tests para comprobar:

```text
GET público
POST sin autenticación
POST autenticado
DELETE usuario normal
DELETE administrador
```

Esperamos:

```text
GET público          → 200
POST anónimo         → 401
POST autenticado     → 201
DELETE usuario       → 403
DELETE administrador → 204
```

---

# 82. Conceptos que el estudiante debe recordar

## Autenticación

```text
¿Quién eres?
```

## Autorización

```text
¿Qué puedes hacer?
```

## Access Token

```text
Token de corta duración
```

## Refresh Token

```text
Permite obtener un nuevo Access Token
```

## JWT

```text
Header + Payload + Signature
```

## select_related

```text
ForeignKey / OneToOne
```

## prefetch_related

```text
ManyToMany / relaciones reversas
```

## Paginación

```text
No devolver cantidades enormes de datos
```

## Caché

```text
Evitar repetir operaciones costosas
```

## Testing

```text
Verificar automáticamente que el sistema funciona
```

## Logging

```text
Conocer qué ocurre dentro de la aplicación
```

---

# 83. Errores comunes

## Error 1 — DEBUG en producción

```python
DEBUG = True
```

Nunca debe utilizarse en producción.

---

## Error 2 — Subir `.env`

No debemos subir:

```text
.env
```

a GitHub.

---

## Error 3 — Exponer SECRET_KEY

Nunca debemos publicar:

```python
SECRET_KEY = "..."
```

---

## Error 4 — Guardar contraseñas directamente

Nunca debemos guardar passwords en texto plano.

---

## Error 5 — No utilizar HTTPS

Los tokens y credenciales deben viajar mediante conexiones seguras.

---

## Error 6 — No proteger endpoints

No todos los endpoints deben ser públicos.

---

## Error 7 — Confundir 401 y 403

```text
401 → no autenticado
403 → autenticado pero sin permisos
```

---

## Error 8 — No utilizar paginación

Evitar devolver miles o millones de registros.

---

## Error 9 — Ignorar N+1

Revisar las consultas generadas por el ORM.

---

## Error 10 — Registrar información sensible

Nunca registrar:

```text
password
access_token
refresh_token
SECRET_KEY
```

---

## Error 11 — Utilizar `runserver` como servidor de producción

`runserver` es para desarrollo.

---

# 84. Arquitectura final

Al finalizar el curso podemos tener:

```text
                         CLIENTE
                            │
                            │ HTTPS
                            ▼
                         NGINX
                            │
                            ▼
                       GUNICORN
                            │
                            ▼
                         DJANGO
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
          JWT/Auth      DRF/API       Validación
              │             │             │
              └─────────────┼─────────────┘
                            │
                            ▼
                        PostgreSQL
                            │
                            ▼
                          Redis
                         (caché)
```

---

# 85. Checklist final de la clase

- [ ] Entender autenticación.
- [ ] Entender autorización.
- [ ] Comprender JWT.
- [ ] Diferenciar Access Token y Refresh Token.
- [ ] Configurar Simple JWT.
- [ ] Proteger endpoints con permisos.
- [ ] Crear permisos personalizados.
- [ ] Comprender `DEBUG=False`.
- [ ] Configurar `ALLOWED_HOSTS`.
- [ ] Proteger `SECRET_KEY`.
- [ ] Utilizar variables de entorno.
- [ ] Comprender CORS.
- [ ] Comprender CSRF.
- [ ] Utilizar HTTPS en producción.
- [ ] Evitar SQL Injection.
- [ ] Validar datos.
- [ ] Comprender N+1.
- [ ] Utilizar `select_related()`.
- [ ] Utilizar `prefetch_related()`.
- [ ] Utilizar paginación.
- [ ] Comprender caché.
- [ ] Crear tests.
- [ ] Probar autenticación y permisos.
- [ ] Utilizar logging.
- [ ] Comprender el despliegue con Gunicorn/Nginx.
- [ ] Ejecutar `python manage.py check --deploy`.

---

# 86. Comandos importantes de la clase

## Instalar JWT

```bash
pip install djangorestframework-simplejwt
```

## Instalar CORS

```bash
pip install django-cors-headers
```

## Ejecutar servidor de desarrollo

```bash
python manage.py runserver
```

## Crear migraciones

```bash
python manage.py makemigrations
```

## Ejecutar migraciones

```bash
python manage.py migrate
```

## Crear superusuario

```bash
python manage.py createsuperuser
```

## Ejecutar tests

```bash
python manage.py test
```

## Recopilar archivos estáticos

```bash
python manage.py collectstatic
```

## Verificar configuración de despliegue

```bash
python manage.py check --deploy
```

## Generar requirements

```bash
pip freeze > requirements.txt
```

## Ejecutar Gunicorn

```bash
gunicorn config.wsgi:application
```

---

# 87. Resultado esperado del curso

Al terminar las 6 clases, el estudiante debería ser capaz de construir una aplicación Django con:

```text
Django
│
├── MVT
├── Templates
├── Models
├── ORM
├── Forms
├── CRUD
│
├── Django REST Framework
│   ├── API
│   ├── Serializers
│   ├── APIView
│   ├── Generic Views
│   └── ModelViewSet
│
├── Seguridad
│   ├── Autenticación
│   ├── Autorización
│   ├── JWT
│   ├── Access Token
│   └── Refresh Token
│
├── Optimización
│   ├── QuerySets
│   ├── select_related
│   ├── prefetch_related
│   ├── Índices
│   ├── Paginación
│   └── Caché
│
└── Calidad
    ├── Testing
    ├── Logging
    ├── Variables de entorno
    └── Deployment
```

---

# 88. Cierre de la clase

La idea principal de esta última clase es que una aplicación Django profesional no termina cuando:

```text
"el endpoint funciona"
```

Debemos preguntarnos:

```text
¿Está protegido?

¿Quién puede acceder?

¿Qué puede hacer cada usuario?

¿Los secretos están protegidos?

¿Está preparado para producción?

¿Es eficiente?

¿Tenemos tests?

¿Tenemos logs?

¿Podemos detectar errores?
```

Una aplicación profesional debe buscar el equilibrio entre:

```text
SEGURIDAD
     +
RENDIMIENTO
     +
MANTENIBILIDAD
     +
CALIDAD
     +
OBSERVABILIDAD
```

Y este conjunto de prácticas permite pasar de:

```text
Proyecto que funciona
```

a:

```text
Aplicación lista para producción
```

---

# 89. Conclusión general del curso

Durante las seis clases hemos recorrido el proceso completo de construcción de una aplicación backend con Django:

```text
Clase 1
Django + MVT
       ↓
Clase 2
Autenticación y autorización
       ↓
Clase 3
Middleware + Decorators + ORM
       ↓
Clase 4
Views + Forms + CRUD
       ↓
Clase 5
Django REST Framework
       ↓
Clase 6
Seguridad + Optimización + Testing + Deployment
```

El objetivo final no es solamente conocer comandos de Django.

El estudiante debe comprender cómo construir un backend completo:

```text
                    DJANGO BACKEND
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
    Modelos              API              Seguridad
       │                  │                  │
      ORM                DRF                JWT
       │                  │                  │
       └──────────────────┼──────────────────┘
                          │
                     Optimización
                          │
                     Testing/Logs
                          │
                       Producción
```

---

# Fin de la Clase 6

**Desarrollo Backend con Django y Django REST Framework**