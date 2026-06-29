## 2. Carga del dataset



### Opción A — Plantilla con Fallback Automático



```python

# ── Configuración ──────────────────────────────────────────
DATASET_KAGGLE_PROPIO   = "andrssonsinogrugni/nombre-dataset"  # ← CAMBIAR: tu dataset en Kaggle
DATASET_KAGGLE_EXTERNO  = "mlg-ulb/creditcardfraud"            # ← CAMBIAR: dataset de otro usuario
DATA_PATH_LOCAL         = r"data/archivo.csv"                  # ← CAMBIAR: ruta local
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



### Opción B — Desde archivo local



```python

# ── Configuración ──────────────────────────────────────────
DATA_PATH = r"data/archivo.csv"  # ← CAMBIAR: ruta al archivo
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

