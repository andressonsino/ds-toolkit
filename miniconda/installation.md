# Instalación de Miniconda en Windows

## Descarga

Descargá el instalador directamente sin necesidad de registrarte:

```
https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe
```

---

## Instalación paso a paso

### 1. Ejecutar como administrador
Click derecho en el `.exe` → **Ejecutar como administrador**

### 2. Tipo de instalación
Elegir **Just Me**

### 3. Ruta de instalación
```
C:\miniconda3
```

### 4. Advanced Installation Options
| Opción | Estado |
|--------|--------|
| Create shortcuts | ✅ tildar |
| Add to PATH environment variable | ❌ NO tildar |
| Register as default Python | ✅ tildar |
| Clear the package cache upon completion | ✅ tildar |

> **Por qué no agregar al PATH:** si se agrega al PATH global, conda se carga en cada terminal aunque no lo uses, consumiendo recursos. Sin PATH, conda existe solo cuando abrís el Anaconda Prompt.

---

## Configuración post-instalación

Abrí **Anaconda Prompt como administrador** desde el menú Inicio y ejecutá estos comandos uno por uno:

### Aceptar términos de servicio
```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/msys2
```

### Desactivar auto-activación del entorno base
```bash
conda config --set auto_activate_base false
```

### Actualizar conda
```bash
conda update -n base -c defaults conda -y
```

### Configurar canal conda-forge y solver rápido
```bash
conda config --add channels conda-forge
conda config --set channel_priority strict
conda install -n base conda-libmamba-solver -y
conda config --set solver libmamba
```

---

## Verificar consumo 0% cuando no se usa

Con esta configuración, Miniconda **no corre ningún proceso en background**. No usa RAM, no usa CPU. Es simplemente una carpeta en disco hasta que abrís el Anaconda Prompt.

Para confirmarlo: Administrador de Tareas → no debe aparecer ningún proceso de conda, python ni miniconda cuando el Anaconda Prompt está cerrado.

---

## Uso diario

| Acción | Cómo |
|--------|------|
| Usar conda | Abrir **Anaconda Prompt** desde el menú Inicio |
| Actualizar conda | Abrir Anaconda Prompt **como administrador** |
| Dejar de usar | Cerrar la ventana del Anaconda Prompt |

> Miniconda está instalado en `C:\miniconda3`. Los entornos se guardan en `C:\miniconda3\envs\`.
