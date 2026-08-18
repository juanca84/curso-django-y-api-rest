# Clase 4 — Django ORM, Relaciones y PostgreSQL

## 1. Datos generales

- **Curso:** Desarrollo Backend con Django y Django REST Framework
- **Clase:** 4
- **Duración:** 2 horas
- **Tema:** Django ORM, relaciones entre modelos y PostgreSQL
- **Proyecto:** Sistema de inventario
- **App principal:** `productos`

---

## 2. Objetivos de la clase

Al finalizar la clase, el estudiante podrá:

- Comprender cómo funciona el ORM de Django.
- Crear y modificar modelos relacionados.
- Utilizar `ForeignKey`, `ManyToManyField` y `OneToOneField`.
- Crear migraciones y aplicarlas correctamente.
- Consultar información utilizando QuerySets.
- Crear, actualizar, eliminar y filtrar registros.
- Utilizar relaciones entre modelos.
- Configurar PostgreSQL como base de datos del proyecto.
- Verificar que Django se conecta correctamente a PostgreSQL.

---

# 3. Repaso de la Clase 3

Al finalizar la Clase 3 debemos tener:

```text
mi-proyecto/
├── manage.py
├── config/
│   ├── settings.py
│   ├── urls.py
│   └── ...
├── productos/
│   ├── migrations/
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   └── ...
└── templates/
```

La aplicación `productos` ya forma parte del proyecto.

También debemos tener configurada la autenticación básica de Django y el proyecto funcionando.

---

# 4. Introducción al ORM

## ¿Qué es un ORM?

ORM significa:

> Object-Relational Mapping

Es una herramienta que permite trabajar con una base de datos utilizando objetos y clases de Python en lugar de escribir SQL directamente.

Por ejemplo, en SQL:

```sql
SELECT * FROM productos_producto;
```

Con Django ORM:

```python
Producto.objects.all()
```

La idea principal es:

```text
Clase Python
     ↓
Modelo Django
     ↓
Tabla de base de datos
```

Por ejemplo:

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=150)
    precio = models.DecimalField(
        max_digits=10,
        decimal_places=2
    )
```

Django genera una estructura equivalente a una tabla.

---

# 5. Modelos del sistema de inventario

Vamos a ampliar nuestra aplicación `productos`.

El sistema tendrá:

- Categorías
- Productos
- Proveedores

Las relaciones serán:

```text
Categoria
   │
   │ 1:N
   ▼
Producto
   │
   │ N:M
   ▼
Proveedor
```

---

# 6. Crear el modelo Categoria

En:

```text
productos/models.py
```

agregamos:

```python
from django.db import models


class Categoria(models.Model):
    nombre = models.CharField(
        max_length=100,
        unique=True
    )

    descripcion = models.TextField(
        blank=True
    )

    def __str__(self):
        return self.nombre
```

### Explicación

`CharField`:

```python
nombre = models.CharField(max_length=100)
```

representa texto corto.

`TextField`:

```python
descripcion = models.TextField()
```

permite almacenar textos más largos.

`unique=True` significa que no podemos tener dos categorías con el mismo nombre.

---

# 7. Crear el modelo Proveedor

```python
class Proveedor(models.Model):
    nombre = models.CharField(
        max_length=150
    )

    telefono = models.CharField(
        max_length=30,
        blank=True
    )

    email = models.EmailField(
        blank=True
    )

    def __str__(self):
        return self.nombre
```

---

# 8. Crear el modelo Producto

```python
class Producto(models.Model):
    nombre = models.CharField(
        max_length=150
    )

    descripcion = models.TextField(
        blank=True
    )

    precio = models.DecimalField(
        max_digits=10,
        decimal_places=2
    )

    stock = models.PositiveIntegerField(
        default=0
    )

    categoria = models.ForeignKey(
        Categoria,
        on_delete=models.PROTECT,
        related_name="productos"
    )

    proveedores = models.ManyToManyField(
        Proveedor,
        related_name="productos",
        blank=True
    )

    activo = models.BooleanField(
        default=True
    )

    creado_en = models.DateTimeField(
        auto_now_add=True
    )

    actualizado_en = models.DateTimeField(
        auto_now=True
    )

    def __str__(self):
        return self.nombre
```

---

# 9. Relaciones entre modelos

## ForeignKey

Tenemos:

```python
categoria = models.ForeignKey(
    Categoria,
    on_delete=models.PROTECT,
    related_name="productos"
)
```

Esto representa una relación:

```text
Categoria 1 ─────── N Producto
```

Una categoría puede tener muchos productos.

Pero un producto pertenece a una categoría.

### Ejemplo

Categoría:

```text
Electrónica
```

Productos:

```text
Laptop
Mouse
Teclado
Monitor
```

---

# 10. ¿Qué significa on_delete?

Cuando eliminamos un objeto relacionado, Django necesita saber qué hacer con los registros relacionados.

Usaremos:

```python
on_delete=models.PROTECT
```

Esto impide eliminar una categoría si tiene productos asociados.

## CASCADE

```python
on_delete=models.CASCADE
```

Si se elimina el objeto padre, también se eliminan los objetos relacionados.

## SET_NULL

```python
on_delete=models.SET_NULL
```

Permite conservar el objeto relacionado colocando `NULL`.

Para utilizarlo necesitamos:

```python
null=True
```

---

# 11. related_name

Tenemos:

```python
related_name="productos"
```

Esto permite acceder desde una categoría a sus productos:

```python
categoria.productos.all()
```

Ejemplo:

```python
categoria = Categoria.objects.get(id=1)

productos = categoria.productos.all()
```

---

# 12. ManyToManyField

Los proveedores y productos tienen una relación muchos a muchos:

```python
proveedores = models.ManyToManyField(
    Proveedor,
    related_name="productos",
    blank=True
)
```

Esto significa:

```text
Producto N ─────── N Proveedor
```

Un producto puede tener varios proveedores.

Un proveedor puede suministrar varios productos.

Ejemplo:

```text
Laptop
 ├── Proveedor A
 ├── Proveedor B
 └── Proveedor C
```

Y:

```text
Proveedor A
 ├── Laptop
 ├── Mouse
 └── Teclado
```

---

# 13. Migraciones

Después de modificar los modelos ejecutamos:

```bash
python manage.py makemigrations
```

Luego:

```bash
python manage.py migrate
```

Podemos revisar las migraciones:

```bash
python manage.py showmigrations
```

---

# 14. Registrar modelos en el Admin

En:

```text
productos/admin.py
```

agregamos:

```python
from django.contrib import admin

from .models import (
    Categoria,
    Producto,
    Proveedor
)


admin.site.register(Categoria)
admin.site.register(Producto)
admin.site.register(Proveedor)
```

Ahora podemos administrar los datos desde:

```text
/admin/
```

---

# 15. Django Shell

Django proporciona una consola interactiva:

```bash
python manage.py shell
```

Importamos nuestros modelos:

```python
from productos.models import (
    Categoria,
    Producto,
    Proveedor
)
```

---

# 16. Crear registros

## Crear una categoría

```python
categoria = Categoria.objects.create(
    nombre="Electrónica",
    descripcion="Productos electrónicos"
)
```

## Crear un proveedor

```python
proveedor = Proveedor.objects.create(
    nombre="Proveedor ABC",
    telefono="70000000",
    email="ventas@example.com"
)
```

## Crear un producto

```python
producto = Producto.objects.create(
    nombre="Laptop Lenovo",
    descripcion="Laptop para trabajo",
    precio=4500,
    stock=10,
    categoria=categoria
)
```

---

# 17. Agregar relaciones ManyToMany

Después de crear el producto podemos agregar proveedores:

```python
producto.proveedores.add(proveedor)
```

Podemos consultar los proveedores:

```python
producto.proveedores.all()
```

Y desde el proveedor:

```python
proveedor.productos.all()
```

---

# 18. Consultas básicas

## Obtener todos los productos

```python
Producto.objects.all()
```

## Obtener el primer producto

```python
Producto.objects.first()
```

## Obtener el último producto

```python
Producto.objects.last()
```

## Contar productos

```python
Producto.objects.count()
```

---

# 19. Filtrar registros

## Productos activos

```python
Producto.objects.filter(
    activo=True
)
```

## Productos con stock

```python
Producto.objects.filter(
    stock__gt=0
)
```

## Productos con precio mayor a 1000

```python
Producto.objects.filter(
    precio__gt=1000
)
```

## Productos con precio menor o igual a 5000

```python
Producto.objects.filter(
    precio__lte=5000
)
```

---

# 20. get()

Cuando esperamos obtener un único registro:

```python
producto = Producto.objects.get(
    id=1
)
```

Importante:

`get()` genera una excepción si:

- No existe el registro.
- Existe más de un registro que coincide.

Por eso debemos utilizarlo cuando esperamos un único resultado.

---

# 21. get() vs filter()

```python
Producto.objects.get(id=1)
```

devuelve:

```text
Producto
```

Mientras:

```python
Producto.objects.filter(
    activo=True
)
```

devuelve:

```text
QuerySet
```

Un QuerySet representa un conjunto de resultados.

---

# 22. Ordenar resultados

## Ascendente

```python
Producto.objects.order_by(
    "precio"
)
```

## Descendente

```python
Producto.objects.order_by(
    "-precio"
)
```

## Por nombre

```python
Producto.objects.order_by(
    "nombre"
)
```

---

# 23. Buscar por texto

```python
Producto.objects.filter(
    nombre__icontains="laptop"
)
```

`icontains` permite buscar sin importar mayúsculas o minúsculas.

Otros ejemplos:

```python
Producto.objects.filter(
    nombre__startswith="Lap"
)
```

```python
Producto.objects.filter(
    nombre__endswith="Pro"
)
```

---

# 24. Consultar relaciones

## Obtener todos los productos de una categoría

```python
categoria.productos.all()
```

## Obtener la categoría de un producto

```python
producto.categoria
```

## Obtener los proveedores de un producto

```python
producto.proveedores.all()
```

## Obtener los productos de un proveedor

```python
proveedor.productos.all()
```

---

# 25. Actualizar registros

Modificar un producto:

```python
producto.precio = 5000
producto.stock = 8

producto.save()
```

También podemos utilizar:

```python
Producto.objects.filter(
    id=1
).update(
    precio=5000
)
```

---

# 26. Eliminar registros

Eliminar un producto:

```python
producto.delete()
```

Como utilizamos:

```python
on_delete=models.PROTECT
```

Django impedirá eliminar una categoría que tenga productos asociados.

---

# 27. PostgreSQL

Hasta este punto podemos haber trabajado con SQLite.

Ahora vamos a utilizar PostgreSQL.

La arquitectura será:

```text
Django
  │
  │ Django ORM
  ▼
psycopg
  │
  ▼
PostgreSQL
```

Django no se comunica directamente con PostgreSQL.

Utiliza un driver para establecer la conexión.

---

# 28. Instalar el driver de PostgreSQL

En el entorno virtual:

```bash
pip install "psycopg[binary]"
```

Verificamos:

```bash
pip show psycopg
```

También podemos comprobar desde Python:

```bash
python -c "import psycopg; print(psycopg.__version__)"
```

---

# 29. PostgreSQL con Docker

Podemos levantar PostgreSQL utilizando Docker:

```bash
docker run --name postgres-django \
  -e POSTGRES_USER=root \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=django_db \
  -p 5432:5432 \
  -d postgres
```

Verificar:

```bash
docker ps
```

Debemos observar el contenedor:

```text
postgres-django
```

---

# 30. Configurar Django

En:

```text
config/settings.py
```

cambiamos `DATABASES`:

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "django_db",
        "USER": "root",
        "PASSWORD": "password",
        "HOST": "localhost",
        "PORT": "5432",
    }
}
```

---

# 31. Probar la conexión

Ejecutamos:

```bash
python manage.py migrate
```

Si Django consigue conectarse correctamente, aplicará las migraciones sobre PostgreSQL.

También podemos ejecutar:

```bash
python manage.py showmigrations
```

---

# 32. Verificar PostgreSQL

Podemos conectarnos al contenedor:

```bash
docker exec -it postgres-django psql \
  -U root \
  -d django_db
```

Dentro de PostgreSQL:

```sql
\dt
```

Esto mostrará las tablas.

Podemos observar tablas como:

```text
auth_user
auth_group
django_migrations
django_session
productos_categoria
productos_producto
productos_proveedor
```

Salir:

```sql
\q
```

---

# 33. Flujo completo de trabajo

El flujo que debemos enseñar a los estudiantes es:

```text
1. Crear/modificar modelos
        ↓
2. makemigrations
        ↓
3. migrate
        ↓
4. Crear datos
        ↓
5. Consultar mediante ORM
        ↓
6. Modificar/eliminar
        ↓
7. Verificar en PostgreSQL
```

---

# 34. Ejercicio práctico

## Parte 1 — Modelos

Crear los modelos:

- `Categoria`
- `Producto`
- `Proveedor`

Con las relaciones:

```text
Categoria 1:N Producto
Producto N:M Proveedor
```

## Parte 2 — Migraciones

Ejecutar:

```bash
python manage.py makemigrations
python manage.py migrate
```

## Parte 3 — Admin

Registrar los tres modelos en Django Admin.

## Parte 4 — Datos

Crear como mínimo:

- 3 categorías
- 3 proveedores
- 6 productos

## Parte 5 — Relaciones

Cada producto debe tener:

- Una categoría.
- Uno o más proveedores.

## Parte 6 — Consultas

Realizar desde Django Shell:

```python
Producto.objects.all()
```

```python
Producto.objects.filter(
    stock__gt=0
)
```

```python
Producto.objects.filter(
    precio__gt=1000
)
```

```python
Producto.objects.order_by(
    "-precio"
)
```

```python
Categoria.objects.get(
    id=1
).productos.all()
```

## Parte 7 — PostgreSQL

Migrar el proyecto desde SQLite a PostgreSQL y verificar las tablas.

---

# 35. Actividad de evaluación

Responder y demostrar:

### Pregunta 1

¿Qué diferencia existe entre:

```python
get()
```

y:

```python
filter()
```

### Pregunta 2

¿Qué representa una `ForeignKey`?

### Pregunta 3

¿Qué representa un `ManyToManyField`?

### Pregunta 4

¿Qué función cumple:

```python
on_delete=models.PROTECT
```

### Pregunta 5

¿Qué es un QuerySet?

### Pregunta 6

¿Qué función cumple `makemigrations`?

### Pregunta 7

¿Qué función cumple `migrate`?

### Pregunta 8

¿Qué componente permite que Django se comunique con PostgreSQL?

---

# 36. Cierre de la Clase 4

Al finalizar debemos tener:

```text
Django
   │
   ├── Autenticación
   ├── Middleware
   ├── ORM
   ├── Modelos
   ├── Relaciones
   └── PostgreSQL
```

Y nuestro proyecto:

```text
Sistema de Inventario
        │
        ├── Categorias
        ├── Productos
        └── Proveedores
```

Con PostgreSQL como base de datos.

---

# 37. Preparación para la Clase 5

En la siguiente clase podemos utilizar los modelos construidos para comenzar a exponer funcionalidad mediante:

- Views.
- Forms.
- CRUD.
- Validaciones.
- Manejo de errores.
- Templates.
- Introducción a Django REST Framework.

El objetivo será pasar progresivamente de:

```text
Modelo → ORM → View → Template
```

hacia:

```text
Modelo → ORM → API REST → JSON
```

Esto preparará el proyecto para la parte de Django REST Framework.