# Mi Guía de Git

### Configuración inicial:
* `git config --global user.name "Tu Nombre"`: Configura tu nombre
* `git config --global user.email "tu_email@ejemplo.com"`: Configura tu correo

### Configuración y Clonado
* `git clone <url>`: Descarga un repositorio a mi PC.
* `git init`: Inicializa un nuevo repositorio de Git en la carpeta donde estás. Es el primer paso para empezar a
rastrear cambios en un proyecto.
* `cd <nombre-carpeta>`: Entra a la carpeta del proyecto. Ya estás dentro del repositorio haciendo esto.
### ¿Dónde estoy?
- `pwd`: Muestra la carpeta actual.
- `git status`: Si tira "fatal: not a git repository...", no estás dentro de ningún repo.
  Si muestra estado de archivos, estás dentro (no importa en qué subcarpeta).
- `git log`: Muestra el historial de commits, con detalles como el autor, la fecha y el mensaje de cada uno.
- `git pull`: Descarga los cambios del repositorio remoto y los fusiona con tu rama local.

### Manejo de Carpetas
* `git mv nombre_viejo nombre-nuevo`: Renombra una carpeta manteniendo el historial.

### Flujo de Trabajo
1. `git add .`: Prepara los cambios.
2. `git commit -m "Mensaje"`: Guarda el cambio localmente.
3. `git push`: Sube los cambios a la nube.

### Ramas 
* `git branch`: Lista todas las ramas del repositorio.
* `git branch <nombre_de_rama>`: Crea una nueva rama.
* `git checkout <nombre_de_rama>`: Cambia a otra rama para empezar a trabajar en ella.
* `git merge <nombre_de_rama>`: Fusiona los cambios de la rama especificada a la rama en la que te
encuentras.
* `git push -u origin <nombre_de_rama>`: La primera vez que subes una rama, usas este comando para
establecer la conexión y subir los cambios.

# Uso de Git/Github con VSC
**Visual Studio Code tiene una integración excelente con Git.**
* `Vista de Control de Código Fuente`: En el lado izquierdo de VSC, hay un ícono de tres círculos conectados.
Al hacer clic en él, se abre la vista de Control de código fuente.
* `Inicializar un repositorio`: En la vista de Control de código fuente, VSC detectará automáticamente si la carpeta
del proyecto no es un repositorio de Git y te dará la opción de "Inicializar repositorio". Esto es el equivalente a
git init.
* `Hacer un Commit`: Los archivos modificados aparecerán en la vista de Control de código fuente. Puedes pasar
el cursor sobre un archivo y hacer clic en el + para agregarlo al Staging (el equivalente a git add). Una vez que
los archivos estén en Staging, escribe tu mensaje de commit en la caja de texto superior y luego haz clic en el
tic para guardarlo (el equivalente a git commit).
Uso de Git/Github con VSC
* `Sincronizar y Subir Cambios`: En la barra de estado de VSC, en la parte inferior, verás el nombre de la rama
actual (por ejemplo, main). Junto a ella, puede aparecer un indicador de "sincronizar cambios" (una flecha hacia
arriba y otra hacia abajo) o directamente las opciones de pull y push si tienes un repositorio remoto
conectado. Haz clic en "Sincronizar cambios" para subir los commits locales y descargar los remotos al mismo
tiempo, o usa pull para bajar los cambios y push para subirlos.
Para conectar tu proyecto local con un repositorio en GitHub, la primera vez VSC te pedirá que publiques la
rama.
* `Abrir la Terminal`: Si en algún momento necesitas usar un comando directamente, puedes abrir la terminal
integrada de VSC y escribir los comandos que mencionamos antes.