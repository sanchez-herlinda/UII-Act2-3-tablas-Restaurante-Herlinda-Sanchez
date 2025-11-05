
---

# **Guía paso a paso (Primera parte) — Proyecto Restaurante (Django, Python, VS Code)**

**Proyecto:** Restaurante
**Lenguaje:** Python
**Framework:** Django
**Editor:** VS Code

---

### **1. Procedimiento para crear carpeta del Proyecto:**

Crear la carpeta llamada **UIII_Restaurante_0344**.

---

### **2. Procedimiento para abrir VS Code sobre la carpeta UIII_Restaurante_0344:**

Abrir Visual Studio Code y seleccionar **Archivo → Abrir carpeta → UIII_Restaurante_0344.**

---

### **3. Procedimiento para abrir terminal en VS Code:**

En VS Code abrir la terminal con la combinación de teclas **Ctrl + ñ** o desde el menú **Ver → Terminal.**

---

### **4. Procedimiento para crear carpeta entorno virtual “.venv” desde terminal de VS Code:**

En la terminal escribir el siguiente comando:

```
python -m venv .venv
```

---

### **5. Procedimiento para activar el entorno virtual:**

Ejecutar el siguiente comando:

```
.venv\Scripts\activate
```

---

### **6. Procedimiento para activar intérprete de Python:**

En la parte inferior derecha de VS Code seleccionar el intérprete de Python correspondiente al entorno virtual **.venv**.

---

### **7. Procedimiento para instalar Django:**

En la terminal escribir el comando:

```
pip install django
```

---

### **8. Procedimiento para crear proyecto backend_Restaurante sin duplicar carpeta:**

Desde la terminal ejecutar el comando:

```
django-admin startproject backend_Restaurante .
```

El punto al final evita que se duplique la carpeta del proyecto.

---

### **9. Procedimiento para ejecutar servidor en el puerto 8044:**

En la terminal ejecutar:

```
python manage.py runserver 8044
```

---

### **10. Procedimiento para copiar y pegar el link en el navegador:**

Copiar el enlace que aparece en la terminal:

```
http://127.0.0.1:8044/
```

y pegarlo en el navegador para comprobar que el servidor funciona correctamente.

---

### **11. Procedimiento para crear aplicación app_Restaurante:**

En la terminal escribir el comando:

```
python manage.py startapp app_Restaurante
```

---

### **12. Aquí el modelo models.py**

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
    fecha_creacion = models.DateTimeField(auto_now_add=True)  
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

### **12.5 Procedimiento para realizar las migraciones (makemigrations y migrate):**

Ejecutar los siguientes comandos:

```
python manage.py makemigrations
python manage.py migrate
```

---

### **13. Primero trabajamos con el MODELO: PLATILLO**

Se realizarán las vistas, plantillas y rutas correspondientes únicamente para el modelo **Platillo**.

---

### **14. En views de app_Restaurante crear las funciones con sus códigos correspondientes:**

* inicio_restaurante
* agregar_platillo
* actualizar_platillo
* realizar_actualizacion_platillo
* borrar_platillo

---

### **15. Crear la carpeta “templates” dentro de “app_Restaurante”:**

Ruta:

```
app_Restaurante/templates
```

---

### **16. En la carpeta templates crear los archivos html:**

* base.html
* header.html
* navbar.html
* footer.html
* inicio.html

---

### **17. En el archivo base.html agregar Bootstrap para CSS y JS.**

---

### **18. En el archivo navbar.html incluir las opciones:**

**“Sistema de Administración Restaurante”**, **“Inicio”**, **“Platillos”** con su submenú (Agregar Platillo, Ver Platillos, Actualizar Platillo, Borrar Platillo),
**“Pedidos”** con su submenú (Agregar, Ver, Actualizar, Borrar) y
**“Detalle Pedido”** con su submenú (Agregar, Ver, Actualizar, Borrar).
Agregar íconos en las opciones principales, no en los submenús.

---

### **19. En el archivo footer.html incluir:**

Derechos de autor, fecha del sistema y la frase:
**“Creado por Ing. Eliseo Nava, Cbtis 128”**, manteniéndola fija al final de la página.

---

### **20. En el archivo inicio.html colocar información del sistema:**

Agregar una descripción del sistema y una imagen relacionada con un restaurante, tomada desde la red.

---

### **21. Crear la subcarpeta “platillo” dentro de app_Restaurante/templates:**

Ruta:

```
app_Restaurante/templates/platillo
```

---

### **22. Crear los archivos html dentro de platillo:**

* agregar_platillo.html
* ver_platillos.html (mostrar en tabla con botones Ver, Editar y Borrar)
* actualizar_platillo.html
* borrar_platillo.html

---

### **23. No utilizar forms.py.**

---

### **24. Procedimiento para crear el archivo urls.py en app_Restaurante:**

Agregar el código correspondiente para enlazar las funciones CRUD de views.py.

---

### **25. Procedimiento para agregar app_Restaurante en settings.py de backend_Restaurante:**

Dentro de **INSTALLED_APPS** agregar:

```python
'app_Restaurante',
```

---

### **26. Realizar configuraciones correspondientes a urls.py de backend_Restaurante:**

Enlazar el archivo **urls.py** principal con las rutas de **app_Restaurante**.

---

### **27. Procedimiento para registrar los modelos en admin.py y volver a realizar las migraciones:**

Registrar los modelos **Platillo**, **Pedido** y **DetallePedido** en admin.py.

---

### **27.5 Por lo pronto solo trabajar con “platillo”:**

Dejar pendiente **# MODELO: PEDIDO** y **# MODELO: DETALLEPEDIDO**.

---

### **28. Utilizar colores suaves, atractivos y modernos:**

Diseñar las páginas con estilos sencillos y agradables visualmente.

---

### **28.5 No validar entrada de datos.**

---

### **29. Al inicio crear la estructura completa de carpetas y archivos.**

---

### **30. Proyecto totalmente funcional.**

---

### **31. Finalmente ejecutar servidor en el puerto 8044:**

```
python manage.py runserver 8044
```

---

