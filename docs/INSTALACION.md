# Guía de Instalación de Northwind en PostgreSQL

Esta guía te ayudará a instalar la base de datos Northwind en PostgreSQL, tanto en **Windows** como en **Linux**.

---

## 1. Descargar el script de Northwind

Puedes descargar el script oficial adaptado para PostgreSQL desde:
- [Northwind PostgreSQL GitHub](https://github.com/pthom/northwind_psql)
- O buscar "northwind postgresql script" en Google.

---

## 2. Crear la base de datos

### Opción A: Usando pgAdmin (GUI)

1. Abre **pgAdmin**.
2. Haz clic derecho en "Databases" y selecciona **Create > Database...**
3. Ponle nombre `northwind` y selecciona el propietario (por defecto, `postgres`).
4. Opcionalmente, ajusta la codificación y el locale si lo necesitas.

### Opción B: Usando la terminal (Windows o Linux)

Abre una terminal (CMD, PowerShell o Bash) y ejecuta:

```bash
createdb -U postgres -E UTF8 -T template0 northwind
```

> Si tu usuario no es `postgres`, cámbialo por el tuyo.  
> En Windows, asegúrate de que la carpeta `bin` de PostgreSQL está en el PATH.

---

## 3. Importar el script SQL

### Opción A: Usando pgAdmin

1. Selecciona la base de datos `northwind` en el panel izquierdo.
2. Ve a **Tools > Query Tool**.
3. Haz clic en el icono de abrir archivo y selecciona el script descargado (por ejemplo, `northwind.sql`).
4. Haz clic en el botón de **ejecutar** (play ▶️) para importar todas las tablas y datos.

### Opción B: Usando la terminal (Windows o Linux)

Navega hasta la carpeta donde tienes el script SQL y ejecuta:

```bash
psql -U postgres -d northwind -f northwind.sql
```

> Si necesitas contraseña, añade la opción `-W` para que la pida.  
> En Linux, puedes usar `sudo -u postgres psql ...` si tu usuario es distinto.

---

## 4. Verificar la instalación

Puedes comprobar que las tablas se han creado correctamente:

- **En pgAdmin:** Navega por el árbol de objetos y revisa las tablas.
- **En la terminal (psql):**

```bash
psql -U postgres -d northwind
```

Y luego, dentro de psql:

```sql
\dt
```

---

## Problemas frecuentes y soluciones

- **psql o createdb no se reconocen como comandos:**
  - Asegúrate de que la carpeta `bin` de PostgreSQL (por ejemplo, `C:\Program Files\PostgreSQL\17\bin` en Windows) está añadida a la variable de entorno `PATH`.
  - En Windows:  
    1. Ve a Panel de control > Sistema > Configuración avanzada del sistema > Variables de entorno.
    2. Busca la variable `Path` y edítala.
    3. Añade la ruta donde está instalado PostgreSQL, normalmente `C:\Program Files\PostgreSQL\XX\bin`.
    4. Reinicia la terminal o el equipo si es necesario.
  - En Linux:  
    Añade la línea siguiente a tu archivo `~/.bashrc` o `~/.zshrc` (ajusta la ruta según tu instalación):
    ```bash
    export PATH=$PATH:/usr/pgsql-17/bin
    ```
    Luego ejecuta `source ~/.bashrc` o abre una nueva terminal.

- **Permisos denegados o problemas de usuario:**
  - Asegúrate de ejecutar los comandos como el usuario correcto (`postgres` por defecto).
  - En Linux, puedes usar `sudo -u postgres ...` para ejecutar como el usuario de PostgreSQL.

- **Error de conexión:**
  - Verifica que el servidor de PostgreSQL está iniciado.
  - Comprueba el puerto (por defecto, 5432) y el usuario/contraseña.

---

**¡y... Acabaste! (Si todo ha salido bien ;P)**  
Ahora tienes Northwind instalado en PostgreSQL y puedes empezar a trabajar o modificar la base de datos.

