# Clase 5 — Práctica paso a paso
## Desarrollo de APIs REST con Django REST Framework

**Proyecto:** Sistema de Inventario  
**Duración:** 2 horas  
**Tecnología:** Django + Django REST Framework + PostgreSQL

---

# 1. Objetivo de la práctica

Al finalizar la práctica, construiremos una API REST para administrar los productos del sistema de inventario.

La API permitirá:

- Consultar todos los productos.
- Crear productos.
- Consultar un producto específico.
- Actualizar un producto.
- Eliminar un producto.
- Trabajar con JSON.
- Utilizar serializers.
- Utilizar `APIView`.
- Finalmente simplificar la API utilizando `ModelViewSet` y `Router`.

La evolución será:

```text
Modelo Django
     ↓
Serializer
     ↓
APIView
     ↓
URLs
     ↓
API REST
     ↓
ModelViewSet
     ↓
Router
     ↓
CRUD completo
```

---

# 2. Estado inicial del proyecto

Antes de comenzar debemos tener nuestro proyecto Django funcionando.

La estructura aproximada será:

```text
mi_proyecto/
│
├── manage.py
│
├── config/
│   ├── settings.py
│   ├── urls.py
│   └── ...
│
└── productos/
    ├── models.py
    ├── views.py
    ├── urls.py
    ├── serializers.py
    └── ...
```

La aplicación que utilizaremos será:

```text
productos
```

---

# 3. Verificar el modelo Producto

Nuestro modelo `Producto` será el recurso que expondremos mediante la API.

En:

```text
productos/models.py
```

debemos tener algo similar a:

```python
from django.db import models


class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField(blank=True)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    activo = models.BooleanField(default=True)

    def __str__(self):
        return self.nombre
```

## ¿Qué representa este modelo?

Cada registro de `Producto` será un recurso de nuestra API.

Por ejemplo:

```text
Producto
├── id
├── nombre
├── descripcion
├── precio
├── stock
└── activo
```

Un registro podría convertirse en JSON:

```json
{
    "id": 1,
    "nombre": "Laptop Lenovo",
    "descripcion": "Laptop para oficina",
    "precio": "4500.00",
    "stock": 10,
    "activo": true
}
```

---

# 4. Verificar migraciones

Ejecutamos:

```bash
python manage.py makemigrations
```

Después:

```bash
python manage.py migrate
```

Podemos verificar que existen productos utilizando el shell:

```bash
python manage.py shell
```

Dentro del shell:

```python
from productos.models import Producto

Producto.objects.all()
```

Si todavía no tenemos datos:

```python
Producto.objects.create(
    nombre="Laptop Lenovo",
    descripcion="Laptop para oficina",
    precio=4500,
    stock=10
)
```

Otro ejemplo:

```python
Producto.objects.create(
    nombre="Mouse Logitech",
    descripcion="Mouse inalámbrico",
    precio=150,
    stock=25
)
```

Salir:

```python
exit()
```

---

# 5. Instalar Django REST Framework

Instalamos DRF:

```bash
pip install djangorestframework
```

Podemos verificar:

```bash
pip show djangorestframework
```

---

# 6. Agregar DRF al proyecto

Abrimos:

```text
config/settings.py
```

Buscamos:

```python
INSTALLED_APPS = [
    ...
]
```

Agregamos:

```python
INSTALLED_APPS = [
    ...
    "rest_framework",
    "productos",
]
```

Por ejemplo:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",

    "rest_framework",
    "productos",
]
```

---

# 7. Primera explicación: ¿qué hará DRF?

Hasta ahora Django trabaja principalmente con:

```text
Request
   ↓
View
   ↓
Template
   ↓
HTML
```

Con una API REST tendremos:

```text
Request
   ↓
View DRF
   ↓
Serializer
   ↓
Modelo
   ↓
JSON
```

Por ejemplo:

```text
GET /api/productos/
```

La respuesta podría ser:

```json
[
    {
        "id": 1,
        "nombre": "Laptop Lenovo",
        "descripcion": "Laptop para oficina",
        "precio": "4500.00",
        "stock": 10,
        "activo": true
    },
    {
        "id": 2,
        "nombre": "Mouse Logitech",
        "descripcion": "Mouse inalámbrico",
        "precio": "150.00",
        "stock": 25,
        "activo": true
    }
]
```

---

# 8. Crear el Serializer

Ahora debemos crear:

```text
productos/serializers.py
```

Contenido:

```python
from rest_framework import serializers

from .models import Producto


class ProductoSerializer(serializers.ModelSerializer):

    class Meta:
        model = Producto
        fields = "__all__"
```

---

# 9. ¿Qué hace un Serializer?

El serializer tiene dos responsabilidades fundamentales.

## Django → JSON

Convierte un objeto Django:

```python
producto = Producto.objects.get(id=1)
```

en datos que pueden enviarse mediante una API:

```json
{
    "id": 1,
    "nombre": "Laptop Lenovo",
    "precio": "4500.00",
    "stock": 10
}
```

## JSON → Django

También puede recibir información:

```json
{
    "nombre": "Teclado Logitech",
    "descripcion": "Teclado inalámbrico",
    "precio": "250.00",
    "stock": 15,
    "activo": true
}
```

y convertirla en datos que Django puede validar y guardar.

Por eso podemos visualizarlo así:

```text
Modelo Django
      ↕
Serializer
      ↕
JSON
```

---

# 10. Crear nuestra primera API con APIView

Ahora vamos a construir la API manualmente.

Esto es importante porque queremos que los estudiantes entiendan qué está ocurriendo antes de utilizar abstracciones como `ModelViewSet`.

Abrimos:

```text
productos/views.py
```

Agregamos:

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

from .models import Producto
from .serializers import ProductoSerializer
```

---

# 11. Crear ProductoListCreateAPIView

En `views.py`:

```python
class ProductoListCreateAPIView(APIView):

    def get(self, request):
        productos = Producto.objects.all()

        serializer = ProductoSerializer(
            productos,
            many=True
        )

        return Response(serializer.data)

    def post(self, request):
        serializer = ProductoSerializer(
            data=request.data
        )

        if serializer.is_valid():
            serializer.save()

            return Response(
                serializer.data,
                status=status.HTTP_201_CREATED
            )

        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )
```

---

# 12. Analizar el método GET

Tenemos:

```python
def get(self, request):
```

Esto responde a:

```http
GET /api/productos/
```

Primero obtenemos los productos:

```python
productos = Producto.objects.all()
```

Después utilizamos el serializer:

```python
serializer = ProductoSerializer(
    productos,
    many=True
)
```

`many=True` significa que estamos serializando múltiples objetos.

Finalmente:

```python
return Response(serializer.data)
```

DRF devuelve los datos como respuesta de la API.

---

# 13. Analizar el método POST

El método:

```python
def post(self, request):
```

responde a:

```http
POST /api/productos/
```

Los datos enviados por el cliente están disponibles en:

```python
request.data
```

Por ejemplo:

```json
{
    "nombre": "Monitor Samsung",
    "descripcion": "Monitor 24 pulgadas",
    "precio": "1200.00",
    "stock": 8,
    "activo": true
}
```

Creamos el serializer:

```python
serializer = ProductoSerializer(
    data=request.data
)
```

Validamos:

```python
serializer.is_valid()
```

Si es válido:

```python
serializer.save()
```

Y devolvemos:

```python
return Response(
    serializer.data,
    status=status.HTTP_201_CREATED
)
```

---

# 14. Crear la segunda API: detalle del producto

Ahora necesitamos manejar:

```text
GET
PUT
PATCH
DELETE
```

sobre un producto específico.

En `views.py` agregamos:

```python
class ProductoDetailAPIView(APIView):

    def get_object(self, pk):
        try:
            return Producto.objects.get(pk=pk)
        except Producto.DoesNotExist:
            return None

    def get(self, request, pk):
        producto = self.get_object(pk)

        if producto is None:
            return Response(
                {"detail": "Producto no encontrado"},
                status=status.HTTP_404_NOT_FOUND
            )

        serializer = ProductoSerializer(producto)

        return Response(serializer.data)
```

---

# 15. Probar GET de un producto

La URL será:

```text
GET /api/productos/1/
```

La respuesta:

```json
{
    "id": 1,
    "nombre": "Laptop Lenovo",
    "descripcion": "Laptop para oficina",
    "precio": "4500.00",
    "stock": 10,
    "activo": true
}
```

---

# 16. Agregar PUT

Ahora agregamos:

```python
def put(self, request, pk):
    producto = self.get_object(pk)

    if producto is None:
        return Response(
            {"detail": "Producto no encontrado"},
            status=status.HTTP_404_NOT_FOUND
        )

    serializer = ProductoSerializer(
        producto,
        data=request.data
    )

    if serializer.is_valid():
        serializer.save()

        return Response(serializer.data)

    return Response(
        serializer.errors,
        status=status.HTTP_400_BAD_REQUEST
    )
```

`PUT` representa una actualización completa del recurso.

Ejemplo:

```http
PUT /api/productos/1/
```

```json
{
    "nombre": "Laptop Lenovo ThinkPad",
    "descripcion": "Laptop empresarial",
    "precio": "5000.00",
    "stock": 8,
    "activo": true
}
```

---

# 17. Agregar PATCH

`PATCH` se utiliza normalmente para una actualización parcial.

Por ejemplo:

```http
PATCH /api/productos/1/
```

Podemos modificar solamente el stock:

```json
{
    "stock": 20
}
```

Implementación:

```python
def patch(self, request, pk):
    producto = self.get_object(pk)

    if producto is None:
        return Response(
            {"detail": "Producto no encontrado"},
            status=status.HTTP_404_NOT_FOUND
        )

    serializer = ProductoSerializer(
        producto,
        data=request.data,
        partial=True
    )

    if serializer.is_valid():
        serializer.save()

        return Response(serializer.data)

    return Response(
        serializer.errors,
        status=status.HTTP_400_BAD_REQUEST
    )
```

La diferencia importante es:

```python
partial=True
```

Esto permite enviar solamente los campos que queremos modificar.

---

# 18. Agregar DELETE

Finalmente:

```python
def delete(self, request, pk):
    producto = self.get_object(pk)

    if producto is None:
        return Response(
            {"detail": "Producto no encontrado"},
            status=status.HTTP_404_NOT_FOUND
        )

    producto.delete()

    return Response(
        status=status.HTTP_204_NO_CONTENT
    )
```

Ahora tenemos:

```text
GET       → consultar
POST      → crear
PUT       → actualizar completamente
PATCH     → actualizar parcialmente
DELETE    → eliminar
```

---

# 19. Configurar las URLs

Creamos:

```text
productos/urls.py
```

Contenido:

```python
from django.urls import path

from .views import (
    ProductoListCreateAPIView,
    ProductoDetailAPIView,
)


urlpatterns = [
    path(
        "productos/",
        ProductoListCreateAPIView.as_view(),
        name="productos-list"
    ),

    path(
        "productos/<int:pk>/",
        ProductoDetailAPIView.as_view(),
        name="producto-detail"
    ),
]
```

---

# 20. Conectar las URLs de la aplicación

En:

```text
config/urls.py
```

agregamos:

```python
from django.contrib import admin
from django.urls import include, path


urlpatterns = [
    path("admin/", admin.site.urls),

    path(
        "api/",
        include("productos.urls")
    ),
]
```

Nuestra API queda:

```text
/api/productos/
/api/productos/<id>/
```

---

# 21. Ejecutar el servidor

Ejecutamos:

```bash
python manage.py runserver
```

Deberíamos tener:

```text
http://127.0.0.1:8000/
```

---

# 22. Primera prueba — GET

Abrimos:

```text
GET /api/productos/
```

Resultado esperado:

```json
[
    {
        "id": 1,
        "nombre": "Laptop Lenovo",
        "descripcion": "Laptop para oficina",
        "precio": "4500.00",
        "stock": 10,
        "activo": true
    }
]
```

---

# 23. Probar POST

Utilizamos Postman, Insomnia o Thunder Client.

Método:

```text
POST
```

URL:

```text
http://127.0.0.1:8000/api/productos/
```

Body:

```json
{
    "nombre": "Teclado Logitech",
    "descripcion": "Teclado inalámbrico",
    "precio": "250.00",
    "stock": 15,
    "activo": true
}
```

Respuesta esperada:

```text
201 Created
```

---

# 24. Probar GET por ID

```text
GET /api/productos/1/
```

Respuesta:

```json
{
    "id": 1,
    "nombre": "Laptop Lenovo",
    "descripcion": "Laptop para oficina",
    "precio": "4500.00",
    "stock": 10,
    "activo": true
}
```

---

# 25. Probar PATCH

Método:

```text
PATCH
```

URL:

```text
http://127.0.0.1:8000/api/productos/1/
```

Body:

```json
{
    "stock": 30
}
```

Resultado:

```json
{
    "id": 1,
    "nombre": "Laptop Lenovo",
    "descripcion": "Laptop para oficina",
    "precio": "4500.00",
    "stock": 30,
    "activo": true
}
```

---

# 26. Probar DELETE

Método:

```text
DELETE
```

URL:

```text
http://127.0.0.1:8000/api/productos/1/
```

Respuesta:

```text
204 No Content
```

---

# 27. Analizar el problema de APIView

En este punto tenemos una API completamente funcional.

Pero observemos nuestro código.

Para una sola entidad tuvimos que implementar:

```text
GET lista
POST
GET detalle
PUT
PATCH
DELETE
```

Además tuvimos que escribir manualmente:

```python
get_object()
```

validaciones de existencia:

```python
if producto is None:
```

serialización:

```python
ProductoSerializer(...)
```

y respuestas.

Esto funciona, pero podemos reducir considerablemente el código.

Aquí aparece una de las principales ventajas de DRF:

```text
ModelViewSet
```

---

# 28. Introducción a ModelViewSet

DRF proporciona:

```python
ModelViewSet
```

que permite implementar rápidamente operaciones CRUD.

En lugar de escribir manualmente:

```python
get()
post()
put()
patch()
delete()
```

podemos utilizar:

```python
class ProductoViewSet(ModelViewSet):
    ...
```

---

# 29. Crear ProductoViewSet

En `productos/views.py` agregamos:

```python
from rest_framework.viewsets import ModelViewSet
```

Después:

```python
class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
```

Con solamente esto DRF puede proporcionar las operaciones CRUD.

---

# 30. ¿Qué proporciona ModelViewSet?

Nuestro ViewSet puede manejar:

```text
GET     /api/productos/
POST    /api/productos/

GET     /api/productos/1/
PUT     /api/productos/1/
PATCH   /api/productos/1/
DELETE  /api/productos/1/
```

Podemos visualizarlo:

```text
ModelViewSet
      │
      ├── list()
      ├── create()
      ├── retrieve()
      ├── update()
      ├── partial_update()
      └── destroy()
```

No necesitamos implementar manualmente cada método.

---

# 31. Crear un Router

Ahora creamos:

```text
productos/urls.py
```

Podemos utilizar un router:

```python
from rest_framework.routers import DefaultRouter

from .views import ProductoViewSet


router = DefaultRouter()

router.register(
    "productos",
    ProductoViewSet,
    basename="producto"
)

urlpatterns = router.urls
```

---

# 32. ¿Qué hace el Router?

El router genera automáticamente las rutas.

```python
router.register(
    "productos",
    ProductoViewSet
)
```

genera las rutas necesarias para el CRUD.

Conceptualmente:

```text
Router
   ↓
ViewSet
   ↓
CRUD
```

---

# 33. API final

Ahora podemos consultar:

### Listar productos

```http
GET /api/productos/
```

### Crear producto

```http
POST /api/productos/
```

### Consultar producto

```http
GET /api/productos/1/
```

### Actualizar completamente

```http
PUT /api/productos/1/
```

### Actualizar parcialmente

```http
PATCH /api/productos/1/
```

### Eliminar

```http
DELETE /api/productos/1/
```

---

# 34. Comparación

## APIView

Tenemos que implementar manualmente:

```python
def get(...)
def post(...)
def put(...)
def patch(...)
def delete(...)
```

Ventaja:

- Permite comprender qué está ocurriendo.
- Tenemos mayor control.
- Es excelente para aprender.

---

## ModelViewSet

Podemos escribir:

```python
class ProductoViewSet(ModelViewSet):

    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
```

Ventajas:

- Menos código.
- CRUD rápido.
- Integración sencilla con routers.
- Muy útil para APIs CRUD.

---

# 35. Validación del Serializer

Ahora vamos a agregar una validación sencilla.

En:

```text
productos/serializers.py
```

podemos agregar:

```python
class ProductoSerializer(serializers.ModelSerializer):

    class Meta:
        model = Producto
        fields = "__all__"

    def validate_precio(self, value):
        if value <= 0:
            raise serializers.ValidationError(
                "El precio debe ser mayor a 0."
            )

        return value
```

Ahora una petición como:

```json
{
    "nombre": "Producto prueba",
    "descripcion": "Producto inválido",
    "precio": -100,
    "stock": 10,
    "activo": true
}
```

producirá:

```text
400 Bad Request
```

Y una respuesta similar a:

```json
{
    "precio": [
        "El precio debe ser mayor a 0."
    ]
}
```

---

# 36. Validación del stock

También podemos validar el stock:

```python
def validate_stock(self, value):
    if value < 0:
        raise serializers.ValidationError(
            "El stock no puede ser negativo."
        )

    return value
```

Ahora el serializer controla reglas relacionadas con los datos.

Podemos visualizar:

```text
Request JSON
     ↓
Serializer
     ↓
Validación
     ↓
Modelo
```

---

# 37. Práctica final para los estudiantes

Ahora los estudiantes deben comprobar todo el CRUD.

## GET

```text
GET /api/productos/
```

Debe devolver todos los productos.

---

## POST

Crear:

```json
{
    "nombre": "Monitor LG",
    "descripcion": "Monitor de 24 pulgadas",
    "precio": "1500.00",
    "stock": 10,
    "activo": true
}
```

Debe responder:

```text
201 Created
```

---

## GET por ID

```text
GET /api/productos/1/
```

Debe devolver el producto.

---

## PUT

Modificar todos los datos del producto.

```json
{
    "nombre": "Monitor LG 24",
    "descripcion": "Monitor Full HD de 24 pulgadas",
    "precio": "1600.00",
    "stock": 12,
    "activo": true
}
```

---

## PATCH

Modificar únicamente:

```json
{
    "stock": 20
}
```

---

## DELETE

```text
DELETE /api/productos/1/
```

Debe responder:

```text
204 No Content
```

---

# 38. Prueba de errores

También debemos comprobar qué ocurre cuando enviamos información incorrecta.

## Precio negativo

```json
{
    "nombre": "Producto incorrecto",
    "descripcion": "Prueba",
    "precio": -50,
    "stock": 10,
    "activo": true
}
```

Esperamos:

```text
400 Bad Request
```

---

## Producto inexistente

Solicitar:

```text
GET /api/productos/9999/
```

Esperamos:

```text
404 Not Found
```

---

# 39. Resumen de lo construido

Al finalizar tendremos:

```text
                  Django
                     │
                     ▼
               Modelo Producto
                     │
                     ▼
                  Serializer
                     │
                     ▼
                Django REST
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      APIView              ModelViewSet
          │                     │
          ▼                     ▼
       URLs                 Router
          │                     │
          └──────────┬──────────┘
                     ▼
                  REST API
```

---

# 40. Conceptos que deben quedar claros

Al terminar esta práctica, los estudiantes deberían poder explicar:

### API

Una interfaz que permite que diferentes aplicaciones se comuniquen mediante solicitudes y respuestas.

### REST

Un estilo arquitectónico para construir APIs utilizando recursos y operaciones HTTP.

### Endpoint

Una URL específica de nuestra API.

Ejemplo:

```text
/api/productos/
```

### Serializer

Convierte datos entre objetos Django y representaciones como JSON y permite validar los datos recibidos.

### APIView

Permite construir APIs definiendo explícitamente los métodos HTTP.

### ViewSet

Agrupa las operaciones relacionadas con un recurso.

### ModelViewSet

Proporciona operaciones CRUD basadas en un modelo.

### Router

Genera automáticamente las URLs asociadas a un ViewSet.

---

# 41. Orden pedagógico de la práctica

El orden que debemos mantener durante la clase es:

```text
1. Modelo Producto
        ↓
2. Instalar DRF
        ↓
3. Configurar DRF
        ↓
4. Serializer
        ↓
5. APIView
        ↓
6. GET
        ↓
7. POST
        ↓
8. GET por ID
        ↓
9. PUT
        ↓
10. PATCH
        ↓
11. DELETE
        ↓
12. Probar API
        ↓
13. ModelViewSet
        ↓
14. Router
        ↓
15. CRUD automático
        ↓
16. Validaciones
        ↓
17. Prueba final
```

Este orden es importante porque primero los estudiantes entienden **cómo funciona una API por dentro** y después descubren cómo DRF permite reducir el código mediante `ModelViewSet` y `Router`.

---

# 42. Resultado esperado al finalizar la Clase 5

El proyecto debe terminar con una API REST funcional para `Producto`.

```text
GET     /api/productos/
POST    /api/productos/

GET     /api/productos/{id}/
PUT     /api/productos/{id}/
PATCH   /api/productos/{id}/
DELETE  /api/productos/{id}/
```

Y los estudiantes deben haber trabajado con:

```text
Django
Django REST Framework
REST
HTTP
JSON
Serializers
APIView
ModelViewSet
Router
CRUD
Validaciones
Postman / Insomnia / Thunder Client
```

**Nota para la Clase 6:** la autenticación avanzada, JWT, permisos, seguridad, optimización, testing y buenas prácticas se desarrollarán en la siguiente clase.