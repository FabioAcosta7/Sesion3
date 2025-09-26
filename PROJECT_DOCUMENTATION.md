# Proyecto Integrador - Documentación del Código

**Descripción breve**
Este proyecto es una pequeña aplicación demo construida con `tkinter` que muestra varias ventanas (home, formulario, lista CRUD, tabla con `Treeview`, canvas de dibujo). Está organizada en módulos dentro del paquete `app`. El punto de entrada es `main.py`.

---

## Estructura de directorios (esperada)
```
<repo>/
├─ main.py
├─ app/
│  ├─ win_home.py
│  ├─ win_form.py
│  ├─ win_list.py
│  ├─ win_table.py
│  └─ win_canvas.py
├─ data/
│  └─ sample.csv   # usado por win_table.py
└─ README.md (opcional)
```

> Nota: En el archivo entregado, los módulos están concatenados en `all.txt`. La estructura anterior corresponde a cómo deberían estar organizados en el proyecto real.

---

## Requisitos
- Python 3.8+ (se usa `tkinter` estándar)
- Módulos estándar: `tkinter`, `csv`, `pathlib`.
- No hay dependencias externas adicionales.

Para crear un entorno virtual y ejecutar:
```bash
python -m venv .venv
source .venv/bin/activate   # Linux / macOS
.venv\Scripts\activate    # Windows (PowerShell/CMD)
python -m pip install --upgrade pip
# No requirements.txt necesario: solo librerías stdlib
python main.py
```

---

## Archivos y explicación (por archivo)

### `main.py`
Punto de entrada de la aplicación. Crea la ventana principal con botones que abren ventanas secundarias (cada ventana en su propio módulo dentro de `app`):

- Configura `root` con tamaño `420x340` y un `Frame` con botones:
  - `open_win_home(root)` — ventana de bienvenida.
  - `open_win_form(root)` — formulario que guarda un archivo texto.
  - `open_win_list(root)` — lista con operaciones básicas (agregar, eliminar, limpiar).
  - `open_win_table(root)` — carga `data/sample.csv` en un `Treeview`.
  - `open_win_canvas(root)` — muestra dibujos en un `Canvas`.

Buenas prácticas observadas:
- Interfaz simple con `ttk` para estilos modernos.
- Uso de `lambda` para pasar `root` como `parent` a las funciones que abren ventanas.

### `app/win_home.py`
Función `open_win_home(parent: tk.Tk)` que crea una ventana `Toplevel` con mensaje de bienvenida y un botón que muestra un `messagebox` informativo.

Comportamiento:
- Ventana modal no bloqueante (no se usa `grab_set()`).
- Botón "Mostrar mensaje" muestra `messagebox.showinfo()`.

Sugerencia: si desea que la ventana actúe modalmente, usar `win.grab_set()` y `win.transient(parent)`.

### `app/win_form.py`
Función `open_win_form(parent: tk.Tk)` que crea un formulario con campos `Nombre` y `Edad`.

Flujo:
1. Valida que `Nombre` no esté vacío.
2. Verifica que `Edad` sea numérica (`.isdigit()`).
3. Abre diálogo de guardado `filedialog.asksaveasfilename` y escribe un archivo `.txt` con los datos.
4. Usa `messagebox` para informar errores o éxito.

Observaciones/sugerencias:
- `ent_edad.get().strip().isdigit()` no acepta edades negativas ni números con espacios o signos. Si desea permitir rangos válidos, convertir con `int()` dentro de `try/except` y validar rango razonable.
- Considerar separar la lógica de validación en funciones para facilitar pruebas unitarias.
- Añadir `win.transient(parent)` y `win.grab_set()` si quiere que sea modal.
- Manejar excepciones al escribir el archivo (p. ej. permisos) con `try/except` para mostrar un `messagebox.showerror()`.

### `app/win_list.py`
Función `open_win_list(parent: tk.Tk)` que ofrece una lista `Listbox` con operaciones CRUD básicas en memoria (no persistente):

- `agregar()` inserta texto del `Entry` en el `Listbox` si no está vacío.
- `eliminar()` borra el elemento seleccionado.
- `limpiar()` borra todos los elementos.

Observaciones:
- La interfaz usa `grid` y configura `columnconfigure`/`rowconfigure` para dimensionado. Esto está bien para esta escala.
- Si se quiere persistencia, añadir guardado en disco (CSV/JSON) o usar SQLite.

### `app/win_table.py`
Función `open_win_table(parent: tk.Tk)` que crea un `Treeview` y lo llena leyendo `data/sample.csv`.

Detalles importantes:
- Columnas: `("nombre", "valor1", "valor2")`
- Ubicación del CSV calculada con:
```py
ruta = Path(__file__).resolve().parents[2] / "data" / "sample.csv"
```
Esto asume que la jerarquía de carpetas es así: `<repo>/app/<subfolder?>/...`. `parents[2]` sube dos niveles desde `app/win_table.py` — revisar que en su repo real esa ruta exista.
- Si no existe, se muestra una `messagebox` y la función retorna sin mostrar el contenido.

Formato esperado del `sample.csv` (encabezados):
```csv
nombre,valor1,valor2
A,10,20
B,5,7
```

Sugerencias:
- Comprobar `KeyError` si las columnas no existen y mostrar mensaje claro.
- Permitir seleccionar un archivo CSV mediante `filedialog.askopenfilename` como alternativa.
- Añadir botón para exportar/guardar cambios (si se implementa edición).
- Considerar `tv.delete(*tv.get_children())` antes de cargar si la ventana puede recargarse.

### `app/win_canvas.py`
Función `open_win_canvas(parent: tk.Tk)` que abre una ventana con un `Canvas` de ejemplo y dibuja:
- Rectángulo, óvalo, línea y un texto central.
Es útil como demostración de uso básico del widget `Canvas`.

Sugerencias:
- Permitir interacción (dibujar con el mouse) añadiendo bindings a eventos `<Button-1>`, `<B1-Motion>`, etc.
- Añadir botones para limpiar o guardar como imagen (requiere PIL/pillow para exportar).

---

## Errores potenciales y recomendaciones de robustez
1. **Ruta relativa en `win_table.py`**: `parents[2]` puede producir una ruta incorrecta si la estructura de carpetas cambia. Mejor determinar la ruta del proyecto con una variable raíz o usar `Path.cwd()` según cómo se distribuya la app.
2. **Validaciones**: mejorar validación y manejo de excepciones en `win_form` al guardar archivo y en `win_table` al leer CSV.
3. **Modalidad de ventanas**: si desea ventanas modales, usar `win.transient(parent)` y `win.grab_set()` para evitar que el usuario interactúe con la ventana padre mientras la secundaria está abierta.
4. **Internacionalización**: el texto está en español; si se desea multilenguaje, extraer textos a un archivo de recursos.
5. **Pruebas**: agregar pruebas unitarias para lógica no UI (por ejemplo, validación de edad) y tests de integración con `pytest` y `pytest-mock` para funciones aisladas.

---

## Mejoras recomendadas (priorizadas)
1. **Agregar `requirements.txt` y scripts de inicio** (si se añaden dependencias externas).
2. **Refactor**: extraer lógica de validación y acceso a archivos a módulos separados (`app/utils.py`), haciendo las funciones testables sin GUI.
3. **Persistencia**: añadir guardado/lectura en JSON/CSV o una pequeña DB SQLite para la lista (win_list).
4. **Mejor experiencia de usuario**: campos con validación en tiempo real, placeholders, mensajes más descriptivos.
5. **Packaging**: crear un instalador simple o `pyproject.toml` para distribuir la aplicación.
6. **Soporte para dark mode**: `ttk` themes o una librería para skins.

---

## Ejecución y pruebas rápidas
- Ejecutar `python main.py` abre la ventana principal.
- Probar `Formulario`: escribir nombre y edad; intentar dejar nombre vacío o edad con texto para ver validaciones.
- Probar `Tabla (Treeview)`: crear `data/sample.csv` con los encabezados `nombre,valor1,valor2`. Si no existe, aparecerá un aviso.

---

## Licencia y atribución
No se incluyó una licencia. Recomiendo agregar un `LICENSE` (MIT o similar) si desea compartir el proyecto.

---

## Conclusión
Proyecto pequeño y didáctico, ideal como MVP para aprender `tkinter` y estructura de ventanas. Recomendado separar responsabilidades (UI vs lógica) para escalar el proyecto y facilitar pruebas unitarias.

---
