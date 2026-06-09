# 🔍 Plantilla EDA — Análisis Exploratorio Estándar

> **Variable:** `df_clean` (DataFrame ya limpiado)  
> **Uso:** Copiá cada bloque en una celda separada del notebook  
> **Orden:** Seguí las secciones de arriba hacia abajo

---

## Tabla de Contenidos

1. [Setup](#1-setup)
2. [Carga del dataset limpio](#2-carga-del-dataset-limpio)
3. [Diagnóstico estructural](#3-diagnóstico-estructural)
4. [Variables numéricas — distribución](#4-variables-numéricas--distribución)
5. [Variables numéricas — outliers](#5-variables-numéricas--outliers)
6. [Variables numéricas — correlación](#6-variables-numéricas--correlación)
7. [Variables categóricas — frecuencias](#7-variables-categóricas--frecuencias)
8. [Relación entre variables](#8-relación-entre-variables)
9. [Serie temporal (si hay fecha)](#9-serie-temporal-si-hay-fecha)
10. [Qué gráfico usar según el caso](#10-qué-gráfico-usar-según-el-caso)

---

> **Orden recomendado en todo EDA:**
> 1. Setup
> 2. Carga y diagnóstico estructural ← siempre
> 3. Histograma por cada numérica ← siempre
> 4. Boxplot para outliers ← siempre
> 5. Heatmap de correlación ← siempre si hay 2+ numéricas
> 6. Countplot por cada categórica ← si hay categóricas
> 7. Scatterplot o barplot ← según lo que muestre el heatmap
> 8. Lineplot ← solo si hay fecha

---

## 1. Setup

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
import warnings

# Supresión de warnings
warnings.filterwarnings('ignore')
pd.options.mode.chained_assignment = None

# Configuración de gráficos
sns.set_theme(style='whitegrid')
sns.set_palette('husl')
plt.rcParams['figure.figsize'] = (12, 6)
plt.rcParams['axes.titlesize'] = 16
plt.rcParams['axes.labelsize'] = 13
```

---

## 2. Carga del dataset limpio

```python
df_clean = pd.read_csv('nombre_archivo_limpio.csv')  # ← cambiar nombre
print(f'Dataset cargado: {df_clean.shape[0]} filas x {df_clean.shape[1]} columnas')
```

---

## 3. Diagnóstico estructural

**Esta sección siempre se ejecuta, en todo EDA, sin excepción.**

```python
# Tipos de datos y nulos
df_clean.info()
```

```python
# Estadísticas descriptivas de variables numéricas
df_clean.describe()
```

```python
# Primeras filas
df_clean.head()
```

```python
# Nulos y porcentaje
nulos = pd.DataFrame({
    'Nulos': df_clean.isnull().sum(),
    'Porcentaje': (df_clean.isnull().sum() / len(df_clean) * 100).round(2)
})
display(nulos[nulos['Nulos'] > 0])
```

```python
# Duplicados
print(f'Filas duplicadas: {df_clean.duplicated().sum()}')
```

```python
# Valores únicos por columna
df_clean.nunique().sort_values()
```

---

## 4. Variables numéricas — distribución

**Cuándo usarlo:** siempre. Es el primer gráfico de cualquier variable numérica.  
**Qué buscás:** si los datos están concentrados, dispersos, sesgados o tienen dos grupos.

```python
columnas_numericas = df_clean.select_dtypes(include='number').columns.tolist()

for col in columnas_numericas:
    plt.figure(figsize=(12, 5))
    sns.histplot(data=df_clean, x=col, kde=True, color='steelblue', alpha=0.7)
    plt.title(f'Distribución de {col}')
    plt.xlabel(col)
    plt.tight_layout()
    plt.show()
```

> **Cómo interpretar:**
> - Pico a la izquierda, cola larga a la derecha → sesgo positivo, mayoría de valores bajos
> - Forma de campana simétrica → distribución normal
> - Dos picos → probablemente hay dos grupos distintos en los datos
> - Pico en un extremo → posibles outliers o errores de carga

---

## 5. Variables numéricas — outliers

**Cuándo usarlo:** siempre, después del histograma.  
**Qué buscás:** puntos fuera de los bigotes = outliers estadísticos.

```python
# Boxplot de todas las variables numéricas
plt.figure(figsize=(14, 6))
df_clean[columnas_numericas].boxplot()
plt.title('Detección de outliers — Variables numéricas')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

```python
# Boxplot individual por categoría
# Reemplazá las columnas con las tuyas

plt.figure(figsize=(12, 6))
sns.boxplot(
    data=df_clean,
    x='COLUMNA_NUMERICA',     # ← reemplazar
    y='COLUMNA_CATEGORICA',   # ← reemplazar
    hue='COLUMNA_CATEGORICA',
    legend=False,
    palette='Set2',
    flierprops={'marker': 'o', 'markerfacecolor': 'red', 'markersize': 7}
)
plt.title('Distribución por categoría')
plt.tight_layout()
plt.show()
```

> **Cómo leer un boxplot:**
> ```
> |── bigote ──[  Q1 │ MEDIANA │ Q3  ]── bigote ──|  ● outlier
>               25%              75%
> ```
> - Caja = 50% central de los datos
> - Línea del medio = mediana
> - Puntos rojos sueltos = outliers → investigar si son errores o casos reales

---

## 6. Variables numéricas — correlación

**Cuándo usarlo:** siempre que tengas 2 o más variables numéricas.  
**Qué buscás:** qué variables se mueven juntas. Especialmente útil antes de modelado.

```python
plt.figure(figsize=(10, 8))
correlacion = df_clean[columnas_numericas].corr()

sns.heatmap(
    correlacion,
    annot=True,
    fmt='.2f',
    cmap='coolwarm',
    center=0,
    linewidths=0.5,
    square=True
)
plt.title('Correlación entre variables numéricas')
plt.tight_layout()
plt.show()
```

> **Cómo interpretar:**
> - Cercano a `1.0` → correlación positiva fuerte
> - Cercano a `-1.0` → correlación negativa fuerte
> - Cercano a `0.0` → sin correlación
> - Rojo o azul intenso → relación que vale investigar más

---

## 7. Variables categóricas — frecuencias

**Cuándo usarlo:** siempre que tengas columnas de texto o categorías.  
**Qué buscás:** distribución de categorías, si hay categorías dominantes o muy raras.

```python
# Ver valores únicos de cada categórica
columnas_categoricas = df_clean.select_dtypes(include='object').columns.tolist()
for col in columnas_categoricas:
    print(f'\n{col}: {df_clean[col].nunique()} valores únicos')
    print(df_clean[col].value_counts().head(10))
```

```python
# Barras horizontal — cuando las etiquetas son largas (más de 8 caracteres)
plt.figure(figsize=(12, 6))
p = sns.countplot(
    data=df_clean,
    y='COLUMNA_CATEGORICA',   # ← reemplazar
    hue='COLUMNA_CATEGORICA',
    legend=False,
    order=df_clean['COLUMNA_CATEGORICA'].value_counts().index,
    palette='Blues_r'
)
for container in p.containers:
    p.bar_label(container, label_type='edge', padding=5)
plt.title('Frecuencia por categoría')
plt.tight_layout()
plt.show()
```

```python
# Barras vertical — cuando las etiquetas son cortas y pocas categorías
plt.figure(figsize=(12, 6))
p = sns.countplot(
    data=df_clean,
    x='COLUMNA_CATEGORICA',   # ← reemplazar
    hue='COLUMNA_CATEGORICA',
    legend=False,
    order=df_clean['COLUMNA_CATEGORICA'].value_counts().index,
    palette='husl'
)
for container in p.containers:
    p.bar_label(container, label_type='edge', padding=5)
plt.title('Frecuencia por categoría')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

> **Horizontal vs vertical:**
> - Etiquetas largas (más de 8 caracteres) → horizontal con `y=`
> - Pocas categorías con etiquetas cortas → vertical con `x=`

---

## 8. Relación entre variables

### 8.1 Numérica vs numérica → Scatterplot

**Cuándo usarlo:** cuando el heatmap muestra correlación fuerte y querés verla visualmente.

```python
plt.figure(figsize=(12, 6))
sns.scatterplot(
    data=df_clean,
    x='COLUMNA_NUMERICA_1',   # ← reemplazar
    y='COLUMNA_NUMERICA_2',   # ← reemplazar
    hue='COLUMNA_CATEGORICA', # ← reemplazar (opcional)
    alpha=0.6
)
plt.title('Relación entre variables')
plt.tight_layout()
plt.show()
```

> - Diagonal ascendente → correlación positiva
> - Diagonal descendente → correlación negativa
> - Nube dispersa → sin correlación

### 8.2 Numérica vs categórica → Barplot de métrica

**Cuándo usarlo:** cuando querés comparar suma o promedio entre categorías.

```python
orden = df_clean.groupby('COLUMNA_CATEGORICA')['COLUMNA_NUMERICA']\
                .sum().sort_values(ascending=False).index  # ← reemplazar

plt.figure(figsize=(12, 6))
sns.barplot(
    data=df_clean,
    x='COLUMNA_CATEGORICA',   # ← reemplazar
    y='COLUMNA_NUMERICA',     # ← reemplazar
    estimator=sum,            # cambiar a mean para promedio
    order=orden,
    hue='COLUMNA_CATEGORICA',
    legend=False,
    palette='Blues_r'
)
plt.title('Métrica por categoría')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

### 8.3 Todas las numéricas → Pairplot

**Cuándo usarlo:** al inicio del EDA cuando no sabés qué relaciones existen. Máximo 5-6 columnas.

```python
cols_pairplot = ['COL1', 'COL2', 'COL3']  # ← reemplazar con tus columnas numéricas

sns.pairplot(
    df_clean[cols_pairplot],
    diag_kind='kde',
    plot_kws={'alpha': 0.5}
)
plt.suptitle('Relaciones entre variables numéricas', y=1.02)
plt.show()
```

---

## 9. Serie temporal (si hay fecha)

**Cuándo usarlo:** solo si el dataset tiene una columna de fecha.

```python
# Asegurar tipo datetime
df_clean['COLUMNA_FECHA'] = pd.to_datetime(df_clean['COLUMNA_FECHA'])  # ← reemplazar

# Agrupar por mes
serie = df_clean.groupby(
    df_clean['COLUMNA_FECHA'].dt.to_period('M')
)['COLUMNA_NUMERICA'].sum().reset_index()  # ← reemplazar columnas
serie['COLUMNA_FECHA'] = serie['COLUMNA_FECHA'].astype(str)

plt.figure(figsize=(14, 6))
sns.lineplot(data=serie, x='COLUMNA_FECHA', y='COLUMNA_NUMERICA', marker='o')
plt.title('Evolución temporal')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

---

## 10. Qué gráfico usar según el caso

| Tenés... | Querés saber... | Usá |
|----------|-----------------|-----|
| 1 variable numérica | Cómo se distribuye | Histograma |
| 1 variable numérica | Si hay outliers | Boxplot |
| 2+ variables numéricas | Si están relacionadas | Heatmap de correlación |
| 2 numéricas específicas | Cómo se relacionan visualmente | Scatterplot |
| Muchas numéricas | Ver todas las relaciones de una | Pairplot |
| 1 variable categórica | Cuántos hay por categoría | Countplot |
| 1 categórica + 1 numérica | Métrica por categoría (suma, media) | Barplot |
| 1 numérica + 1 categórica | Dispersión y outliers por grupo | Boxplot agrupado |
| 1 fecha + 1 numérica | Cómo evoluciona en el tiempo | Lineplot |
| 2 categóricas + 1 numérica | Cruzar dos dimensiones | Heatmap de pivot |

*Para variantes avanzadas de cada gráfico → `visualizacion_datos.md`*
