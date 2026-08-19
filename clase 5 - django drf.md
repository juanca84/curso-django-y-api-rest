# Clase 5 — Desarrollo de APIs REST con Django REST Framework

## Curso

**Desarrollo Backend con Django y Django REST Framework**

## Duración

2 horas

## Tema de la clase

**Desarrollo de APIs REST con Django REST Framework**

---

# 1. Objetivos de aprendizaje

Al finalizar la clase, el estudiante será capaz de:

- Comprender qué es una API y para qué sirve.
- Comprender los principios básicos de una API REST.
- Diferenciar una aplicación web tradicional de una API REST.
- Comprender la función de Django REST Framework (DRF).
- Crear y configurar una API REST dentro de un proyecto Django.
- Crear `Serializers` y comprender su función.
- Convertir modelos Django a representaciones JSON.
- Validar datos recibidos desde una API.
- Utilizar `APIView` para construir endpoints.
- Implementar operaciones CRUD mediante HTTP.
- Comprender la diferencia entre `PUT` y `PATCH`.
- Utilizar `ModelViewSet` para simplificar APIs CRUD.
- Utilizar `Router` para generar rutas automáticamente.
- Comprender la diferencia entre autenticación y autorización.
- Probar endpoints utilizando Postman o Thunder Client.

---

# 2. Contexto: ¿qué hemos hecho hasta ahora?

Hasta la Clase 4 hemos trabajado principalmente con Django como framework para construir aplicaciones web.

El flujo tradicional de Django es:

```text
Navegador
    ↓
HTTP Request
    ↓
URL
    ↓
View
    ↓
Model / ORM
    ↓
Base de datos
    ↓
View
    ↓
Template
    ↓
HTML
    ↓
Navegador
```

Por ejemplo:

```text
GET /productos/
```

Django podría responder con:

```html
<h1>Productos</h1>

<ul>
    <li>Laptop</li>
    <li>Mouse</li>
    <li>Teclado</li>
</ul>
```

Esto funciona perfectamente para una aplicación web tradicional.

Pero ahora imaginemos que queremos que nuestro backend sea consumido por:

- React
- Angular
- Vue
- Una aplicación móvil
- Otro backend
- Postman
- Un sistema externo

En esos casos normalmente no queremos enviar HTML.

Queremos enviar **datos estructurados**.

Por ejemplo:

```json
[
    {
        "id": 1,
        "nombre": "Laptop",
        "precio": "4500.00",
        "stock": 10
    },
    {
        "id": 2,
        "nombre": "Mouse",
        "precio": "100.00",
        "stock": 25
    }
]
```

Aquí aparece la necesidad de construir una **API**.

---

# 3. ¿Qué es una API?

API significa:

> **Application Programming Interface**

En español:

> **Interfaz de Programación de Aplicaciones**

Una API es un mecanismo mediante el cual un software puede comunicarse con otro software utilizando reglas previamente definidas.

Una forma sencilla de entenderlo:

```text
Aplicación A
     ↓
    API
     ↓
Aplicación B
```

La API define:

- Qué operaciones existen.
- Cómo solicitar una operación.
- Qué datos se deben enviar.
- Qué respuesta se recibirá.
- Qué errores pueden producirse.
- Qué permisos son necesarios.

---

# 4. Ejemplo cotidiano de una API

Podemos imaginar un restaurante.

```text
Cliente
   ↓
Mesero
   ↓
Cocina
```

El cliente no entra directamente a la cocina.

El mesero funciona como intermediario.

El cliente solicita:

> Quiero una hamburguesa.

El mesero lleva la solicitud a la cocina.

La cocina prepara el pedido.

Después el mesero devuelve la respuesta.

En una aplicación:

```text
Frontend
   ↓
API
   ↓
Backend
   ↓
Base de datos
```

El frontend no debería conectarse directamente a PostgreSQL.

En su lugar:

```text
React
   ↓
HTTP Request
   ↓
Django REST API
   ↓
Django ORM
   ↓
PostgreSQL
```

---

# 5. API no significa necesariamente REST

Es importante no confundir ambos conceptos.

**API** es un concepto general.

**REST** es una forma de diseñar APIs.

Podemos tener:

```text
APIs
├── REST
├── GraphQL
├── SOAP
├── RPC
└── otras arquitecturas/protocolos
```

Por lo tanto:

> Toda API REST es una API, pero no toda API es REST.

---

# 6. ¿Qué es REST?

REST significa:

> **Representational State Transfer**

Es un estilo arquitectónico utilizado para diseñar servicios web.

REST propone trabajar con **recursos**.

Por ejemplo, nuestro sistema de inventario tiene recursos:

```text
productos
categorias
proveedores
movimientos
usuarios
```

Cada recurso puede tener una URL.

Por ejemplo:

```text
/api/productos/
```

---

# 7. Recursos en REST

Supongamos que tenemos tres productos.

El recurso general es:

```text
/api/productos/
```

Un producto específico puede ser:

```text
/api/productos/1/
```

Entonces:

```text
/api/productos/
        ↑
    colección

/api/productos/1/
        ↑
      recurso
```

Esta diferencia es importante.

### Colección

```text
/api/productos/
```

Representa el conjunto de productos.

### Recurso individual

```text
/api/productos/1/
```

Representa un producto específico.

---

# 8. HTTP y REST

REST utiliza principalmente los métodos HTTP.

Los más importantes para nuestro CRUD son:

| Método | Propósito |
|---|---|
| GET | Obtener información |
| POST | Crear un recurso |
| PUT | Actualizar completamente |
| PATCH | Actualizar parcialmente |
| DELETE | Eliminar |

Podemos relacionarlos con CRUD:

| CRUD | HTTP |
|---|---|
| Create | POST |
| Read | GET |
| Update | PUT / PATCH |
| Delete | DELETE |

---

# 9. Ejemplo completo de REST

Para productos:

### Listar productos

```http
GET /api/productos/
```

### Obtener un producto

```http
GET /api/productos/1/
```

### Crear producto

```http
POST /api/productos/
```

### Actualizar producto

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

# 10. ¿Qué devuelve una API?

Normalmente una API REST devuelve datos estructurados.

Uno de los formatos más utilizados es **JSON**.

Ejemplo:

```json
{
    "id": 1,
    "nombre": "Laptop Lenovo",
    "precio": "4500.00",
    "stock": 10
}
```

JSON significa:

> JavaScript Object Notation

Aunque originalmente está basado en la sintaxis de JavaScript, actualmente es utilizado por prácticamente cualquier lenguaje.

---

# 11. JSON y Python

En Python podemos tener:

```python
producto = {
    "id": 1,
    "nombre": "Laptop",
    "precio": 4500,
    "stock": 10
}
```

Y representarlo como JSON:

```json
{
    "id": 1,
    "nombre": "Laptop",
    "precio": 4500,
    "stock": 10
}
```

La API utilizará este tipo de representación para intercambiar información.

---

# 12. Aplicación tradicional vs API REST

## Django tradicional

```text
Browser
   ↓
GET /productos/
   ↓
Django View
   ↓
Model
   ↓
Template
   ↓
HTML
```

Respuesta:

```html
<h1>Productos</h1>
```

## Django REST Framework

```text
Frontend
   ↓
GET /api/productos/
   ↓
DRF View
   ↓
Serializer
   ↓
Model
   ↓
Database
```

Respuesta:

```json
[
    {
        "id": 1,
        "nombre": "Laptop"
    }
]
```

---

# 13. ¿Qué es Django REST Framework?

Django REST Framework, conocido como **DRF**, es un framework construido sobre Django para facilitar el desarrollo de APIs web.

DRF proporciona herramientas para:

- Serialización.
- Validación.
- Views para APIs.
- Autenticación.
- Permisos.
- Respuestas HTTP.
- ViewSets.
- Routers.
- Browsable API.
- Manejo de errores.

Conceptualmente:

```text
Django
│
├── ORM
├── Templates
├── Forms
├── Authentication
├── Middleware
├── Admin
│
└── Django REST Framework
      ├── Serializers
      ├── API Views
      ├── ViewSets
      ├── Routers
      ├── Authentication
      └── Permissions
```

DRF no reemplaza Django.

> DRF extiende Django para facilitar la construcción de APIs.

---

# 14. Instalación de Django REST Framework

Instalar:

```bash
pip install djangorestframework
```

Verificar:

```bash
pip show djangorestframework
```

Después agregarlo a `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # ...

    'rest_framework',

    'productos',
]
```

---

# 15. Estructura recomendada

Podemos tener:

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
    ├── migrations/
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── serializers.py
    ├── views.py
    ├── urls.py
    └── ...
```

Creamos:

```text
productos/serializers.py
```

---

# 16. ¿Qué es un Serializer?

Este es uno de los conceptos más importantes de la clase.

Un Serializer de DRF permite transformar datos entre diferentes representaciones.

Principalmente:

```text
Modelo / objeto Python
        ↓
    Serializer
        ↓
       JSON
```

Y también:

```text
JSON recibido
        ↓
    Serializer
        ↓
Datos validados
        ↓
Modelo / objeto Python
```

Por eso un serializer participa en dos procesos:

1. Serialización.
2. Deserialización y validación.

---

# 17. Serialización

Supongamos que tenemos un modelo:

```python
producto = Producto.objects.get(pk=1)
```

Tenemos un objeto Django.

```text
Producto
├── id = 1
├── nombre = "Laptop"
├── precio = 4500
└── stock = 10
```

Una API necesita representar este objeto como datos que pueda consumir otro sistema.

El serializer lo convierte conceptualmente en:

```json
{
    "id": 1,
    "nombre": "Laptop",
    "precio": "4500.00",
    "stock": 10
}
```

Este proceso es **serialización**.

---

# 18. Deserialización y validación

Ahora ocurre el proceso contrario.

El cliente envía:

```json
{
    "nombre": "Monitor",
    "precio": "1200.00",
    "stock": 15
}
```

DRF recibe los datos.

El serializer puede:

1. Leer los datos.
2. Validarlos.
3. Convertirlos a tipos apropiados.
4. Crear o actualizar un objeto.

Conceptualmente:

```text
JSON
 ↓
Serializer
 ↓
Validación
 ↓
Datos Python
 ↓
Modelo
```

---

# 19. ModelSerializer

DRF proporciona `ModelSerializer`.

Es una versión especializada para trabajar con modelos Django.

Supongamos:

```python
from django.db import models


class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    precio = models.DecimalField(
        max_digits=10,
        decimal_places=2
    )
    stock = models.PositiveIntegerField()
```

Creamos:

```python
from rest_framework import serializers
from .models import Producto


class ProductoSerializer(serializers.ModelSerializer):

    class Meta:
        model = Producto
        fields = '__all__'
```

---

# 20. ¿Qué hace ModelSerializer?

`ModelSerializer` puede generar automáticamente campos basándose en el modelo.

Por ejemplo:

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    precio = models.DecimalField(...)
    stock = models.PositiveIntegerField()
```

El serializer puede generar:

```text
id
nombre
precio
stock
```

No tenemos que declarar manualmente cada campo en los casos sencillos.

---

# 21. Comparación con ModelForm

Los estudiantes ya conocen `ModelForm`.

En Clase 4 podríamos tener:

```python
class ProductoForm(forms.ModelForm):

    class Meta:
        model = Producto
        fields = '__all__'
```

En DRF:

```python
class ProductoSerializer(serializers.ModelSerializer):

    class Meta:
        model = Producto
        fields = '__all__'
```

La idea es parecida, pero el objetivo es diferente.

| ModelForm | ModelSerializer |
|---|---|
| Formularios HTML | APIs |
| Entrada desde formulario | Entrada JSON |
| Salida para HTML | Salida JSON |
| Validación | Validación |
| `Model` | `Model` |

No debemos decir que son exactamente iguales.

El concepto importante es:

> Ambos pueden utilizar un modelo Django para facilitar la validación y representación de datos, pero están diseñados para contextos diferentes.

---

# 22. `fields`

Podemos utilizar:

```python
fields = '__all__'
```

Pero también podemos especificar campos:

```python
class ProductoSerializer(serializers.ModelSerializer):

    class Meta:
        model = Producto
        fields = [
            'id',
            'nombre',
            'precio',
            'stock'
        ]
```

Esto permite controlar qué información exponemos mediante la API.

---

# 23. ¿Por qué no siempre debemos utilizar `__all__`?

En proyectos reales puede existir información que no queremos exponer.

Por ejemplo:

```text
id
nombre
precio
stock
costo_interno
margen
usuario_creador
```

Quizá el cliente solo debería recibir:

```text
id
nombre
precio
stock
```

Por eso es importante controlar los campos que expone el serializer.

---

# 24. Serializer como capa de validación

Una de las ventajas importantes de DRF es que el serializer puede validar datos.

Ejemplo:

```python
serializer = ProductoSerializer(
    data=request.data
)

if serializer.is_valid():
    serializer.save()
```

La llamada:

```python
serializer.is_valid()
```

ejecuta las validaciones disponibles.

Si los datos son correctos:

```python
True
```

Si existen errores:

```python
False
```

Los errores pueden consultarse con:

```python
serializer.errors
```

---

# 25. Ejemplo de datos inválidos

Supongamos que `stock` es:

```python
models.PositiveIntegerField()
```

El cliente envía:

```json
{
    "nombre": "Laptop",
    "precio": "4500.00",
    "stock": -10
}
```

El serializer puede detectar que el valor no cumple las reglas.

Podemos devolver:

```json
{
    "stock": [
        "Ensure this value is greater than or equal to 0."
    ]
}
```

Esto es una ventaja importante:

> La API no debería confiar ciegamente en los datos enviados por el cliente.

---

# 26. API Views

Ahora necesitamos recibir las peticiones HTTP.

DRF proporciona diferentes tipos de Views.

Las principales que debemos conocer son:

```text
APIView
GenericAPIView
Generic Views
ViewSets
ModelViewSet
```

No necesitamos profundizar en todas durante esta clase.

Nos concentraremos en:

```text
APIView
ModelViewSet
Router
```

---

# 27. ¿Qué es APIView?

`APIView` es una clase de DRF diseñada para crear endpoints basados en métodos HTTP.

Ejemplo:

```python
from rest_framework.views import APIView
from rest_framework.response import Response


class ProductoAPIView(APIView):

    def get(self, request):
        ...

    def post(self, request):
        ...
```

Los métodos corresponden a HTTP:

```text
GET      → get()
POST     → post()
PUT      → put()
PATCH    → patch()
DELETE   → delete()
```

---

# 28. Ventaja de APIView

`APIView` permite tener bastante control sobre el comportamiento.

Podemos decidir:

- Cómo obtener datos.
- Cómo validar.
- Cómo responder.
- Qué código HTTP utilizar.
- Qué lógica ejecutar.
- Qué permisos aplicar.

Es excelente para aprender qué está pasando detrás de una API.

---

# 29. Primera API con APIView

En `views.py`:

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

from .models import Producto
from .serializers import ProductoSerializer


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

# 30. Analizando el GET

Cuando llega:

```http
GET /api/productos/
```

DRF ejecuta:

```python
def get(self, request):
```

Obtenemos los productos:

```python
productos = Producto.objects.all()
```

Aquí estamos utilizando el ORM de Django que ya conocemos.

Después:

```python
serializer = ProductoSerializer(
    productos,
    many=True
)
```

---

# 31. ¿Por qué `many=True`?

Porque estamos serializando múltiples objetos.

```python
Producto.objects.all()
```

puede devolver:

```text
Producto 1
Producto 2
Producto 3
Producto 4
```

Por eso:

```python
many=True
```

significa:

> Estoy serializando una colección de objetos.

Para un único objeto:

```python
serializer = ProductoSerializer(producto)
```

no necesitamos `many=True`.

---

# 32. `Response`

Finalmente:

```python
return Response(serializer.data)
```

`Response` es proporcionado por DRF:

```python
from rest_framework.response import Response
```

DRF se encarga de construir una respuesta HTTP apropiada.

El resultado podría ser:

```json
[
    {
        "id": 1,
        "nombre": "Laptop",
        "precio": "4500.00",
        "stock": 10
    },
    {
        "id": 2,
        "nombre": "Mouse",
        "precio": "100.00",
        "stock": 25
    }
]
```

---

# 33. POST y `request.data`

Cuando el cliente envía:

```http
POST /api/productos/
```

con:

```json
{
    "nombre": "Teclado",
    "precio": "150.00",
    "stock": 20
}
```

DRF nos permite acceder a los datos mediante:

```python
request.data
```

Por ejemplo:

```python
print(request.data)
```

conceptualmente produce:

```python
{
    "nombre": "Teclado",
    "precio": "150.00",
    "stock": 20
}
```

---

# 34. Crear mediante Serializer

Recibimos:

```python
request.data
```

y lo pasamos al serializer:

```python
serializer = ProductoSerializer(
    data=request.data
)
```

Después:

```python
serializer.is_valid()
```

Si todo está correcto:

```python
serializer.save()
```

El serializer puede crear el objeto.

---

# 35. `serializer.save()`

Una confusión común es pensar que:

```python
serializer.save()
```

siempre ejecuta directamente:

```python
Producto.objects.create(...)
```

La idea correcta es:

- Si estamos creando un objeto, el serializer utiliza la lógica de creación.
- Si estamos actualizando un objeto existente, utiliza la lógica correspondiente de actualización.

Con un `ModelSerializer` estándar, DRF proporciona este comportamiento automáticamente.

---

# 36. PUT y PATCH

Para actualizar debemos distinguir:

## PUT

Representa una actualización completa.

Ejemplo:

```http
PUT /api/productos/1/
```

```json
{
    "nombre": "Laptop Lenovo",
    "precio": "5000.00",
    "stock": 15
}
```

## PATCH

Representa una actualización parcial.

```http
PATCH /api/productos/1/
```

```json
{
    "stock": 20
}
```

PATCH es especialmente útil cuando solamente necesitamos modificar uno o algunos campos.

---

# 37. Serializer para actualizar

Para actualizar necesitamos proporcionar la instancia:

```python
producto = Producto.objects.get(pk=pk)
```

Y luego:

```python
serializer = ProductoSerializer(
    producto,
    data=request.data
)
```

Para actualización parcial:

```python
serializer = ProductoSerializer(
    producto,
    data=request.data,
    partial=True
)
```

La opción:

```python
partial=True
```

indica que no es obligatorio enviar todos los campos.

---

# 38. Endpoint de detalle

Podemos crear:

```python
class ProductoDetailAPIView(APIView):

    def get_object(self, pk):
        try:
            return Producto.objects.get(pk=pk)
        except Producto.DoesNotExist:
            return None
```

Esta función nos ayuda a localizar el producto.

---

# 39. GET de un producto

```python
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

Petición:

```http
GET /api/productos/1/
```

Respuesta:

```json
{
    "id": 1,
    "nombre": "Laptop",
    "precio": "4500.00",
    "stock": 10
}
```

---

# 40. DELETE

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

Petición:

```http
DELETE /api/productos/1/
```

Respuesta:

```text
204 No Content
```

---

# 41. Códigos de estado HTTP

Una API debe utilizar códigos HTTP correctamente.

| Código | Significado |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 500 | Internal Server Error |

Los más importantes durante esta clase:

```text
200 → operación exitosa
201 → recurso creado
204 → operación exitosa sin contenido
400 → datos inválidos
404 → recurso no encontrado
```

---

# 42. `status` de DRF

En lugar de escribir:

```python
return Response(
    serializer.data,
    status=201
)
```

podemos utilizar:

```python
return Response(
    serializer.data,
    status=status.HTTP_201_CREATED
)
```

Esto hace el código más legible.

Importamos:

```python
from rest_framework import status
```

---

# 43. URLs

Creamos:

```text
productos/urls.py
```

```python
from django.urls import path

from .views import (
    ProductoListCreateAPIView,
    ProductoDetailAPIView
)


urlpatterns = [
    path(
        'productos/',
        ProductoListCreateAPIView.as_view()
    ),
    path(
        'productos/<int:pk>/',
        ProductoDetailAPIView.as_view()
    ),
]
```

En las URLs principales:

```python
from django.urls import include, path


urlpatterns = [
    path(
        'api/',
        include('productos.urls')
    ),
]
```

---

# 44. Endpoints resultantes

Tendremos:

```text
GET     /api/productos/
POST    /api/productos/

GET     /api/productos/1/
PUT     /api/productos/1/
PATCH   /api/productos/1/
DELETE  /api/productos/1/
```

Esto representa nuestro CRUD REST.

---

# 45. Probar con Postman o Thunder Client

## GET

```http
GET http://localhost:8000/api/productos/
```

## POST

```http
POST http://localhost:8000/api/productos/
Content-Type: application/json
```

Body:

```json
{
    "nombre": "Monitor",
    "precio": "1200.00",
    "stock": 15
}
```

## PATCH

```http
PATCH http://localhost:8000/api/productos/1/
Content-Type: application/json
```

Body:

```json
{
    "stock": 25
}
```

## DELETE

```http
DELETE http://localhost:8000/api/productos/1/
```

---

# 46. De APIView a ViewSet

Después de construir la API manualmente, debemos mostrar que DRF proporciona una forma más compacta.

Nuestro código anterior requiere métodos:

```text
get()
post()
put()
patch()
delete()
```

Para APIs CRUD esto puede producir bastante código repetitivo.

DRF proporciona los **ViewSets**.

---

# 47. ¿Qué es un ViewSet?

Un ViewSet agrupa la lógica relacionada con un recurso.

En lugar de pensar:

```text
GET → método get()
POST → método post()
```

con un `ViewSet` pensamos en acciones:

```text
list
retrieve
create
update
partial_update
destroy
```

Por ejemplo:

```python
from rest_framework.viewsets import ModelViewSet


class ProductoViewSet(ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
```

---

# 48. ¿Qué proporciona ModelViewSet?

`ModelViewSet` incluye acciones para CRUD:

```text
list
retrieve
create
update
partial_update
destroy
```

Por eso:

```python
class ProductoViewSet(ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
```

puede proporcionar automáticamente las operaciones principales de nuestro CRUD.

---

# 49. ViewSet vs APIView

| APIView | ModelViewSet |
|---|---|
| Mayor control manual | Mayor automatización |
| Métodos HTTP explícitos | Acciones de ViewSet |
| Más código | Menos código |
| Excelente para aprender | Excelente para CRUD |
| Flexible | Convencional |

No significa que `ModelViewSet` sea siempre mejor.

La elección depende del problema.

---

# 50. Routers

Si utilizamos ViewSets necesitamos definir sus rutas.

DRF proporciona `Router`.

```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()

router.register(
    'productos',
    ProductoViewSet
)
```

Después:

```python
urlpatterns = router.urls
```

El Router genera automáticamente las rutas correspondientes.

---

# 51. ViewSet + Router

El flujo queda:

```text
Cliente
   ↓
URL
   ↓
Router
   ↓
ViewSet
   ↓
Serializer
   ↓
Model
   ↓
Database
```

Esto permite construir APIs CRUD con mucho menos código.

---

# 52. Endpoints generados

Con:

```python
router.register(
    'productos',
    ProductoViewSet
)
```

tendremos conceptualmente:

```text
GET       /productos/
POST      /productos/

GET       /productos/1/
PUT       /productos/1/
PATCH     /productos/1/
DELETE    /productos/1/
```

---

# 53. ¿Cuándo usar APIView?

`APIView` es útil cuando:

- Necesitamos lógica personalizada.
- El endpoint no representa un CRUD tradicional.
- Necesitamos un control específico sobre HTTP.
- Queremos combinar diferentes fuentes de datos.
- Queremos entender y controlar el flujo manualmente.

Ejemplo:

```text
POST /api/productos/importar/
```

Quizá realiza una operación específica que no encaja directamente en CRUD.

---

# 54. ¿Cuándo usar ModelViewSet?

`ModelViewSet` es especialmente útil cuando:

- Tenemos un modelo.
- Queremos CRUD.
- Las operaciones siguen las convenciones habituales.
- Queremos reducir código repetitivo.

Ejemplo:

```text
Producto
Categoría
Proveedor
```

Todos pueden ser buenos candidatos para ViewSets.

---

# 55. Autenticación vs autorización

En esta clase solamente introduciremos el concepto.

No debemos confundir:

## Autenticación

Pregunta:

> ¿Quién eres?

Ejemplo:

```text
Usuario
   ↓
Login
   ↓
Credenciales
   ↓
Token / Sesión
```

## Autorización

Pregunta:

> ¿Qué puedes hacer?

Ejemplo:

```text
Usuario autenticado
        │
        ├── GET productos       ✓
        ├── POST productos      ✓
        ├── PATCH productos     ✓
        └── DELETE productos    ✗
```

La autenticación y autorización serán profundizadas en la Clase 6.

---

# 56. Permisos de DRF

DRF permite controlar el acceso mediante permisos.

Por ejemplo:

```python
from rest_framework.permissions import IsAuthenticated
```

Y:

```python
class ProductoViewSet(ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    permission_classes = [IsAuthenticated]
```

Esto significa:

> Solo usuarios autenticados pueden acceder al ViewSet.

En la Clase 6 trabajaremos con mayor profundidad:

- Autenticación.
- JWT.
- Permisos.
- Seguridad.
- CORS.
- Buenas prácticas.

---

# 57. Flujo completo de una petición

Este es uno de los conceptos que el estudiante debe llevarse de la clase.

Cuando un cliente ejecuta:

```http
POST /api/productos/
```

podemos imaginar:

```text
Cliente
   │
   │ HTTP POST + JSON
   ▼
URL Router
   │
   ▼
APIView / ViewSet
   │
   ▼
request.data
   │
   ▼
Serializer
   │
   ├── Validación
   │
   ▼
Model / ORM
   │
   ▼
PostgreSQL
   │
   ▼
Objeto creado
   │
   ▼
Serializer
   │
   ▼
Response
   │
   ▼
JSON
   │
   ▼
Cliente
```

---

# 58. Conceptos clave de la clase

Al finalizar, el estudiante debería poder explicar:

### API

Interfaz que permite la comunicación entre aplicaciones.

### REST

Estilo arquitectónico para diseñar APIs utilizando recursos y operaciones HTTP.

### Endpoint

URL específica de una API que permite acceder a una funcionalidad o recurso.

Ejemplo:

```text
/api/productos/
```

### Serializer

Componente de DRF encargado de representar datos y participar en la validación/deserialización.

### APIView

Clase de DRF que permite implementar endpoints utilizando métodos HTTP.

### ViewSet

Agrupa las operaciones relacionadas con un recurso.

### ModelViewSet

Proporciona las operaciones CRUD comunes de un modelo.

### Router

Genera automáticamente URLs para ViewSets.

---

# 59. Ejercicio práctico

Crear una API REST para `Categoria`.

Debe implementar:

```text
GET     /api/categorias/
POST    /api/categorias/

GET     /api/categorias/<id>/
PUT     /api/categorias/<id>/
PATCH   /api/categorias/<id>/
DELETE  /api/categorias/<id>/
```

El estudiante debe crear:

```text
categorias/
├── serializers.py
├── views.py
└── urls.py
```

Debe probar todos los endpoints.

---

# 60. Ejercicio adicional

Implementar un filtro sencillo para productos:

```http
GET /api/productos/?nombre=laptop
```

La API debería devolver solamente productos cuyo nombre coincida con el criterio.

Este ejercicio sirve como introducción a:

- Query parameters.
- Filtrado.
- Optimización de consultas.

Estos conceptos se profundizarán en la Clase 6.

---

# 61. Reto final de la clase

Crear un `ProductoViewSet` utilizando:

```python
ModelViewSet
```

y registrar el ViewSet mediante:

```python
DefaultRouter
```

El estudiante debe conseguir:

```text
GET       /api/productos/
POST      /api/productos/
GET       /api/productos/1/
PUT       /api/productos/1/
PATCH     /api/productos/1/
DELETE    /api/productos/1/
```

Después debe comprobar cada endpoint utilizando Postman o Thunder Client.

---

# 62. Resultado esperado

Al terminar la Clase 5, el proyecto debe haber evolucionado desde:

```text
Django
   ↓
Templates
   ↓
HTML
```

hacia:

```text
Django
   ↓
Django REST Framework
   ↓
API REST
   ↓
JSON
```

Y el sistema de inventario tendrá:

```text
Sistema de Inventario
│
├── Django
│   ├── Templates
│   ├── Forms
│   ├── Authentication
│   └── ORM
│
├── PostgreSQL
│
└── REST API
    │
    ├── Productos
    │   ├── GET
    │   ├── POST
    │   ├── PUT
    │   ├── PATCH
    │   └── DELETE
    │
    └── Categorías
        └── CRUD
```

---

# 63. Preparación para la Clase 6

La Clase 5 debe dejar preparada la base para la siguiente clase.

## Clase 6 — Seguridad, Optimización y Buenas Prácticas

Trabajaremos:

### Seguridad

- Autenticación.
- JWT.
- Access Token.
- Refresh Token.
- Permissions.
- Roles.
- CORS.
- Protección de endpoints.

### Optimización

- QuerySets eficientes.
- `select_related`.
- `prefetch_related`.
- Evitar consultas innecesarias.
- Paginación.
- Filtrado.
- Optimización de consultas.

### Testing

- Tests de modelos.
- Tests de endpoints.
- `APITestCase`.
- Validación de respuestas HTTP.

### Logging

- Qué es logging.
- Niveles de log.
- Configuración básica.
- Registro de errores y eventos.

### Buenas prácticas

- Separación de responsabilidades.
- Estructura del proyecto.
- Variables de entorno.
- Manejo de errores.
- Serializers bien diseñados.
- Permisos.
- Código mantenible.

---

# 64. Mensaje final para el estudiante

La idea central de esta clase es comprender que Django puede funcionar como un backend completo para diferentes tipos de clientes.

No estamos abandonando Django.

Estamos agregando una nueva forma de utilizarlo:

```text
                    Django
                       │
          ┌────────────┴────────────┐
          │                         │
       Templates                   DRF
          │                         │
         HTML                      JSON
          │                         │
      Navegador              React / Angular /
                             Vue / Mobile /
                             otros sistemas
```

El objetivo no es memorizar todas las clases de DRF.

El objetivo es comprender el flujo:

```text
HTTP Request
     ↓
View / ViewSet
     ↓
Serializer
     ↓
ORM
     ↓
Database
     ↓
Serializer
     ↓
HTTP Response
     ↓
JSON
```

Ese flujo será la base para desarrollar APIs profesionales con Django.
