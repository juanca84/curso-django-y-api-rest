# Clase 3 — Modelado de Datos, Django ORM y PostgreSQL

## Objetivo general

Al finalizar esta clase, el estudiante podrá:

- Crear modelos en Django.
- Definir campos y relaciones entre modelos.
- Comprender y utilizar las migraciones.
- Utilizar el Django ORM.
- Crear, consultar, actualizar y eliminar registros.
- Comprender la diferencia entre un objeto y un `QuerySet`.
- Trabajar con relaciones `ForeignKey`.
- Utilizar el Django Shell.
- Configurar Django para utilizar PostgreSQL.
- Cambiar la base de datos del proyecto de SQLite a PostgreSQL.

---

# 1. Repaso de la clase anterior

Antes de comenzar el nuevo contenido, recordar:

```text
Usuario
   ↓
Autenticación
   ↓
Autorización
   ↓
Permisos
   ↓
Middleware
   ↓
View
   ↓
Response
```

En la clase anterior trabajamos con:

- Usuarios de Django.
- Login y autenticación.
- Permisos.
- Grupos.
- Middleware.
- `request.user`.
- El permiso:

```python
"auth.add_user"
```

Ahora necesitamos aprender dónde se almacenan y cómo se gestionan los datos de nuestra aplicación.

---

# 2. ¿Dónde se almacenan los datos?

Django necesita una base de datos.

Podemos utilizar diferentes motores:

```text
SQLite
PostgreSQL
MySQL
MariaDB
Oracle
```

Para nuestra práctica comenzaremos con SQLite y posteriormente cambiaremos a PostgreSQL.

La arquitectura será:

```text
                 Django
                    │
                    ▼
                  ORM
                    │
             ┌──────┴──────┐
             ▼             ▼
          SQLite       PostgreSQL
```

Una de las ventajas del ORM es que gran parte del código Python que escribimos no depende directamente del motor de base de datos.

---

# 3. ¿Qué es un modelo?

Un modelo Django representa una estructura de datos.

Podemos pensar:

```text
Modelo Django
     ↓
Tabla de base de datos

Campo Django
     ↓
Columna

Objeto Django
     ↓
Registro
```

Por ejemplo:

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=150)
    precio = models.DecimalField(
        max_digits=10,
        decimal_places=2
    )
    stock = models.IntegerField(default=0)
```

Conceptualmente:

```text
Producto
────────────────────
id
nombre
precio
stock
```

---

# 4. Crear la aplicación de productos

Si todavía no existe:

```bash
python manage.py startapp productos
```

La estructura será:

```text
productos/
├── migrations/
├── __init__.py
├── admin.py
├── apps.py
├── models.py
├── tests.py
└── views.py
```

---

# 5. Registrar la aplicación

En `settings.py`:

```python
INSTALLED_APPS = [
    # Aplicaciones de Django
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",

    # Aplicaciones propias
    "productos",
]
```

---

# 6. Crear el modelo Categoria

En:

```text
productos/models.py
```

Escribimos:

```python
from django.db import models


class Categoria(models.Model):
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField(blank=True)

    def __str__(self):
        return self.nombre
```

## `CharField`

Se utiliza para texto de longitud limitada:

```python
nombre = models.CharField(max_length=100)
```

## `TextField`

Se utiliza normalmente para textos más largos:

```python
descripcion = models.TextField()
```

## `blank=True`

Indica que el campo puede quedar vacío en formularios.

---

# 7. Crear el modelo Producto

Añadimos:

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=150)
    descripcion = models.TextField(blank=True)
    precio = models.DecimalField(
        max_digits=10,
        decimal_places=2
    )
    stock = models.IntegerField(default=0)

    def __str__(self):
        return self.nombre
```

Nuestro modelo tiene:

```text
Producto
├── id
├── nombre
├── descripcion
├── precio
└── stock
```

Django creará automáticamente un `id` como clave primaria si no definimos otra.

---

# 8. Relacionar Producto con Categoria

Un producto debe pertenecer a una categoría.

Agregamos:

```python
categoria = models.ForeignKey(
    Categoria,
    on_delete=models.CASCADE,
    related_name="productos"
)
```

El modelo completo queda:

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=150)
    descripcion = models.TextField(blank=True)
    precio = models.DecimalField(
        max_digits=10,
        decimal_places=2
    )
    stock = models.IntegerField(default=0)

    categoria = models.ForeignKey(
        Categoria,
        on_delete=models.CASCADE,
        related_name="productos"
    )

    def __str__(self):
        return self.nombre
```

---

# 9. ¿Qué relación estamos utilizando?

Tenemos:

```text
Categoria 1 ─────────── N Producto
```

Una categoría puede tener muchos productos.

Cada producto pertenece a una categoría.

Esto corresponde a:

```python
models.ForeignKey()
```

Es una relación:

```text
Uno a Muchos
1 : N
```

---

# 10. ¿Qué significa `on_delete`?

Tenemos:

```python
on_delete=models.CASCADE
```

Supongamos:

```text
Categoria: Electrónica

    │
    ├── Laptop
    ├── Monitor
    └── Teclado
```

Si eliminamos la categoría:

```text
Eliminar Electrónica
        ↓
Laptop
Monitor
Teclado
```

Con `CASCADE`, también se eliminan los registros relacionados.

Existen otras opciones:

```python
models.PROTECT
models.SET_NULL
models.SET_DEFAULT
models.CASCADE
```

En nuestra práctica utilizaremos:

```python
models.CASCADE
```

---

# 11. Migraciones

Después de modificar los modelos debemos informar a Django.

Primero:

```bash
python manage.py makemigrations
```

Después:

```bash
python manage.py migrate
```

La idea fundamental:

```text
models.py
    │
    ▼
makemigrations
    │
    ▼
Archivo de migración
    │
    ▼
migrate
    │
    ▼
Base de datos
```

---

# 12. Diferencia entre `makemigrations` y `migrate`

## `makemigrations`

Detecta los cambios realizados en los modelos y crea archivos de migración.

```bash
python manage.py makemigrations
```

## `migrate`

Aplica esas migraciones a la base de datos.

```bash
python manage.py migrate
```

Podemos resumirlo:

```text
makemigrations = preparar cambios

migrate = aplicar cambios
```

---

# 13. ¿Qué es una migración?

Django crea archivos dentro de:

```text
productos/migrations/
```

Por ejemplo:

```text
productos/
└── migrations/
    ├── __init__.py
    └── 0001_initial.py
```

Una migración representa un cambio en la estructura de la base de datos.

Por ejemplo:

```text
Crear tabla Categoria
Crear tabla Producto
Crear ForeignKey
```

---

# 14. Ver las migraciones

Podemos ejecutar:

```bash
python manage.py showmigrations
```

Esto nos permite comprobar qué migraciones están aplicadas.

---

# 15. Django Shell

Ahora trabajaremos con los objetos directamente.

Ejecutamos:

```bash
python manage.py shell
```

Importamos los modelos:

```python
from productos.models import Categoria, Producto
```

---

# 16. El ORM

El ORM permite trabajar con la base de datos utilizando objetos Python.

Por ejemplo:

```python
Categoria.objects.all()
```

Estamos utilizando:

```text
Modelo.objects.operacion()
```

`objects` es el manager que nos permite realizar consultas.

---

# 17. Obtener todos los registros

```python
Categoria.objects.all()
```

Si todavía no tenemos categorías:

```text
<QuerySet []>
```

Esto significa que tenemos un `QuerySet` vacío.

---

# 18. Crear una categoría

```python
categoria = Categoria.objects.create(
    nombre="Electrónica",
    descripcion="Productos electrónicos"
)
```

Podemos revisar:

```python
categoria
```

Y:

```python
Categoria.objects.all()
```

---

# 19. Crear varias categorías

```python
Categoria.objects.create(
    nombre="Oficina",
    descripcion="Productos para oficina"
)

Categoria.objects.create(
    nombre="Hogar",
    descripcion="Productos para el hogar"
)
```

Comprobamos:

```python
Categoria.objects.all()
```

---

# 20. Crear un producto

```python
producto = Producto.objects.create(
    nombre="Laptop Lenovo",
    descripcion="Laptop para oficina",
    precio=4500,
    stock=10,
    categoria=categoria
)
```

---

# 21. Consultar todos los productos

```python
Producto.objects.all()
```

Resultado conceptual:

```text
<QuerySet [<Producto: Laptop Lenovo>]>
```

Aquí debemos remarcar algo:

```text
Producto.objects.all()
            ↓
        QuerySet
```

No estamos obteniendo directamente un objeto `Producto`.

---

# 22. `get()` y `filter()`

## `get()`

Busca un único objeto:

```python
Producto.objects.get(id=1)
```

Conceptualmente:

```text
get()
  ↓
Objeto
```

## `filter()`

Busca cero, uno o muchos registros:

```python
Producto.objects.filter(stock=10)
```

Conceptualmente:

```text
filter()
   ↓
QuerySet
```

---

# 23. Diferencia entre objeto y QuerySet

```text
Producto.objects.get(id=1)
            ↓
        Producto
```

Mientras:

```text
Producto.objects.filter(stock=10)
            ↓
        QuerySet
```

Podemos pensar:

```text
Objeto
=
un registro

QuerySet
=
conjunto de registros
```

---

# 24. Consultas con filtros

Buscar productos con stock mayor a 5:

```python
Producto.objects.filter(
    stock__gt=5
)
```

Menor que:

```python
Producto.objects.filter(
    stock__lt=5
)
```

Mayor o igual:

```python
Producto.objects.filter(
    stock__gte=5
)
```

Menor o igual:

```python
Producto.objects.filter(
    stock__lte=5
)
```

---

# 25. Consultar por precio

Productos con precio menor a 1000:

```python
Producto.objects.filter(
    precio__lt=1000
)
```

Productos con precio mayor a 3000:

```python
Producto.objects.filter(
    precio__gt=3000
)
```

---

# 26. Actualizar un objeto

Obtenemos un producto:

```python
producto = Producto.objects.get(id=1)
```

Modificamos:

```python
producto.precio = 4800
```

Guardamos:

```python
producto.save()
```

Comprobamos:

```python
producto.precio
```

---

# 27. Eliminar un objeto

```python
producto = Producto.objects.get(id=1)
```

Luego:

```python
producto.delete()
```

Comprobamos:

```python
Producto.objects.all()
```

---

# 28. Navegar las relaciones

Si tenemos:

```python
producto.categoria
```

Podemos obtener la categoría del producto.

Por ejemplo:

```python
producto.categoria.nombre
```

Resultado:

```text
Electrónica
```

Gracias a:

```python
related_name="productos"
```

podemos obtener los productos de una categoría:

```python
categoria.productos.all()
```

---

# 29. Consultas sobre relaciones

Podemos consultar productos cuya categoría sea "Electrónica":

```python
Producto.objects.filter(
    categoria__nombre="Electrónica"
)
```

Aquí aparece una característica importante del ORM:

```text
categoria__nombre
```

Los dos guiones bajos:

```text
__
```

permiten navegar relaciones y realizar búsquedas.

---

# 30. Ejercicio práctico

Crear las siguientes categorías:

```text
Electrónica
Oficina
Hogar
```

Crear al menos cinco productos.

Ejemplo:

```text
Laptop Lenovo
Monitor Samsung
Teclado Logitech
Escritorio
Silla de oficina
```

Cada producto debe tener:

- Nombre.
- Descripción.
- Precio.
- Stock.
- Categoría.

---

# 31. Consultas del ejercicio

## Todos los productos

```python
Producto.objects.all()
```

## Productos con stock mayor a 5

```python
Producto.objects.filter(stock__gt=5)
```

## Productos con precio menor a 1000

```python
Producto.objects.filter(precio__lt=1000)
```

## Productos de Electrónica

```python
Producto.objects.filter(
    categoria__nombre="Electrónica"
)
```

## Productos de una categoría

```python
categoria = Categoria.objects.get(
    nombre="Oficina"
)

categoria.productos.all()
```

---

# 32. Cambiar de SQLite a PostgreSQL

Hasta este momento trabajamos con:

```text
Django
   ↓
ORM
   ↓
SQLite
```

Ahora cambiaremos a:

```text
Django
   ↓
ORM
   ↓
PostgreSQL
```

La gran ventaja es que nuestro código ORM prácticamente no cambia.

---

# 33. PostgreSQL con Docker

Para la práctica podemos utilizar PostgreSQL mediante Docker.

Crear el contenedor:

```bash
docker run --name postgres-django \
  -e POSTGRES_USER=root \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=django_db \
  -p 5432:5432 \
  -d postgres:17
```

Comprobar:

```bash
docker ps
```

---

# 34. PostgreSQL con Docker Compose

También podemos utilizar un archivo:

```text
docker-compose.yml
```

Contenido:

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-django
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: password
      POSTGRES_DB: django_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Levantar PostgreSQL:

```bash
docker compose up -d
```

Detener:

```bash
docker compose down
```

---

# 35. Instalar el driver de PostgreSQL

Dentro del entorno virtual:

```bash
pip install "psycopg[binary]" 
```

Podemos comprobar:

```bash
pip show psycopg
```

---

# 36. Configurar Django

Inicialmente tenemos algo similar a:

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

Lo cambiamos por:

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

# 37. Aplicar las migraciones en PostgreSQL

Ejecutamos:

```bash
python manage.py migrate
```

Ahora Django creará las tablas en PostgreSQL.

Podemos verificar nuevamente:

```bash
python manage.py shell
```

Y:

```python
from productos.models import Categoria, Producto

Categoria.objects.all()
```

---

# 38. Importante: SQLite y PostgreSQL son bases diferentes

Cuando cambiamos la configuración:

```text
SQLite
   ↓
PostgreSQL
```

No estamos convirtiendo automáticamente la base de datos.

Tenemos dos bases de datos diferentes.

```text
SQLite
└── db.sqlite3

PostgreSQL
└── django_db
```

Las migraciones crean la estructura en PostgreSQL.

Los datos que estaban en SQLite no se copian automáticamente.

---

# 39. Arquitectura final

Al finalizar:

```text
                     DJANGO
                        │
                        ▼
                       ORM
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
       Modelos Django          QuerySets
            │                       │
            └───────────┬───────────┘
                        ▼
                    PostgreSQL
```

Y el flujo de las migraciones:

```text
models.py
    │
    ▼
makemigrations
    │
    ▼
migrations/
    │
    ▼
migrate
    │
    ▼
PostgreSQL
```

---

# 40. Conceptos clave de la clase

Los estudiantes deben poder explicar:

### Modelo

Representa una estructura de datos y normalmente una tabla.

### Campo

Representa una columna.

### Objeto

Representa un registro.

### QuerySet

Representa un conjunto de resultados de una consulta.

### ORM

Permite trabajar con la base de datos utilizando objetos y código Python.

### Migración

Representa cambios en el esquema de la base de datos.

### `makemigrations`

Genera las migraciones.

### `migrate`

Aplica las migraciones.

### `ForeignKey`

Representa una relación uno a muchos.

---

# 41. Resumen visual

```text
               MODELO
                  │
                  ▼
             MIGRACIONES
                  │
         ┌────────┴────────┐
         ▼                 ▼
       SQLite          PostgreSQL
         │                 │
         └────────┬────────┘
                  ▼
                 ORM
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      create     get      filter
        │         │         │
        ▼         ▼         ▼
      Crear    Obtener    Buscar
```

---

# 42. Preparación para la siguiente clase

En esta clase aprendimos a:

```text
Crear modelos
     ↓
Crear relaciones
     ↓
Generar migraciones
     ↓
Aplicar migraciones
     ↓
Consultar con ORM
     ↓
Trabajar con QuerySets
     ↓
Cambiar a PostgreSQL
```

El siguiente paso será utilizar estos modelos desde nuestras vistas.

```text
Clase 3
Modelos + ORM + PostgreSQL
           ↓
Clase 4
Vistas + Formularios + CRUD
           ↓
Clase 5
Django REST Framework + APIs
```

## Idea principal para la pizarra

```text
MODELO
  ↓
ORM
  ↓
QUERYSET
  ↓
BASE DE DATOS
```

> Django nos permite trabajar con los datos mediante Python, mientras el ORM se encarga de traducir nuestras operaciones a consultas para la base de datos.
