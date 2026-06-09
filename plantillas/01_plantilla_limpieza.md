# 🧹 Plantilla Limpieza de Datos — Estándar de Industria

> **Variable:** `df_raw` → `df_clean` (entrada sucia, salida limpia)  
> **Uso:** Copiá cada bloque en una celda separada del notebook  
> **Orden:** Seguí las secciones de arriba hacia abajo  
> **Salida:** archivo `_limpio.csv` listo para el notebook de EDA

---

## Tabla de Contenidos

1. [Setup](#1-setup)
2. [Carga del dataset](#2-carga-del-dataset)
3. [Diagnóstico inicial](#3-diagnóstico-inicial)
4. [Duplicados](#4-duplicados)
5. [Valores nulos (NaN)](#5-valores-nulos-nan)
6. [Tipos de datos](#6-tipos-de-datos)
7. [Inconsistencias en texto](#7-inconsistencias-en-texto)
8. [Outliers](#8-outliers)
9. [Reset de índice](#9-reset-de-índice)
10. [Verificación final](#10-verificación-final)
11. [Exportar dataset limpio](#11-exportar-dataset-limpio)
12. [Cuándo usar cada estrategia](#12-cuándo-usar-cada-estrategia)

---

> **Orden fijo de limpieza — nunca cambiar:**
> 1. Diagnóstico inicial ← fotografía del estado original
> 2. Duplicados ← primero, antes de contar nulos
> 3. Nulos ← después de sacar duplicados
> 4. Tipos de datos ← después de nulos (un NaN impide convertir a int)
> 5. Inconsistencias de texto ← después de tipos
> 6. Outliers ← cuando el dataset ya está estructuralmente limpio
> 7. Reset de índice ← siempre al final
> 8. Verificación y exportación

---
## 1. Setup

```python
import kagglehub
import pandas as pd
import numpy as np
import os
import warnings

# Supresión de warnings
warnings.filterwarnings('ignore')
pd.options.mode.chained_assignment = None

# Configuración de visualización del DataFrame
pd.set_option('display.max_columns', None)   # muestra todas las columnas
pd.set_option('display.max_rows', 100)
pd.set_option('display.float_format', '{:.2f}'.format)
```

---

## 2. Carga del dataset

### Opción A — Plantilla con Fallback Automático

```python
# ── Configuración ──────────────────────────────────────────
DATASET_KAGGLE_PROPIO   = "andrssonsinogrugni/nombre-dataset"  # ← CAMBIAR: tu dataset en Kaggle
DATASET_KAGGLE_EXTERNO  = "mlg-ulb/creditcardf_rawraud"           # ← CAMBIAR: dataset de otro usuario
DATA_PATH_LOCAL         = r"data/archivo.csv"                 # ← CAMBIAR: ruta local
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
DATASET_KAGGLE = "mlg-ulb/creditcardf_rawraud"  # ← CAMBIAR: usuario/nombre-dataset
# ───────────────────────────────────────────────────────────

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
---

## 3. Diagnóstico inicial

**Ejecutar completo antes de tocar cualquier dato. Nunca saltear esta sección.**

```python
# ─── Primeras y últimas filas ──────────────────────────────────────────────
print('\n--- Primeras 5 filas ---')
display(df_raw.head())

print('\n--- Últimas 5 filas ---')
display(df_raw.tail())

# ─── Dimensiones ───────────────────────────────────────────────────────────
print(f'Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}')

# ─── Tipos de datos ────────────────────────────────────────────────────────
print('\n--- Info general ---')
df_raw.info()

# ─── Valores nulos por columna ─────────────────────────────────────────────
print('\n--- Nulos por columna ---')
# Verificamos si hay algún nulo en toda la tabla
if df_raw.isnull().sum().sum() == 0:
    print('✅ Sin valores nulos')
else:
    # Sino mostramos tabla filtrada de nulos
    nulos = pd.DataFrame({
        'Nulos': df_raw.isnull().sum(),
        'Porcentaje': (df_raw.isnull().sum() / len(df_raw) * 100).round(2)
    })
    display(nulos[nulos['Nulos'] > 0])

# ─── Duplicados ────────────────────────────────────────────────────────────
print(f'\nFilas duplicadas: {df_raw.duplicated().sum()}')

# ─── Estadísticas descriptivas ─────────────────────────────────────────────
print('\n--- Estadísticas descriptivas ---')
display(df_raw.describe())
```

---

## 4. Duplicados

**Hacer siempre. Antes de cualquier otra limpieza.**

```python
# Ver cuántos hay
print(f'Duplicados antes: {df_raw.duplicated().sum()}')

# Ver las filas duplicadas (opcional)
display(df_raw[df_raw.duplicated(keep=False)].head(10))
```

```python
# Eliminar duplicados
df_raw = df_raw.drop_duplicates()

# Verificar
print(f'Duplicados después: {df_raw.duplicated().sum()}')
print(f'Filas restantes: {df_raw.shape[0]}')
```

> **Variantes:**
> ```python
> # Duplicados por columnas específicas (no fila completa)
> df_raw = df_raw.drop_duplicates(subset=['COL1', 'COL2'])  # ← reemplazar
>
> # Mantener última ocurrencia en lugar de primera
> df_raw = df_raw.drop_duplicates(keep='last')
> ```

---

## 5. Valores nulos (NaN)

**La estrategia depende del porcentaje de nulos y del tipo de columna.**

```python
# Ver estado actual de nulos
nulos = pd.DataFrame({
    'Nulos': df_raw.isnull().sum(),
    'Porcentaje': (df_raw.isnull().sum() / len(df_raw) * 100).round(2),
    'Tipo': df_raw.dtypes
})
display(nulos[nulos['Nulos'] > 0].sort_values('Porcentaje', ascending=False))
```

```python
# ── Imputar con MEDIANA ────────────────────────────────────────────────────
# Cuándo: columnas numéricas con outliers (precios, salarios, satisfacción)
mediana = df_raw['COLUMNA'].median()                          # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna(mediana)
print(f'Mediana usada: {mediana}')
```

```python
# ── Imputar con MEDIA ──────────────────────────────────────────────────────
# Cuándo: columnas numéricas sin outliers (temperatura, peso, edad)
media = df_raw['COLUMNA'].mean()                              # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna(media)
print(f'Media usada: {media:.2f}')
```

```python
# ── Imputar con MODA ───────────────────────────────────────────────────────
# Cuándo: columnas categóricas (ciudad, categoría, estado)
moda = df_raw['COLUMNA'].mode()[0]                            # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna(moda)
print(f'Moda usada: {moda}')
```

```python
# ── Imputar con valor fijo ─────────────────────────────────────────────────
# Cuándo: cuando "sin valor" tiene un significado específico
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna('Desconocido')   # ← reemplazar
df_raw['COLUMNA_NUM'] = df_raw['COLUMNA_NUM'].fillna(0)       # ← reemplazar
```

```python
# ── Eliminar filas con nulos ───────────────────────────────────────────────
# Cuándo: menos del 5% de nulos en una columna crítica
df_raw = df_raw.dropna(subset=['COLUMNA_CRITICA'])            # ← reemplazar
```

```python
# ── Eliminar columna entera ────────────────────────────────────────────────
# Cuándo: más del 50% de nulos y la columna no aporta valor
df_raw = df_raw.drop(columns=['COLUMNA_INUTILIZABLE'])        # ← reemplazar
```

```python
# Verificar que no quedan nulos
print(df_raw.isnull().sum())
```

---

## 6. Tipos de datos

**Hacer después de limpiar nulos. Un nulo en una columna impide convertir a int.**

```python
# Ver tipos actuales
print(df_raw.dtypes)
```

```python
# ── Fechas ────────────────────────────────────────────────────────────────
# Cuándo: columnas que son fechas pero llegaron como texto (object)
df_raw['COLUMNA_FECHA'] = pd.to_datetime(df_raw['COLUMNA_FECHA'])              # ← reemplazar
df_raw['COLUMNA_FECHA'] = pd.to_datetime(df_raw['COLUMNA_FECHA'],
                                          format='%d/%m/%Y')   # formato específico
df_raw['COLUMNA_FECHA'] = pd.to_datetime(df_raw['COLUMNA_FECHA'],
                                          errors='coerce')     # NaN si no puede parsear
```

```python
# ── Numéricos ────────────────────────────────────────────────────────────
df_raw['COLUMNA'] = pd.to_numeric(df_raw['COLUMNA'], errors='coerce')  # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].astype(int)
df_raw['COLUMNA'] = df_raw['COLUMNA'].astype(float)
```

```python
# ── Numérico con formato texto ($, puntos, comas) ────────────────────────
# Cuándo: columna tiene '$1.234,56' en lugar de 1234.56
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.replace('$', '', regex=False)  # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.replace('.', '', regex=False)
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.replace(',', '.', regex=False)
df_raw['COLUMNA'] = pd.to_numeric(df_raw['COLUMNA'], errors='coerce')
```

```python
# ── Categórico ───────────────────────────────────────────────────────────
# Cuándo: columna tiene pocos valores únicos repetidos (ahorra memoria)
df_raw['COLUMNA'] = df_raw['COLUMNA'].astype('category')                # ← reemplazar

# Categórico con orden lógico (Baja < Media < Alta)
df_raw['COLUMNA'] = pd.Categorical(
    df_raw['COLUMNA'],
    categories=['Baja', 'Media', 'Alta'],                               # ← reemplazar
    ordered=True
)
```

```python
# Verificar tipos después de convertir
print(df_raw.dtypes)
```

---

## 7. Inconsistencias en texto

**Hacer en columnas categóricas antes de análisis o agrupaciones.**

```python
# ── Espacios extra ────────────────────────────────────────────────────────
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.strip()                       # ← reemplazar

# ── Normalizar mayúsculas ────────────────────────────────────────────────
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.title()   # Primera Letra Mayúscula
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.upper()   # TODO MAYÚSCULAS
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.lower()   # todo minúsculas
```

```python
# ── Reemplazar valores mal escritos ─────────────────────────────────────
reemplazos = {
    'VALOR_MAL': 'VALOR_CORRECTO',    # ← reemplazar
    'VALOR_MAL_2': 'VALOR_CORRECTO',
}
df_raw['COLUMNA'] = df_raw['COLUMNA'].replace(reemplazos)               # ← reemplazar
```

```python
# ── Verificar resultado ──────────────────────────────────────────────────
print(df_raw['COLUMNA'].value_counts())                                 # ← reemplazar
```

---

## 8. Outliers

**Detectar siempre. Tratar según contexto de negocio.**

```python
# Detectar con IQR
def detectar_outliers(df, columna):
    Q1 = df[columna].quantile(0.25)
    Q3 = df[columna].quantile(0.75)
    IQR = Q3 - Q1
    lim_inf = Q1 - 1.5 * IQR
    lim_sup = Q3 + 1.5 * IQR
    outliers = df[(df[columna] < lim_inf) | (df[columna] > lim_sup)]
    print(f'{columna}:')
    print(f'  Rango normal: [{lim_inf:.2f}, {lim_sup:.2f}]')
    print(f'  Outliers: {len(outliers)} ({len(outliers)/len(df)*100:.1f}%)\n')
    return lim_inf, lim_sup

lim_inf, lim_sup = detectar_outliers(df_raw, 'COLUMNA_NUMERICA')       # ← reemplazar
```

```python
# ── Opción A: Eliminar filas con outliers ────────────────────────────────
# Cuándo: son errores de carga (precio = 0, edad = 999)
df_raw = df_raw[
    (df_raw['COLUMNA'] >= lim_inf) &                                    # ← reemplazar
    (df_raw['COLUMNA'] <= lim_sup)
]
```

```python
# ── Opción B: Capping (reemplazar por el límite) ─────────────────────────
# Cuándo: los valores extremos son reales pero distorsionan el análisis
df_raw['COLUMNA'] = df_raw['COLUMNA'].clip(                             # ← reemplazar
    lower=lim_inf,
    upper=lim_sup
)
```

```python
# ── Opción C: Dejar los outliers ─────────────────────────────────────────
# Cuándo: son casos reales con sentido de negocio
# (una venta muy grande es real, no un error)
# → Documentar la decisión en celda de texto
```

> **Regla para decidir:**
> - El valor es físicamente imposible (edad = -5, precio = 0) → **eliminar**
> - El valor es extremo pero posible (venta de $10M en retail) → **dejar o hacer capping**
> - No sabés → **dejar y documentar**, el EDA lo va a revelar

---

## 9. Reset de índice

**Hacer siempre al final, después de todas las eliminaciones.**

```python
# Ver si el índice tiene huecos
print(f'Índice antes del reset: {df_raw.index.tolist()[:10]}...')

# Resetear
df_raw = df_raw.reset_index(drop=True)

# Verificar
print(f'Índice después del reset: {df_raw.index.tolist()[:10]}...')
print(f'Filas finales: {df_raw.shape[0]}')
```

> `drop=True` descarta el índice viejo. Sin `drop=True`, el índice viejo se convierte en una columna nueva — casi nunca es lo que querés.

---

## 10. Verificación final

**Ejecutar completo antes de exportar. Confirmar que el dataset está limpio.**

```python
print('=' * 50)
print('REPORTE FINAL DE CALIDAD')
print('=' * 50)
print(f'Filas finales:      {df_raw.shape[0]}')
print(f'Columnas:           {df_raw.shape[1]}')
print(f'Duplicados:         {df_raw.duplicated().sum()}')
print(f'Nulos totales:      {df_raw.isnull().sum().sum()}')
print('=' * 50)
```

```python
# Tipos de datos finales
print(df_raw.dtypes)
```

```python
# Vista previa del dataset limpio
df_raw.head(10)
```

---

## 11. Exportar dataset limpio

**Último paso. El archivo exportado es la entrada del notebook de EDA.**

```python
# Renombrar a df_clean antes de exportar
df_clean = df_raw.copy()

# Exportar
nombre_salida = 'NOMBRE_DATASET_limpio.csv'                             # ← reemplazar
df_clean.to_csv(nombre_salida, index=False, encoding='utf-8')
print(f'Dataset exportado: {nombre_salida}')
print(f'Filas: {df_clean.shape[0]} | Columnas: {df_clean.shape[1]}')
```

```python
# Verificar que el archivo se guardó correctamente
verificacion = pd.read_csv(nombre_salida)
print(f'Verificación: {verificacion.shape[0]} filas x {verificacion.shape[1]} columnas')
```

```python
# Solo en Colab — descargar el archivo a tu PC
# from google.colab import files
# files.download(nombre_salida)
```

---

## 12. Cuándo usar cada estrategia

### Nulos

| % de nulos | Tipo de columna | Estrategia |
|-----------|-----------------|-----------|
| < 5% | Numérica o categórica | Eliminar las filas |
| 5% - 30% | Numérica con outliers | Imputar con mediana |
| 5% - 30% | Numérica sin outliers | Imputar con media |
| 5% - 30% | Categórica | Imputar con moda |
| > 50% | Cualquiera | Evaluar eliminar la columna |

### Outliers

| Situación | Estrategia |
|-----------|-----------|
| Valor imposible (precio = 0, edad negativa) | Eliminar fila |
| Valor extremo pero posible | Capping o dejar |
| No sabés si es error o caso real | Dejar y documentar |

### Tipos de datos

| La columna tiene... | Convertir a... |
|--------------------|----------------|
| Fechas como texto | `pd.to_datetime()` |
| Números como texto | `pd.to_numeric()` |
| Números con `$` o `.` | Limpiar string → `pd.to_numeric()` |
| Texto con pocos valores únicos | `astype('category')` |
| Texto con orden lógico | `pd.Categorical(..., ordered=True)` |

---

*Para el análisis exploratorio del dataset limpio → `plantilla_eda.md`*
