## Carga del dataset



### Opciones A — Plantillas con Fallback Automático



```python

# ── Configuración ──────────────────────────────────────────
DATASET_KAGGLE_PROPIO   = "andrssonsinogrugni/nombre-dataset"  # ← CAMBIAR: tu dataset en Kaggle
DATASET_KAGGLE_EXTERNO  = "mlg-ulb/creditcardfraud"            # ← CAMBIAR: dataset de otro usuario
DATA_PATH_LOCAL         = r"data/raw/archivo.csv"                  # ← CAMBIAR: ruta local
# ───────────────────────────────────────────────────────────

df_raw = None

# ── Prioridad 1: archivo local ya descargado ───────────────
if os.path.exists(DATA_PATH_LOCAL):
    df_raw = pd.read_csv(DATA_PATH_LOCAL)
    print(f"✅ Dataset cargado desde archivo local: {DATA_PATH_LOCAL}")

# ── Prioridad 2: tus propios datasets de Kaggle ────────────
if df_raw is None:
    try:
        import subprocess
        subprocess.run(["pip", "install", "-q", "kagglehub"], check=True)
        import kagglehub
        path = kagglehub.dataset_download(DATASET_KAGGLE_PROPIO)
        archivos = os.listdir(path)
        print("Archivos encontrados:", archivos)
        df_raw = pd.read_csv(f"{path}/{archivos[0]}")
        print("✅ Dataset cargado desde tus datasets de Kaggle")
    except Exception as e:
        print(f"⚠️ No se pudo cargar desde tu Kaggle: {e}")

# ── Prioridad 3: dataset externo de Kaggle ─────────────────
if df_raw is None:
    try:
        import kagglehub
        path = kagglehub.dataset_download(DATASET_KAGGLE_EXTERNO)
        archivos = os.listdir(path)
        print("Archivos encontrados:", archivos)
        df_raw = pd.read_csv(f"{path}/{archivos[0]}")
        print("✅ Dataset cargado desde dataset externo de Kaggle")
    except Exception as e:
        raise FileNotFoundError(
            f"No se pudo cargar el dataset desde ninguna fuente.\n"
            f"Opciones manuales:\n"
            f"  1. Colocá el archivo en: {DATA_PATH_LOCAL}\n"
            f"  2. Subí tu dataset a: https://www.kaggle.com/datasets/{DATASET_KAGGLE_PROPIO}\n"
            f"  3. Descargalo desde: https://www.kaggle.com/datasets/{DATASET_KAGGLE_EXTERNO}\n"
        )

print(f"\nDataset listo — Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}")

```
### Reproducible para Colab, VS code y Jupyter Lab

```python
# ── Configuración ──────────────────────────────────────────
DATA_PATH_LOCAL = r"data/raw/archivo.csv"   # ← CAMBIAR: ruta local
DRIVE_PATH_REL  = "MyDrive/ruta/en/drive/archivo.csv"  # ← CAMBIAR: ruta relativa dentro de Drive
SEPARATOR       = ","                        # ← CAMBIAR: separador del CSV
ENCODING        = "utf-8"                    # ← CAMBIAR: encoding del archivo
# ───────────────────────────────────────────────────────────

df_raw = None

# ── Prioridad 1: archivo local ─────────────────────────────
import os
if os.path.exists(DATA_PATH_LOCAL):
    df_raw = pd.read_csv(DATA_PATH_LOCAL, sep=SEPARATOR, encoding=ENCODING)
    print(f"✅ Dataset cargado desde archivo local: {DATA_PATH_LOCAL}")

# ── Prioridad 2: Google Drive (solo si estamos en Colab) ──
if df_raw is None:
    try:
        # Detectamos si estamos en Colab
        import sys
        IN_COLAB = 'google.colab' in sys.modules
    except:
        IN_COLAB = False

    if IN_COLAB:
        try:
            from google.colab import drive
            drive.mount('/content/drive')
            ruta_drive = f"/content/drive/{DRIVE_PATH_REL}"
            df_raw = pd.read_csv(ruta_drive, sep=SEPARATOR, encoding=ENCODING)
            print(f"✅ Dataset cargado desde Google Drive: {ruta_drive}")
        except Exception as e:
            print(f"⚠️ Error al cargar desde Google Drive: {e}")
    else:
        print("ℹ️ No estás en Colab. Omitiendo carga desde Google Drive.")

# ── Prioridad 3: error si no se pudo cargar ────────────────
if df_raw is None:
    raise FileNotFoundError(
        f"No se pudo cargar el dataset.\n"
        f"Opciones:\n"
        f"  1. Colocá el archivo en: {DATA_PATH_LOCAL}\n"
        f"  2. Si usás Colab, ajustá DRIVE_PATH_REL y asegurate de tener el archivo en Drive.\n"
        f"  3. Subí el archivo manualmente a Colab con files.upload() y ajustá la ruta."
    )

print(f"\nDataset listo — Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}")

```
### Opción B — Desde archivo local



```python

# ── Configuración ──────────────────────────────────────────
DATA_PATH = r"data/raw/archivo.csv"  # ← CAMBIAR: ruta al archivo usar ../ al principio si debo ubicarme en una carpeta anterior
SEPARATOR = ","                  # ← CAMBIAR si es otro separador (";", "\t", etc.)
ENCODING  = "utf-8"              # ← CAMBIAR si hay problemas de caracteres
# ───────────────────────────────────────────────────────────

df_raw = pd.read_csv(DATA_PATH, sep=SEPARATOR, encoding=ENCODING)
print(f"✅ Dataset cargado — Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}")

```



### Opción C — Desde Google Colab



```python

from google.colab import files

df_raw = pd.read_csv(list(files.upload().keys())[0])
print(f'Dataset cargado: {df_raw.shape[0]} filas x {df_raw.shape[1]} columnas')
df_raw.head()

```



### Opción D - Desde Kaggle

Descarga el dataset desde Kaggle y lo guarda en caché local.

Si el dataset se actualiza en Kaggle, kagglehub lo re-descarga automáticamente.



**Cuándo usarla:** dataset de Kaggle, querés reproducibilidad sin descargar manualmente.



```python

# ── Configuración ──────────────────────────────────────────
DATASET_KAGGLE = "mlg-ulb/creditcardfraud"  # ← CAMBIAR: usuario/nombre-dataset
# ───────────────────────────────────────────────────────────

import kagglehub
import os

# Descarga a caché local (no re-descarga si ya está actualizado)
path = kagglehub.dataset_download(DATASET_KAGGLE)

# Detecta el archivo automáticamente
archivos = os.listdir(path)
print("Archivos descargados:", archivos)

# Carga el primer archivo CSV encontrado
df_raw = pd.read_csv(f"{path}/{archivos[0]}")
print(f"✅ Dataset cargado — Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}")

```



### Opción E - Desde Scikit-learn (datasets built-in)

**Cuándo usarla:** datasets clásicos como Iris, Wine, Breast Cancer, Digits.



```python

from sklearn.datasets import load_iris  # ← CAMBIAR: load_wine, load_breast_cancer, etc.

data = load_iris()
df_raw = pd.DataFrame(data.data, columns=data.feature_names)
df_raw['target'] = data.target

print(f"✅ Dataset cargado — Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}")
print(f"Clases: {data.target_names}")

```



### Opción F - Desde Keras (imágenes y series)

**Cuándo usarla:** MNIST, Fashion-MNIST, CIFAR-10, etc.

```python

import tensorflow as tf

# ── Configuración ──────────────────────────────────────────
dataset = tf.keras.datasets.mnist  # ← CAMBIAR: fashion_mnist, cifar10, cifar100, imdb, etc.
# ───────────────────────────────────────────────────────────

(X_train, y_train), (X_test, y_test) = dataset.load_data()

print(f"✅ Dataset cargado")
print(f"Train: {X_train.shape} | Test: {X_test.shape}")

```



### Opción G - URL directa

**Cuándo usarla:** dataset publicado en una URL pública (UCI, GitHub raw, etc.).



```python
# ── Configuración ──────────────────────────────────────────
URL = "https://raw.githubusercontent.com/usuario/repo/main/data.csv"  # ← CAMBIAR
# ───────────────────────────────────────────────────────────

df_raw = pd.read_csv(URL)
print(f"✅ Dataset cargado — Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}")

```



### Opción H — Desde Google Drive

**Cuándo usarla:** el archivo está guardado en tu Google Drive personal (ideal para trabajar en Colab con datasets propios sin subirlos a Kaggle).

```python
from google.colab import drive

# Montar Google Drive (te va a pedir autorización la primera vez)
drive.mount('/content/drive')

# ── Configuración ──────────────────────────────────────────
DRIVE_PATH = "/content/drive/MyDrive/data/archivo.csv"  # ← CAMBIAR: ruta dentro de tu Drive
# ───────────────────────────────────────────────────────────

df_raw = pd.read_csv(DRIVE_PATH)
print(f"✅ Dataset cargado desde Google Drive — Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}")
```
