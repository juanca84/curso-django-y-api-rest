# Clase 4 — ORM, Formularios, GET/POST y CRUD con Django

## Datos generales

**Curso:** Desarrollo Backend con Django y API REST Framework

**Duración:** 2 horas

**Unidad:** ORM, vistas, formularios y CRUD

---

# Objetivo de la clase

Al finalizar la clase, el estudiante será capaz de:

- Comprender cómo Django interactúa con la base de datos mediante el ORM.
- Diferenciar un objeto de un QuerySet.
- Realizar consultas utilizando `all()`, `filter()` y `get()`.
- Comprender el ciclo de una petición HTTP en Django.
- Diferenciar claramente los métodos `GET` y `POST`.
- Crear formularios con `forms.Form` y `forms.ModelForm`.
- Validar datos enviados desde un formulario.
- Implementar un CRUD completo para el modelo `Producto`.
- Utilizar `get_object_or_404()`.
- Utilizar `redirect()`.
- Aplicar el patrón Post/Redirect/Get.
- Integrar modelos, formularios, vistas, URLs y templates.

---

# Proyecto de la clase

Continuaremos con el proyecto desarrollado en las clases anteriores.

La aplicación será:

```text
productos
```

Durante esta clase implementaremos el CRUD completo del modelo `Producto`.

CRUD significa:

| Operación | Significado | Método HTTP |
|---|---|---|
| Create | Crear | GET + POST |
| Read | Listar | GET |
| Update | Editar | GET + POST |
| Delete | Eliminar | GET + POST |

El flujo general será:

```text
                     PRODUCTOS
                         |
              +----------+----------+
              |          |          |
              v          v          v
            LISTAR     CREAR      EDITAR
             GET      GET/POST   GET/POST
              |          |          |
              |          v          v
              |      ModelForm  ModelForm
              |          |          |
              +----------+----------+
                         |
                         v
                      ELIMINAR
                      GET/POST
```

---

# Estructura final de la aplicación

Al terminar la práctica tendremos:

```text
productos/
├── __init__.py
├── admin.py
├── apps.py
├── forms.py
├── models.py
├── urls.py
├── views.py
└── templates/
    └── productos/
        ├── lista.html
        ├── crear.html
        ├── editar.html
        └── eliminar.html
```

---

# Parte 1 — Repaso del ORM

## ¿Qué es el ORM?

ORM significa **Object Relational Mapping**.

Permite trabajar con la base de datos utilizando objetos de Python en lugar de escribir SQL directamente.

Por ejemplo, en SQL escribiríamos:

```sql
SELECT * FROM productos_producto;
```

En Django escribimos:

```python
Producto.objects.all()
```

Django traduce esa instrucción a SQL.

---

# El modelo Producto

Trabajaremos con el siguiente modelo:

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

Cada instancia del modelo representa una fila de la tabla.

Ejemplo:

```text
id: 1
nombre: Laptop Lenovo
descripcion: Laptop para desarrollo
precio: 6500.00
stock: 10
activo: True
```

---

# Migraciones

Después de crear o modificar el modelo ejecutamos:

```bash
python manage.py makemigrations
```

Luego:

```bash
python manage.py migrate
```

Podemos revisar las migraciones con:

```bash
python manage.py showmigrations
```

---

# QuerySets

## Obtener todos los productos

```python
productos = Producto.objects.all()
```

Esto devuelve un `QuerySet`.

Podemos recorrerlo:

```python
for producto in productos:
    print(producto.nombre)
```

---

## Filtrar productos

Productos activos:

```python
Producto.objects.filter(activo=True)
```

Productos con stock:

```python
Producto.objects.filter(stock__gt=0)
```

Productos con precio mayor o igual a 1000:

```python
Producto.objects.filter(precio__gte=1000)
```

---

## Obtener un solo objeto

```python
producto = Producto.objects.get(id=1)
```

Puede generar excepciones si el objeto no existe.

Por eso en las vistas utilizaremos:

```python
get_object_or_404()
```

---

# get_object_or_404

Importamos:

```python
from django.shortcuts import get_object_or_404
```

Uso:

```python
producto = get_object_or_404(Producto, id=1)
```

Si existe, devuelve el objeto.

Si no existe, Django responde con HTTP 404.

---

# Crear registros con ORM

Ejemplo:

```python
Producto.objects.create(
    nombre="Monitor",
    descripcion="Monitor 24 pulgadas",
    precio=1500,
    stock=5
)
```

Otra forma:

```python
producto = Producto(
    nombre="Monitor",
    descripcion="Monitor 24 pulgadas",
    precio=1500,
    stock=5
)

producto.save()
```

En esta clase utilizaremos formularios para recibir los datos.

---

# Parte 2 — Formularios y HTTP

# ¿Qué es un formulario?

Un formulario permite al usuario introducir datos desde el navegador.

Ejemplo:

```text
Nombre:       [____________]

Descripción:  [____________]

Precio:       [____________]

Stock:        [____________]

             [ Guardar ]
```

El navegador envía esos datos al servidor.

Django puede:

- Crear los campos.
- Recibir los datos.
- Validarlos.
- Mostrar errores.
- Guardarlos en la base de datos.

---

# GET y POST

## GET

Se utiliza normalmente para solicitar información.

Ejemplo:

```text
GET /productos/
```

Significa:

> Mostrar la lista de productos.

Otro ejemplo:

```text
GET /productos/crear/
```

Significa:

> Mostrar el formulario vacío.

---

## POST

Se utiliza para enviar datos.

Ejemplo:

```text
POST /productos/crear/
```

Datos enviados:

```text
nombre=Monitor
precio=1500
stock=5
```

El servidor recibe esos datos y los procesa.

---

# Flujo GET y POST

```text
                 GET
                  |
                  v
         Mostrar formulario
                  |
                  v
         Usuario completa
             el formulario
                  |
                  v
                 POST
                  |
                  v
          Validar formulario
                  |
           +------+------+
           |             |
        Inválido        Válido
           |             |
           v             v
     Mostrar errores   Guardar
                         |
                         v
                      Redirect
```

---

# forms.Form vs forms.ModelForm

## forms.Form

Se utiliza cuando el formulario no está directamente relacionado con un modelo.

Ejemplo:

```python
from django import forms


class BusquedaProductoForm(forms.Form):
    nombre = forms.CharField(required=False)
```

---

## forms.ModelForm

Se utiliza cuando el formulario representa un modelo.

Para nuestro CRUD utilizaremos `ModelForm`.

---

# Crear el formulario

Archivo:

```text
productos/forms.py
```

Contenido:

```python
from django import forms
from .models import Producto


class ProductoForm(forms.ModelForm):

    class Meta:
        model = Producto

        fields = [
            "nombre",
            "descripcion",
            "precio",
            "stock",
            "activo",
        ]
```

Django genera automáticamente los campos a partir del modelo.

---

# Validación

Cuando ejecutamos:

```python
form.is_valid()
```

Django valida:

- Campos obligatorios.
- Tipos de datos.
- Longitudes.
- Reglas del modelo.

---

# Validación personalizada

Ejemplo:

```python
class ProductoForm(forms.ModelForm):

    class Meta:
        model = Producto
        fields = [
            "nombre",
            "descripcion",
            "precio",
            "stock",
            "activo",
        ]

    def clean_precio(self):
        precio = self.cleaned_data["precio"]

        if precio <= 0:
            raise forms.ValidationError(
                "El precio debe ser mayor que cero."
            )

        return precio
```

---

# Parte 3 — CRUD

# READ — Listar productos

Vista:

```python
from django.shortcuts import render
from .models import Producto


def lista_productos(request):

    productos = Producto.objects.all()

    return render(
        request,
        "productos/lista.html",
        {
            "productos": productos
        }
    )
```

Template:

```html
{% extends "base.html" %}

{% block content %}

<h1>Productos</h1>

<a href="{% url 'crear_producto' %}">
    Crear producto
</a>

<table>

    <thead>
        <tr>
            <th>Nombre</th>
            <th>Precio</th>
            <th>Stock</th>
            <th>Acciones</th>
        </tr>
    </thead>

    <tbody>

        {% for producto in productos %}

        <tr>
            <td>{{ producto.nombre }}</td>
            <td>{{ producto.precio }}</td>
            <td>{{ producto.stock }}</td>

            <td>
                <a href="{% url 'editar_producto' producto.id %}">
                    Editar
                </a>

                <a href="{% url 'eliminar_producto' producto.id %}">
                    Eliminar
                </a>
            </td>
        </tr>

        {% empty %}

        <tr>
            <td colspan="4">
                No existen productos.
            </td>
        </tr>

        {% endfor %}

    </tbody>

</table>

{% endblock %}
```

---

# CREATE — Crear producto

## Vista

```python
from django.shortcuts import render, redirect
from .forms import ProductoForm


def crear_producto(request):

    if request.method == "POST":

        form = ProductoForm(request.POST)

        if form.is_valid():

            form.save()

            return redirect("lista_productos")

    else:

        form = ProductoForm()

    return render(
        request,
        "productos/crear.html",
        {
            "form": form
        }
    )
```

---

# Explicación de la vista

Cuando llega un GET:

```python
form = ProductoForm()
```

Se crea un formulario vacío.

Cuando llega un POST:

```python
form = ProductoForm(request.POST)
```

Se cargan los datos enviados.

Luego:

```python
form.is_valid()
```

Valida la información.

Si es correcta:

```python
form.save()
```

Guarda el producto.

Finalmente:

```python
redirect("lista_productos")
```

Redirige al listado.

---

# Template crear.html

```html
{% extends "base.html" %}

{% block content %}

<h1>Crear producto</h1>

<form method="post">

    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit">
        Guardar
    </button>

</form>

{% endblock %}
```

---

# CSRF

Todo formulario POST debe incluir:

```django
{% csrf_token %}
```

Django utiliza este token para proteger el formulario contra ataques CSRF.

---

# UPDATE — Editar producto

Vista:

```python
from django.shortcuts import get_object_or_404


def editar_producto(request, id):

    producto = get_object_or_404(
        Producto,
        id=id
    )

    if request.method == "POST":

        form = ProductoForm(
            request.POST,
            instance=producto
        )

        if form.is_valid():

            form.save()

            return redirect(
                "lista_productos"
            )

    else:

        form = ProductoForm(
            instance=producto
        )

    return render(
        request,
        "productos/editar.html",
        {
            "form": form,
            "producto": producto
        }
    )
```

---

# ¿Qué hace instance?

Cuando usamos:

```python
ProductoForm(instance=producto)
```

Django llena el formulario con los datos existentes.

Cuando usamos:

```python
ProductoForm(
    request.POST,
    instance=producto
)
```

Django actualiza ese objeto existente en lugar de crear uno nuevo.

---

# Template editar.html

```html
{% extends "base.html" %}

{% block content %}

<h1>Editar producto</h1>

<form method="post">

    {% csrf_token %}

    {{ form.as_p }}

    <button type="submit">
        Actualizar
    </button>

</form>

<a href="{% url 'lista_productos' %}">
    Cancelar
</a>

{% endblock %}
```

---

# DELETE — Eliminar producto

Vista:

```python
def eliminar_producto(request, id):

    producto = get_object_or_404(
        Producto,
        id=id
    )

    if request.method == "POST":

        producto.delete()

        return redirect(
            "lista_productos"
        )

    return render(
        request,
        "productos/eliminar.html",
        {
            "producto": producto
        }
    )
```

---

# Flujo de eliminación

Primero:

```text
GET /productos/eliminar/5/
```

Se muestra una confirmación.

Después:

```text
POST /productos/eliminar/5/
```

Se ejecuta:

```python
producto.delete()
```

Luego se redirige al listado.

---

# Template eliminar.html

```html
{% extends "base.html" %}

{% block content %}

<h1>Eliminar producto</h1>

<p>
    ¿Estás seguro de eliminar
    <strong>{{ producto.nombre }}</strong>?
</p>

<form method="post">

    {% csrf_token %}

    <button type="submit">
        Sí, eliminar
    </button>

    <a href="{% url 'lista_productos' %}">
        Cancelar
    </a>

</form>

{% endblock %}
```

---

# URLs

Archivo:

```text
productos/urls.py
```

Contenido:

```python
from django.urls import path
from . import views


urlpatterns = [

    path(
        "",
        views.lista_productos,
        name="lista_productos"
    ),

    path(
        "crear/",
        views.crear_producto,
        name="crear_producto"
    ),

    path(
        "editar/<int:id>/",
        views.editar_producto,
        name="editar_producto"
    ),

    path(
        "eliminar/<int:id>/",
        views.eliminar_producto,
        name="eliminar_producto"
    ),
]
```

---

# Mensajes de Django

Importamos:

```python
from django.contrib import messages
```

Después de crear:

```python
messages.success(
    request,
    "Producto creado correctamente."
)
```

Después de editar:

```python
messages.success(
    request,
    "Producto actualizado correctamente."
)
```

Después de eliminar:

```python
messages.success(
    request,
    "Producto eliminado correctamente."
)
```

En `base.html` mostramos los mensajes:

```html
{% if messages %}

    {% for message in messages %}

        <div>
            {{ message }}
        </div>

    {% endfor %}

{% endif %}
```

---

# Patrón Post/Redirect/Get

Después de un POST no debemos devolver directamente la misma página.

Utilizamos:

```text
POST
 |
 v
Guardar
 |
 v
Redirect
 |
 v
GET
```

Esto evita que al actualizar el navegador se vuelva a enviar el formulario.

---

# Flujo completo del CRUD

```text
Usuario
   |
   | GET /productos/
   v
lista_productos()
   |
   v
Producto.objects.all()
   |
   v
lista.html


Usuario
   |
   | GET /productos/crear/
   v
crear_producto()
   |
   v
ProductoForm()
   |
   v
crear.html


Usuario
   |
   | POST /productos/crear/
   v
crear_producto()
   |
   v
ProductoForm(request.POST)
   |
   v
form.is_valid()
   |
   +--------+
   |        |
  No       Sí
   |        |
   v        v
Errores   form.save()
            |
            v
         redirect()
            |
            v
       lista_productos()
```

---

# Resumen de métodos HTTP

| Operación | GET | POST |
|---|---|---|
| Listar | Sí | No |
| Crear | Mostrar formulario | Guardar |
| Editar | Mostrar formulario | Actualizar |
| Eliminar | Confirmar | Eliminar |

---

# Ejercicio práctico

Implementar el CRUD completo del modelo `Producto`.

## Requisitos

- [ ] Listar productos.
- [ ] Crear productos.
- [ ] Editar productos.
- [ ] Eliminar productos.
- [ ] Crear `ProductoForm`.
- [ ] Utilizar `ModelForm`.
- [ ] Utilizar `GET`.
- [ ] Utilizar `POST`.
- [ ] Validar con `is_valid()`.
- [ ] Guardar con `save()`.
- [ ] Obtener objetos con `get_object_or_404()`.
- [ ] Redirigir con `redirect()`.
- [ ] Incluir `{% csrf_token %}`.
- [ ] Mostrar mensajes de éxito.

---

# Conceptos que deben quedar claros

Al finalizar la clase, el estudiante debe poder explicar:

## ORM

Permite interactuar con la base de datos mediante objetos Python.

## QuerySet

Representa el resultado de una consulta.

Ejemplo:

```python
Producto.objects.all()
```

## ModelForm

Crea formularios basados en modelos.

## GET

Se utiliza principalmente para solicitar y mostrar información.

## POST

Se utiliza para enviar información al servidor.

## form.is_valid()

Ejecuta las validaciones del formulario.

## form.save()

Guarda o actualiza el objeto.

## instance

Permite editar un objeto existente.

## get_object_or_404()

Obtiene un objeto o responde con HTTP 404.

## redirect()

Redirige al usuario después de una operación.

---

# Conexión con la Clase 5

En esta clase trabajamos con:

```text
HTML
  |
Formulario
  |
POST
  |
Vista
  |
ModelForm
  |
Modelo
  |
Base de datos
```

En la siguiente clase, con Django REST Framework, el flujo será:

```text
Cliente
  |
JSON
  |
HTTP POST
  |
View / ViewSet
  |
Serializer
  |
Modelo
  |
Base de datos
```

La diferencia principal será que pasaremos de trabajar con:

```text
HTML + Formularios
```

a trabajar con:

```text
JSON + Serializers + API REST
```

Esta clase deja preparado el proyecto para comenzar la implementación de APIs con Django REST Framework en la Clase 5.
