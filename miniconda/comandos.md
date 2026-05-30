# 🐍 Guía Completa de Miniconda y Entornos de Trabajo

> **Nivel:** Básico → Avanzado  
> **Requisitos:** Miniconda instalado en Windows

---

## Tabla de Contenidos

1. [Conceptos clave](#1-conceptos-clave)
2. [Instalación y configuración inicial](#2-instalación-y-configuración-inicial)
3. [Gestión de entornos](#3-gestión-de-entornos)
4. [Gestión de paquetes](#4-gestión-de-paquetes)
5. [Flujo de trabajo diario](#5-flujo-de-trabajo-diario-miniconda-jupyterLab)
6. [Jupyter desde Miniconda](#6-jupyter-desde-miniconda)
7. [Solución de problemas comunes](#7-solución-de-problemas-comunes)
8. [Comandos avanzados](#8-comandos-avanzados)
9. [Resumen rápido (cheat sheet)](#9-resumen-rápido-cheat-sheet)

---

## 1. Conceptos clave

**¿Qué es Miniconda?**
Miniconda es un gestor de entornos y paquetes para Python. Permite tener múltiples versiones de Python y librerías instaladas de forma aislada, sin que interfieran entre sí.

**¿Qué es un entorno (environment)?**
Una carpeta aislada con su propia versión de Python y sus propias librerías. Si algo se rompe en un entorno, los demás no se ven afectados.

| Entorno | Para qué usarlo |
|---------|----------------|
| `(base)` | Entorno principal de Miniconda — no instalar nada acá |
| Entorno propio | Un entorno por proyecto o área de trabajo |

**¿Dónde se guardan los entornos?**

Por defecto en:
```
C:\miniconda3\envs\nombre-entorno\
```

Si tu usuario tiene caracteres especiales (como `é`, `ñ`, espacios), crear el entorno en una ruta limpia:
```
C:\nombre-entorno\
```

---

## 2. Instalación y configuración inicial

**Abrir Miniconda en Windows:**
Buscás **Anaconda Prompt (miniconda3)** en el menú inicio. Es una terminal especial — no uses el CMD normal.

**Verificar instalación:**
```bash
conda --version
python --version
```

**Actualizar conda:**
```bash
conda update conda
```

---

## 3. Gestión de entornos

### 3.1 Crear un entorno

```bash
# Forma estándar
conda create -n nombre-entorno python=3.11

# ⚠️ Si tu usuario Windows tiene caracteres especiales (é, ñ, espacios)
# usá ruta absoluta para evitar errores de OpenSSL y PATH
conda create --prefix C:/nombre-entorno python=3.11
```

> 💡 **Caso real:** Si tu usuario es `Andrés`, la ruta `C:\Users\Andrés\.conda\envs\` puede romper scripts de OpenSSL. La solución es crear el entorno con `--prefix` en una ruta limpia como `C:/datasci`.

### 3.2 Activar y desactivar

```bash
# Activar entorno por nombre
conda activate nombre-entorno

# Activar entorno por ruta (cuando usaste --prefix)
conda activate C:/nombre-entorno

# Desactivar entorno activo
conda deactivate
```

El prompt cambia de `(base)` a `(nombre-entorno)` cuando está activo.

### 3.3 Listar entornos

```bash
conda env list
```

Ejemplo de salida:
```
# conda environments:
base          *  C:\miniconda3        ← activo (*)
datasci          C:\miniconda3\envs\datasci
```

### 3.4 Eliminar un entorno

```bash
# Por nombre
conda env remove -n nombre-entorno

# Por ruta
conda env remove --prefix C:/nombre-entorno
```

### 3.5 Clonar un entorno

```bash
conda create -n entorno-copia --clone entorno-original
```

### 3.6 Exportar e importar entornos

```bash
# Exportar — genera un archivo con todas las dependencias
conda env export > environment.yml

# Importar — recrea el entorno en otra máquina
conda env create -f environment.yml
```

> 💡 El archivo `environment.yml` es lo que subís a GitHub para que otros puedan replicar tu entorno exacto.

---

## 4. Gestión de paquetes

### 4.1 Instalar paquetes

```bash
# Con pip (recomendado para paquetes de ciencia de datos)
pip install nombre-paquete

# Con conda
conda install nombre-paquete

# Versión específica
pip install tensorflow==2.12.0

# Varios paquetes de una vez
pip install jupyter tensorflow pandas numpy matplotlib scikit-learn
```

### 4.2 Ver paquetes instalados

```bash
# Lista completa
pip list

# Buscar un paquete específico
pip list | findstr tensorflow   # Windows
pip list | grep tensorflow      # Mac/Linux
```

### 4.3 Actualizar paquetes

```bash
pip install --upgrade nombre-paquete
pip install --upgrade pip
```

### 4.4 Desinstalar paquetes

```bash
pip uninstall nombre-paquete
```

### 4.5 Ver dependencias de un paquete

```bash
pip show nombre-paquete
```

### 4.6 Guardar paquetes instalados en un archivo

```bash
# Genera requirements.txt con todas las versiones exactas
pip freeze > requirements.txt

# Instalar desde requirements.txt
pip install -r requirements.txt
```

---

## 5. Flujo de trabajo diario — Miniconda + JupyterLab
 
---

## Arrancar a trabajar

### 5.1. Abrir Anaconda Prompt** desde el menú inicio de Windows.

### 5.2. Activar el entorno:**
```bash
conda activate datasci
```
El prompt cambia de `(base)` a `(datasci)`.

### 5.3. Ir a la carpeta de repos:**
```bash
cd C:/ds
```

### 5.4. Abrir JupyterLab:**
```bash
jupyter lab
```

### 5.5. Entrar al navegador:**
- Si no abre solo, pegá en el navegador:
```
http://localhost:8888/lab
```
- Si pide token, copiá la URL completa del Anaconda Prompt que tiene el formato:
```
http://localhost:8888/lab?token=XXXXXXXXXXXXXXXX
```

### 5.6. Trabajar en el notebook:**
- Navegás a la carpeta del repo desde el panel izquierdo
- Abrís el archivo `.ipynb`
- Guardás con `Ctrl+S` cada vez que hagas cambios importantes

---

## Terminar de trabajar

Seguí este orden para liberar toda la memoria RAM y dejar la PC limpia:

### 5.7. Guardar el notebook:**
```
Ctrl+S
```

### 5.8. Cerrar los kernels activos** (esto libera la RAM que usa Python):
```
En JupyterLab → Kernel → Shut Down All Kernels
```

### 5.9. Cerrar JupyterLab correctamente:**
```
File → Shut Down
```
Esto apaga el servidor de Jupyter. Cerrá también la pestaña del navegador.

### 5.10. Confirmar en Anaconda Prompt:**
Vas a ver este mensaje en la terminal:
```
[C] Shutdown confirmed
Jupyter Server stopped.
```

### 5.11. Desactivar el entorno:**
```bash
conda deactivate
```
El prompt vuelve a `(base)`.

### 5.12. Cerrar Anaconda Prompt.**

---

## Subir cambios a GitHub

Desde **Git Bash** (no desde Anaconda Prompt):

```bash
cd C:/ds/nombre-del-repo
git add .
git commit -m "descripción del cambio"
git push
```

> 💡 Hacé esto antes de cerrar JupyterLab para no olvidarte.

---

## Resumen visual

```
ARRANCAR                          TERMINAR
────────────────────              ────────────────────────────
Anaconda Prompt                   Ctrl+S (guardar notebook)
  ↓                                 ↓
conda activate datasci            Kernel → Shut Down All Kernels
  ↓                                 ↓
cd C:/ds                          File → Shut Down
  ↓                                 ↓
jupyter lab                       Cerrar pestaña del navegador
  ↓                                 ↓
Abrir navegador                   conda deactivate
  ↓                                 ↓
Trabajar                          Cerrar Anaconda Prompt
  ↓
Ctrl+S para guardar
  ↓
git add . → commit → push
```

---

> ⚠️ **No cierres el Anaconda Prompt mientras Jupyter esté corriendo** — eso mata el servidor abruptamente y puede corromper notebooks no guardados.


---

## 6. Jupyter desde Miniconda

### 6.1 Abrir Jupyter

```bash
# Jupyter Notebook (interfaz clásica)
jupyter notebook

# JupyterLab (interfaz moderna — recomendada)
jupyter lab
```

Se abre automáticamente en el navegador. Si no abre solo:
```
http://localhost:8888
```

### 6.2 Abrir en una carpeta específica

```bash
# Navegar primero a la carpeta
cd C:/ds/clasificacion-redes-neuronales
jupyter lab
```

### 6.3 Cerrar Jupyter correctamente

Desde el navegador: **File → Shut Down**  
O en la terminal: `Ctrl+C` dos veces

### 6.4 Registrar el entorno como kernel de Jupyter

Si Jupyter no reconoce tu entorno:
```bash
pip install ipykernel
python -m ipykernel install --user --name=datasci --display-name "Python (datasci)"
```

---

## 7. Solución de problemas comunes

### ❌ Error: OpenSSL activation script failed

```
ERROR: Activation script "C:\Users\Andrés\.conda\envs\ds-env\etc\conda\activate.d\openssl_activate-win.bat" failed with code 0.
```

**Causa:** El nombre de usuario Windows tiene caracteres especiales (`é`, `ñ`, espacios).

**Solución:** Crear el entorno con `--prefix` en una ruta sin caracteres especiales:
```bash
conda env remove -n nombre-entorno    # borrar el entorno problemático
conda create --prefix C:/datasci python=3.11
conda activate C:/datasci
```

---

### ❌ Error: python no se reconoce como comando

```
"python" no se reconoce como un comando interno o externo
```

**Causa:** El entorno se activó pero Python no se instaló correctamente (relacionado con el error de OpenSSL).

**Solución:** Borrar y recrear el entorno en ruta limpia (ver error anterior).

---

### ❌ Error: conda no se reconoce

**Causa:** Estás usando CMD normal en lugar de Anaconda Prompt.

**Solución:** Cerrar CMD y abrir **Anaconda Prompt (miniconda3)** desde el menú inicio.

---

### ❌ Warning: script no está en PATH

```
WARNING: The script X is installed in 'C:\Users\...' which is not on PATH.
```

**Causa:** Scripts instalados en una ubicación no registrada en las variables de entorno.

**Solución:** Ignorar si las herramientas principales funcionan. No afecta jupyter, pandas, tensorflow ni numpy.

---

### ❌ Jupyter abre pero no encuentra el entorno como kernel

**Solución:**
```bash
conda activate datasci
pip install ipykernel
python -m ipykernel install --user --name=datasci --display-name "Python (datasci)"
```

---

## 8. Comandos avanzados

### 8.1 Crear entorno desde archivo

```bash
# Crear environment.yml manualmente o exportarlo de otro entorno
conda env create -f environment.yml
```

Ejemplo de `environment.yml`:
```yaml
name: datasci
channels:
  - defaults
dependencies:
  - python=3.11
  - pip
  - pip:
    - jupyter
    - pandas
    - numpy
    - matplotlib
    - scikit-learn
    - tensorflow
```

### 8.2 Ver información del entorno activo

```bash
conda info
```

### 8.3 Ver versión de un paquete específico

```bash
python -c "import pandas; print(pandas.__version__)"
python -c "import tensorflow; print(tensorflow.__version__)"
```

### 8.4 Limpiar caché de conda

```bash
conda clean --all
```

Libera espacio eliminando paquetes descargados que ya no se necesitan.

### 8.5 Buscar paquetes disponibles

```bash
conda search nombre-paquete
pip index versions nombre-paquete
```

### 8.6 Instalar paquete en modo desarrollo

```bash
pip install -e .
```

Útil cuando desarrollás tu propio paquete y querés que los cambios se reflejen sin reinstalar.

---

## 9. Resumen rápido (cheat sheet)

### Entornos

```bash
conda env list                              # listar entornos
conda create -n nombre python=3.11          # crear entorno
conda create --prefix C:/nombre python=3.11 # crear en ruta limpia (Windows con caracteres especiales)
conda activate nombre                       # activar
conda activate C:/nombre                    # activar por ruta
conda deactivate                            # desactivar
conda env remove -n nombre                  # eliminar
conda env export > environment.yml          # exportar
conda env create -f environment.yml         # importar
```

### Paquetes

```bash
pip list                          # ver instalados
pip install paquete               # instalar
pip install paquete==1.0.0        # versión específica
pip install --upgrade paquete     # actualizar
pip uninstall paquete             # desinstalar
pip freeze > requirements.txt     # exportar dependencias
pip install -r requirements.txt   # instalar desde archivo
```

### Jupyter

```bash
jupyter notebook    # abrir Jupyter Notebook
jupyter lab         # abrir JupyterLab
```

### Flujo diario

```bash
conda activate datasci
cd C:/ds
jupyter lab
# ... trabajás ...
# Ctrl+S en Jupyter para guardar
# File → Shut Down para cerrar Jupyter
conda deactivate
```

---

## Entorno configurado para ciencia de datos

Librerías esenciales para instalar en el entorno:

```bash
pip install jupyter pandas numpy matplotlib scikit-learn tensorflow scipy seaborn plotly
```

| Librería | Para qué |
|----------|----------|
| `jupyter` | Entorno de notebooks |
| `pandas` | Manipulación de datos |
| `numpy` | Operaciones numéricas |
| `matplotlib` | Visualización básica |
| `seaborn` | Visualización estadística |
| `plotly` | Visualización interactiva |
| `scikit-learn` | Machine learning clásico |
| `tensorflow` | Redes neuronales |
| `scipy` | Estadística y ciencia |

---

*Guía elaborada para uso personal y académico · Miniconda Documentation: [docs.conda.io](https://docs.conda.io)*
