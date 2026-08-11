# INSTALACIÓN

# Windows

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

# macOS / Linux

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

# ¿Cómo sabemos que el entorno virtual está activo?

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

# CREANDO NUESTRO PRIMER PROYECTO

# Crear la carpeta

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

# EJECUTANDO DJANGO

# Iniciar el servidor

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

# Crear nuestra primera app

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


# Ejercicio práctico

Crear un proyecto llamado:

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