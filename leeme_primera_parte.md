# Guía paso a paso (Primera parte) — Proyecto **Restaurante** (Django, Python, VS Code)

---

# 1 — Crear la carpeta del proyecto

Abre tu terminal (PowerShell / CMD / Terminal) y ejecuta:

```bash
# desde la carpeta donde quieras crear el proyecto
mkdir UIII_Restaurante_0344
cd UIII_Restaurante_0344
```

---

# 2 — Abrir VS Code sobre la carpeta

Si tienes `code` en PATH:

```bash
code .
```

(esto abre VS Code en `UIII_Restaurante_0344`).

También puedes abrir VS Code y usar **File → Open Folder...** y seleccionar `UIII_Restaurante_0344`.

---

# 3 — Abrir terminal en VS Code

En VS Code: menú **Terminal → New Terminal** (o `Ctrl+` ` ` ` ``) — te abre la terminal en la raíz del proyecto.

---

# 4 — Crear carpeta/entorno virtual `.venv` desde la terminal de VS Code

(Comando multiplataforma — usa la versión `python` correcta)

```bash
# Windows / macOS / Linux (asegúrate que 'python' apunta a Python 3.x)
python -m venv .venv
```

Esto crea la carpeta `.venv` dentro de `UIII_Restaurante_0344`.

---

# 5 — Activar el entorno virtual

* **Windows PowerShell**

  ```powershell
  .\.venv\Scripts\Activate.ps1
  ```

  Si te bloquea por política (ExecutionPolicy), puedes ejecutar en PowerShell como administrador:

  ```powershell
  Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
  ```

  y luego activar.

* **Windows CMD**

  ```cmd
  .\.venv\Scripts\activate
  ```

* **macOS / Linux**

  ```bash
  source .venv/bin/activate
  ```

Cuando esté activo verás `(.venv)` al inicio de la línea de tu terminal.

---

# 6 — Activar intérprete de Python en VS Code

1. En VS Code: `Ctrl+Shift+P` → escribe **Python: Select Interpreter**.
2. Selecciona la ruta al intérprete dentro de tu venv, p. ej. `UIII_Restaurante_0344\.venv\Scripts\python` (Windows) o `.venv/bin/python` (mac/linux).
3. Opcional: reinicia la ventana si pide recargar.

---

# 7 — Instalar Django

Con venv activado:

```bash
pip install --upgrade pip
pip install django
```

Verifica:

```bash
python -m django --version
```

---

# 8 — Crear proyecto **backend_Restaurante** SIN duplicar carpeta

Para evitar que Django cree una carpeta extra `backend_Restaurante/backend_Restaurante`, utiliza el punto `.` al crear el proyecto **dentro** de la carpeta actual.

Desde `UIII_Restaurante_0344`:

```bash
django-admin startproject backend_Restaurante .
```

> El `.` indica "crear el proyecto aquí" (no crear subcarpeta adicional). Ahora verás `manage.py` y el paquete `backend_Restaurante/`.

---

# 9 — Ejecutar servidor en el puerto **8044**

Antes de ejecutar, crear app y migraciones. Pero para probar:

```bash
python manage.py runserver 8044
```

Abre en el navegador: `http://127.0.0.1:8044/` o `http://localhost:8044/`.

(Si VS Code te pide permisos de firewall acepta para localhost).

---

# 10 — Copiar y pegar el link en el navegador

Abre tu navegador y pega:
`http://127.0.0.1:8044/` (o `http://localhost:8044/`)

---

# 11 — Crear aplicación `app_Restaurante`

Con venv activado y en la raíz del proyecto:

```bash
python manage.py startapp app_Restaurante
```

Esto crea la carpeta `app_Restaurante` con sus archivos.

---

# 12 — `models.py`

Copia/pega el siguiente contenido en `app_Restaurante/models.py`. Incluye los tres modelos (Platillo, Pedido, DetallePedido) tal como los diste — trabajaremos CRUD sólo para **Platillo** por ahora.

```python
from django.db import models  

# ==========================================
# MODELO: PLATILLO
# ==========================================
class Platillo(models.Model):
    nombre = models.CharField(max_length=100, unique=True)
    descripcion = models.TextField(blank=True, null=True)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    categoria = models.CharField(max_length=50)
    tiempopreparacion = models.IntegerField(verbose_name="Tiempo (min)")
    fecha_creacion = models.DateField(auto_now_add=True)
    activo = models.BooleanField(default=True)
    popularidad = models.PositiveIntegerField(default=0)
    icono = models.CharField(max_length=100, blank=True, null=True)

    def __str__(self):
        return self.nombre

# ==========================================
# MODELO: PEDIDO
# ==========================================
class Pedido(models.Model):
    cliente = models.CharField(max_length=100)
    fechahora = models.DateTimeField(auto_now_add=True)
    estado = models.CharField(max_length=30, default='Pendiente')
    direccionentrega = models.CharField(max_length=200)
    total = models.DecimalField(max_digits=10, decimal_places=2, default=0.00)
    notas = models.TextField(blank=True, null=True)

    class Meta:
        ordering = ['-fechahora']

    def __str__(self):
        return f"Pedido #{self.id} de {self.cliente}"

# ==========================================
# MODELO: DETALLEPEDIDO
# ==========================================
class DetallePedido(models.Model):
    pedido = models.ForeignKey(Pedido, on_delete=models.CASCADE, related_name='detalles')
    platillo = models.ForeignKey(Platillo, on_delete=models.PROTECT, related_name='detalles_pedido')
    cantidad = models.IntegerField()
    preciounitario = models.DecimalField(max_digits=10, decimal_places=2)
    subtotal = models.DecimalField(max_digits=10, decimal_places=2)
    notas = models.CharField(max_length=255, blank=True, null=True)

    class Meta:
        unique_together = ('pedido', 'platillo')

    def __str__(self):
        return f"Detalle {self.id} - Platillo: {self.platillo.nombre}"
```

---

# 12.5 — Procedimiento para realizar migraciones (`makemigrations` y `migrate`)

Primero registra la app en settings (ver paso 25). Después:

```bash
# crear migraciones para app_Restaurante
python manage.py makemigrations app_Restaurante

# aplicar migraciones a la base de datos
python manage.py migrate
```

---

# 13 — Primero trabajamos con el MODELO: **PLATILLO**

Las vistas, urls y templates que te doy a continuación enfocan CRUD en `Platillo` únicamente. `Pedido` y `DetallePedido` quedan en models.py para el futuro.

---

# 14 — `views.py` de `app_Restaurante`

Reemplaza el contenido de `app_Restaurante/views.py` por este (funcional, sin `forms.py`, sin validaciones):

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Platillo

# Página de inicio del sistema
def inicio_restaurante(request):
    total_platillos = Platillo.objects.count()
    platillos = Platillo.objects.all().order_by('-fecha_creacion')[:5]
    return render(request, 'inicio.html', {'total_platillos': total_platillos, 'platillos': platillos})

# Mostrar formulario para agregar platillo
def agregar_platillo(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion')
        precio = request.POST.get('precio') or 0
        categoria = request.POST.get('categoria')
        tiempopreparacion = request.POST.get('tiempopreparacion') or 0
        activo = True if request.POST.get('activo') == 'on' else False
        popularidad = request.POST.get('popularidad') or 0
        icono = request.POST.get('icono')

        Platillo.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            precio=precio,
            categoria=categoria,
            tiempopreparacion=tiempopreparacion,
            activo=activo,
            popularidad=popularidad,
            icono=icono
        )
        return redirect('ver_platillos')

    return render(request, 'platillo/agregar_platillo.html')

# Ver todos los platillos
def ver_platillos(request):
    platillos = Platillo.objects.all().order_by('nombre')
    return render(request, 'platillo/ver_platillos.html', {'platillos': platillos})

# Mostrar formulario con datos para actualizar
def actualizar_platillo(request, platillo_id):
    platillo = get_object_or_404(Platillo, id=platillo_id)
    return render(request, 'platillo/actualizar_platillo.html', {'platillo': platillo})

# Procesa la actualización (POST)
def realizar_actualizacion_platillo(request, platillo_id):
    platillo = get_object_or_404(Platillo, id=platillo_id)
    if request.method == 'POST':
        platillo.nombre = request.POST.get('nombre')
        platillo.descripcion = request.POST.get('descripcion')
        platillo.precio = request.POST.get('precio') or 0
        platillo.categoria = request.POST.get('categoria')
        platillo.tiempopreparacion = request.POST.get('tiempopreparacion') or 0
        platillo.activo = True if request.POST.get('activo') == 'on' else False
        platillo.popularidad = request.POST.get('popularidad') or 0
        platillo.icono = request.POST.get('icono')
        platillo.save()
        return redirect('ver_platillos')
    return redirect('ver_platillos')

# Confirmación y borrado
def borrar_platillo(request, platillo_id):
    platillo = get_object_or_404(Platillo, id=platillo_id)
    if request.method == 'POST':
        platillo.delete()
        return redirect('ver_platillos')
    return render(request, 'platillo/borrar_platillo.html', {'platillo': platillo})
```

---

# 15 — Crear carpeta `templates` dentro de `app_Restaurante`

Ruta: `app_Restaurante/templates/`

Dentro de ella habrá:

* `base.html`
* `header.html`
* `navbar.html`
* `footer.html`
* `inicio.html`
* subcarpeta `platillo/` con los html del CRUD.

---

# 16 — Archivos HTML principales

A continuación plantillas simples y con **Bootstrap CDN**.

Crea `app_Restaurante/templates/base.html`:

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Restaurante - Administración{% endblock %}</title>

    <!-- Bootstrap CSS (CDN) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- Bootstrap Icons -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css" rel="stylesheet">

    <style>
      body { padding-top: 70px; background-color: #f8fafc; }
      .card-soft { border-radius: 12px; box-shadow: 0 6px 16px rgba(18,24,40,0.06); }
      footer.fixed-footer { position: fixed; left:0; bottom:0; width:100%; background:#ffffffcc; padding:10px 0; border-top:1px solid #eee; }
    </style>

    {% block extra_head %}{% endblock %}
  </head>
  <body>
    {% include "navbar.html" %}
    <div class="container my-4">
      {% block content %}{% endblock %}
    </div>

    {% include "footer.html" %}

    <!-- Bootstrap JS Bundle -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
  </body>
</html>
```

Crea `app_Restaurante/templates/navbar.html`:

```html
<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm fixed-top">
  <div class="container">
    <a class="navbar-brand d-flex align-items-center" href="{% url 'inicio' %}">
      <i class="bi bi-shop" style="font-size:1.4rem; margin-right:8px;"></i>
      <strong>Sistema de Administración Restaurante</strong>
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMain">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navMain">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio' %}">Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Platillos</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_platillo' %}">Agregar platillo</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_platillos' %}">Ver platillos</a></li>
            <li><a class="dropdown-item" href="#">Actualizar platillo</a></li>
            <li><a class="dropdown-item" href="#">Borrar platillo</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Pedidos</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar pedido</a></li>
            <li><a class="dropdown-item" href="#">Ver pedidos</a></li>
            <li><a class="dropdown-item" href="#">Actualizar pedido</a></li>
            <li><a class="dropdown-item" href="#">Borrar pedido</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Detalle Pedido</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar detalle</a></li>
            <li><a class="dropdown-item" href="#">Ver detalles</a></li>
            <li><a class="dropdown-item" href="#">Actualizar detalle</a></li>
            <li><a class="dropdown-item" href="#">Borrar detalle</a></li>
          </ul>
        </li>

      </ul>
    </div>
  </div>
</nav>
```

> Nota: los iconos están en las opciones principales, no se usan en los submenús.

Crea `app_Restaurante/templates/footer.html`:

```html
<footer class="fixed-footer text-center">
  <div class="container">
    <small>
      &copy; {% now "Y" %} Restaurante — Todos los derechos reservados.
      &nbsp;|&nbsp; Creado por Ing. Eliseo Nava, Cbtis 128
    </small>
  </div>
</footer>
```

Crea `app_Restaurante/templates/inicio.html`:

```html
{% extends "base.html" %}
{% block title %}Inicio - Restaurante{% endblock %}
{% block content %}
  <div class="row">
    <div class="col-md-8">
      <div class="card card-soft p-3">
        <h4>Bienvenido al Sistema de Administración Restaurante</h4>
        <p>Gestiona platillos, pedidos y detalles. Información del sistema:</p>
        <ul>
          <li>Total de platillos: <strong>{{ total_platillos }}</strong></li>
        </ul>
      </div>
      <div class="mt-3">
        <img src="https://images.unsplash.com/photo-1555992336-03a23c8f4c14?q=80&w=1200&auto=format&fit=crop&s=placeholder" alt="Restaurante" class="img-fluid rounded">
      </div>
    </div>
    <div class="col-md-4">
      <div class="card card-soft p-3">
        <h6>Últimos platillos</h6>
        <ul class="list-group">
          {% for p in platillos %}
            <li class="list-group-item">{{ p.nombre }} <small class="text-muted d-block">Creado: {{ p.fecha_creacion }}</small></li>
          {% empty %}
            <li class="list-group-item">No hay platillos aún.</li>
          {% endfor %}
        </ul>
      </div>
    </div>
  </div>
{% endblock %}
```

> La imagen en `inicio.html` es una URL pública de ejemplo (Unsplash). Puedes reemplazarla.

---

# 21 — Crear subcarpeta `platillo` dentro de templates

Ruta: `app_Restaurante/templates/platillo/`

---

# 22 — Templates CRUD para `platillo`

Crea `app_Restaurante/templates/platillo/agregar_platillo.html`:

```html
{% extends "base.html" %}
{% block title %}Agregar Platillo{% endblock %}
{% block content %}
<div class="card p-4 card-soft">
  <h4>Agregar Platillo</h4>
  <form method="post">
    {% csrf_token %}
    <div class="mb-3">
      <label class="form-label">Nombre</label>
      <input class="form-control" name="nombre" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Descripción</label>
      <textarea class="form-control" name="descripcion"></textarea>
    </div>
    <div class="mb-3">
      <label class="form-label">Precio</label>
      <input class="form-control" name="precio" type="number" step="0.01">
    </div>
    <div class="mb-3">
      <label class="form-label">Categoría</label>
      <input class="form-control" name="categoria">
    </div>
    <div class="mb-3">
      <label class="form-label">Tiempo de preparación (min)</label>
      <input class="form-control" name="tiempopreparacion" type="number">
    </div>
    <div class="mb-3 form-check">
      <input type="checkbox" class="form-check-input" name="activo" checked>
      <label class="form-check-label">Activo</label>
    </div>
    <div class="mb-3">
      <label class="form-label">Popularidad (número)</label>
      <input class="form-control" name="popularidad" type="number" min="0">
    </div>
    <div class="mb-3">
      <label class="form-label">Icono (texto)</label>
      <input class="form-control" name="icono">
    </div>
    <button class="btn btn-primary">Guardar</button>
    <a href="{% url 'ver_platillos' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}
```

Crea `app_Restaurante/templates/platillo/ver_platillos.html`:

```html
{% extends "base.html" %}
{% block title %}Ver Platillos{% endblock %}
{% block content %}
<div class="card card-soft p-3">
  <div class="d-flex justify-content-between align-items-center mb-3">
    <h4>Platillos</h4>
    <a class="btn btn-success" href="{% url 'agregar_platillo' %}"><i class="bi bi-plus"></i> Nuevo Platillo</a>
  </div>

  <table class="table table-hover align-middle">
    <thead>
      <tr>
        <th>Nombre</th>
        <th>Categoría</th>
        <th>Precio</th>
        <th>Activo</th>
        <th>Creado</th>
        <th>Acciones</th>
      </tr>
    </thead>
    <tbody>
      {% for p in platillos %}
      <tr>
        <td>{{ p.nombre }}</td>
        <td>{{ p.categoria }}</td>
        <td>{{ p.precio }}</td>
        <td>{% if p.activo %}Sí{% else %}No{% endif %}</td>
        <td>{{ p.fecha_creacion }}</td>
        <td>
          <a class="btn btn-sm btn-info" href="{% url 'actualizar_platillo' p.id %}">Editar</a>
          <a class="btn btn-sm btn-danger" href="{% url 'borrar_platillo' p.id %}">Borrar</a>
        </td>
      </tr>
      {% empty %}
      <tr><td colspan="6">No existen platillos.</td></tr>
      {% endfor %}
    </tbody>
  </table>
</div>
{% endblock %}
```

Crea `app_Restaurante/templates/platillo/actualizar_platillo.html`:

```html
{% extends "base.html" %}
{% block title %}Actualizar Platillo{% endblock %}
{% block content %}
<div class="card p-4 card-soft">
  <h4>Actualizar Platillo</h4>
  <form method="post" action="{% url 'realizar_actualizacion_platillo' platillo.id %}">
    {% csrf_token %}
    <div class="mb-3">
      <label class="form-label">Nombre</label>
      <input class="form-control" name="nombre" value="{{ platillo.nombre }}" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Descripción</label>
      <textarea class="form-control" name="descripcion">{{ platillo.descripcion }}</textarea>
    </div>
    <div class="mb-3">
      <label class="form-label">Precio</label>
      <input class="form-control" name="precio" type="number" step="0.01" value="{{ platillo.precio }}">
    </div>
    <div class="mb-3">
      <label class="form-label">Categoría</label>
      <input class="form-control" name="categoria" value="{{ platillo.categoria }}">
    </div>
    <div class="mb-3">
      <label class="form-label">Tiempo de preparación (min)</label>
      <input class="form-control" name="tiempopreparacion" type="number" value="{{ platillo.tiempopreparacion }}">
    </div>
    <div class="mb-3 form-check">
      <input type="checkbox" class="form-check-input" name="activo" {% if platillo.activo %}checked{% endif %}>
      <label class="form-check-label">Activo</label>
    </div>
    <div class="mb-3">
      <label class="form-label">Popularidad</label>
      <input class="form-control" name="popularidad" type="number" min="0" value="{{ platillo.popularidad }}">
    </div>
    <div class="mb-3">
      <label class="form-label">Icono</label>
      <input class="form-control" name="icono" value="{{ platillo.icono }}">
    </div>
    <button class="btn btn-primary">Actualizar</button>
    <a href="{% url 'ver_platillos' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}
```

Crea `app_Restaurante/templates/platillo/borrar_platillo.html`:

```html
{% extends "base.html" %}
{% block title %}Borrar Platillo{% endblock %}
{% block content %}
<div class="card p-4 card-soft">
  <h4>Confirmar borrado</h4>
  <p>¿Deseas eliminar el platillo <strong>{{ platillo.nombre }}</strong>?</p>
  <form method="post">
    {% csrf_token %}
    <button class="btn btn-danger">Sí, eliminar</button>
    <a class="btn btn-secondary" href="{% url 'ver_platillos' %}">No, cancelar</a>
  </form>
</div>
{% endblock %}
```

---

# 23 — No utilizar `forms.py`

Como pediste, el CRUD usa `request.POST` directo.

---

# 24 — `urls.py` en `app_Restaurante`

Crea `app_Restaurante/urls.py` con este contenido:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_restaurante, name='inicio'),
    path('platillo/agregar/', views.agregar_platillo, name='agregar_platillo'),
    path('platillo/ver/', views.ver_platillos, name='ver_platillos'),
    path('platillo/actualizar/<int:platillo_id>/', views.actualizar_platillo, name='actualizar_platillo'),
    path('platillo/actualizar/realizar/<int:platillo_id>/', views.realizar_actualizacion_platillo, name='realizar_actualizacion_platillo'),
    path('platillo/borrar/<int:platillo_id>/', views.borrar_platillo, name='borrar_platillo'),
]
```

---

# 25 — Agregar `app_Restaurante` en `settings.py` de `backend_Restaurante`

Edita `backend_Restaurante/settings.py`, en `INSTALLED_APPS` añade `'app_Restaurante',` por ejemplo:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # apps locales
    'app_Restaurante',
]
```

Asegúrate que en `TEMPLATES` `APP_DIRS: True` está activado (por defecto).

---

# 26 — Configurar `urls.py` de `backend_Restaurante` para enlazar con `app_Restaurante`

Edita `backend_Restaurante/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Restaurante.urls')),  # rutas principales de la app
]
```

---

# 27 — Registrar modelos en `admin.py` y volver a migrar/crear superuser

En `app_Restaurante/admin.py`:

```python
from django.contrib import admin
from .models import Platillo, Pedido, DetallePedido

@admin.register(Platillo)
class PlatilloAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'categoria', 'precio', 'activo', 'popularidad', 'fecha_creacion')
    search_fields = ('nombre', 'categoria')
    list_filter = ('activo', 'categoria')

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    list_display = ('id', 'cliente', 'fechahora', 'estado', 'total')
    search_fields = ('cliente',)
    list_filter = ('estado',)

@admin.register(DetallePedido)
class DetallePedidoAdmin(admin.ModelAdmin):
    list_display = ('id', 'pedido', 'platillo', 'cantidad', 'preciounitario', 'subtotal')
```

Luego:

```bash
# crear migraciones (si aún no lo hiciste después de añadir app y models)
python manage.py makemigrations
python manage.py migrate

# crear superusuario para admin
python manage.py createsuperuser
```

---

# 27 (segunda aparición) — Por ahora solo trabajar con “PLATILLO”

Las vistas, urls y templates anteriores se centran sólo en administrar `Platillo`. `Pedido` y `DetallePedido` están en models.py pero **no** tienen vistas ni templates (pendiente).

---

# 28 — Estética: colores suaves y modernos

Las plantillas usan Bootstrap, sombras suaves (`card-soft`) y tipografía predeterminada de Bootstrap para mantener diseño limpio y moderno. Puedes personalizar `style` en `base.html`.

---

# 29 — Estructura completa inicial de carpetas y archivos

Ejemplo de árbol principal (después de aplicar lo anterior):

```
UIII_Restaurante_0344/
├─ .venv/
├─ manage.py
├─ backend_Restaurante/
│  ├─ __init__.py
│  ├─ settings.py
│  ├─ urls.py
│  └─ wsgi.py
├─ app_Restaurante/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  └─ platillo/
│  │     ├─ agregar_platillo.html
│  │     ├─ ver_platillos.html
│  │     ├─ actualizar_platillo.html
│  │     └─ borrar_platillo.html
│  ├─ __init__.py
│  ├─ admin.py
│  ├─ apps.py
│  ├─ models.py
│  ├─ tests.py
│  ├─ urls.py
│  └─ views.py
```

---

# 30 — Proyecto totalmente funcional (pasos finales)

1. Asegúrate que `app_Restaurante` está en `INSTALLED_APPS`.

2. Ejecuta migraciones:

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

3. Crea superusuario (opcional para admin):

   ```bash
   python manage.py createsuperuser
   ```

4. Ejecuta servidor en puerto **8044**:

   ```bash
   python manage.py runserver 8044
   ```

5. Abre en navegador: `http://127.0.0.1:8044/` — deberías ver la página de inicio.

6. Accede a `Platillos` → Ver/Agregar/Editar/Borrar desde la barra de navegación.

---

# 31 — Finalmente ejecutar servidor en el puerto 8044

(Ya incluido arriba; repito el comando final)

```bash
python manage.py runserver 8044
```

---

## Notas rápidas, recomendaciones y consideraciones

* No validé entradas (según tu instrucción). Si en el futuro quieres validación, podemos añadir `forms.py` o validaciones en views.
* Para servir archivos estáticos (si quieres CSS local o imágenes), configura `STATIC_URL` y `STATICFILES_DIRS` en `settings.py` y crea `static/` en la raíz del proyecto.
* Si quieres, preparo el diagrama ERD o el archivo `.drawio` más adelante.

---
