# 🔄 Flujo de Trabajo Diario — Miniconda + JupyterLab

---

## Arrancar a trabajar

**1. Abrir Anaconda Prompt** desde el menú inicio de Windows.

**2. Activar el entorno:**
```bash
conda activate datasci
```
El prompt cambia de `(base)` a `(datasci)`.

**3. Ir a la carpeta de repos:**
```bash
cd C:/ds
```

**4. Abrir JupyterLab:**
```bash
jupyter lab
```

**5. Entrar al navegador:**
- Si no abre solo, pegá en el navegador:
```
http://localhost:8888/lab
```
- Si pide token, copiá la URL completa del Anaconda Prompt que tiene el formato:
```
http://localhost:8888/lab?token=XXXXXXXXXXXXXXXX
```

**6. Trabajar en el notebook:**
- Navegás a la carpeta del repo desde el panel izquierdo
- Abrís el archivo `.ipynb`
- Guardás con `Ctrl+S` cada vez que hagas cambios importantes

---

## Terminar de trabajar

Seguí este orden para liberar toda la memoria RAM y dejar la PC limpia:

**1. Guardar el notebook:**
```
Ctrl+S
```

**2. Cerrar los kernels activos** (esto libera la RAM que usa Python):
```
En JupyterLab → Kernel → Shut Down All Kernels
```

**3. Cerrar JupyterLab correctamente:**
```
File → Shut Down
```
Esto apaga el servidor de Jupyter. Cerrá también la pestaña del navegador.

**4. Confirmar en Anaconda Prompt:**
Vas a ver este mensaje en la terminal:
```
[C] Shutdown confirmed
Jupyter Server stopped.
```

**5. Desactivar el entorno:**
```bash
conda deactivate
```
El prompt vuelve a `(base)`.

**6. Cerrar Anaconda Prompt.**

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
