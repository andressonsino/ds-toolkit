# 🔍 Plantilla EDA — Análisis Exploratorio Estándar

> **Variable:** `df_clean` (DataFrame ya limpiado)  
> **Uso:** Copiá cada bloque en una celda separada del notebook  
> **Orden:** Seguí las secciones de arriba hacia abajo

---

## Tabla de Contenidos

1. [Setup](#1-setup)
2. [Enriquecimiento manual de columnas](#2-enriquecimiento-manual-de-columnas)
3. [Diagnóstico estructural](#3-diagnóstico-estructural)
4. [Variables numéricas — distribución en grilla](#4-variables-numéricas--distribución-en-grilla)
5. [Variables numéricas — outliers](#5-variables-numéricas--outliers)
6. [Variables numéricas — correlación](#6-variables-numéricas--correlación)
7. [Variables categóricas — frecuencias](#7-variables-categóricas--frecuencias)
8. [Relación entre variables](#8-relación-entre-variables)
9. [Gráficos que marcan la diferencia](#9-gráficos-que-marcan-la-diferencia)
10. [Serie temporal (si hay fecha)](#10-serie-temporal-si-hay-fecha)
11. [Qué gráfico usar según el caso](#11-qué-gráfico-usar-según-el-caso)

---

> **Orden recomendado en todo EDA:**
> 1. Setup
> 2. Carga y diagnóstico estructural ← siempre
> 3. Grilla de distribuciones ← siempre
> 4. Boxplot para outliers ← siempre
> 5. Heatmap de correlación ← siempre si hay 2+ numéricas
> 6. Countplot por cada categórica ← si hay categóricas
> 7. Scatterplot o barplot ← según lo que muestre el heatmap
> 8. Gráficos diferenciales ← según el dataset y el objetivo
> 9. Lineplot ← solo si hay fecha

---

## 1. Setup

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
import warnings
import math

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

## 2. Enriquecimiento manual de columnas

**Cuándo:** una columna es relevante para el análisis pero tiene nulos que podés completar
con investigación externa (Transfermarkt, Wikipedia, etc.).
**Por qué acá y no en limpieza:** es una decisión analítica, no una corrección de datos sucios.

```python
# ── Completar valores manualmente ────────────────────────────────────────
# Usar códigos estándar según el tipo de dato:
# Nacionalidades → ISO 3166-1 alpha-2 (AR, BR, UY, EC, etc.)
# Documentar la fuente de consulta en el comentario

valores_manuales = {
    'NOMBRE_1': 'VALOR',    # ← reemplazar
    'NOMBRE_2': 'VALOR',
}

# ── Columna de búsqueda y columna a completar ─────────────────────────────
COL_BUSQUEDA  = 'COLUMNA_NOMBRE'   # ← reemplazar: columna donde buscar el nombre
COL_COMPLETAR = 'COLUMNA_NULOS'    # ← reemplazar: columna a completar

for nombre, valor in valores_manuales.items():
    mask = df_clean[COL_BUSQUEDA].str.contains(nombre, na=False)
    if mask.sum() == 0:
        print(f'⚠️  No se encontró: {nombre}')
    else:
        df_clean.loc[mask, COL_COMPLETAR] = valor

# ── Verificar cobertura ───────────────────────────────────────────────────
total       = len(df_clean)
completados = df_clean[COL_COMPLETAR].notna().sum()
print(f'Cobertura: {completados}/{total} ({completados/total*100:.1f}%)')
display(df_clean[[COL_BUSQUEDA, COL_COMPLETAR]])
```

> **Fuente de consulta:** documentar siempre de dónde sacaste los valores
> ```python
> # Fuente: Transfermarkt — https://www.transfermarkt.com.ar/...
> # Fecha de consulta: DD/MM/AAAA
> ```

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
    'Nulos':       df_clean.isnull().sum(),
    'Porcentaje':  (df_clean.isnull().sum() / len(df_clean) * 100).round(2)
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

## 4. Variables numéricas — distribución en grilla

**Cuándo usarlo:** siempre. Reemplaza el loop de gráficos individuales por una grilla compacta y profesional.  
**Por qué grilla y no loop:** un gráfico por celda para cada columna genera decenas de outputs inconexos que no permiten comparar. La grilla muestra todo junto, permite detectar patrones entre variables de un vistazo, y es lo que se usa en un reporte real.

```python
columnas_numericas = df_clean.select_dtypes(include='number').columns.tolist()

n_cols = 3
n_rows = math.ceil(len(columnas_numericas) / n_cols)

fig, axes = plt.subplots(n_rows, n_cols, figsize=(18, n_rows * 4))
axes = axes.flatten()

for i, col in enumerate(columnas_numericas):
    sns.histplot(
        data=df_clean,
        x=col,
        kde=True,
        ax=axes[i],
        color='steelblue',
        alpha=0.7,
        edgecolor='white'
    )
    axes[i].set_title(col, fontsize=13, fontweight='bold')
    axes[i].set_xlabel('')
    axes[i].grid(False)

# Ocultar subplots vacíos si el número de columnas no es múltiplo de n_cols
for j in range(i + 1, len(axes)):
    axes[j].set_visible(False)

fig.suptitle('Distribución de variables numéricas', fontsize=16, fontweight='bold', y=1.01)
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

**Cuándo usarlo:** siempre, después de la grilla de distribuciones.  
**Qué buscás:** puntos fuera de los bigotes = outliers estadísticos.

```python
# Boxplot de todas las variables numéricas en grilla
n_cols = 3
n_rows = math.ceil(len(columnas_numericas) / n_cols)

fig, axes = plt.subplots(n_rows, n_cols, figsize=(18, n_rows * 4))
axes = axes.flatten()

for i, col in enumerate(columnas_numericas):
    axes[i].boxplot(
        df_clean[col].dropna(),
        patch_artist=True,
        boxprops=dict(facecolor='skyblue', color='navy'),
        whiskerprops=dict(color='navy'),
        capprops=dict(color='navy'),
        medianprops=dict(color='darkred', linewidth=2),
        flierprops=dict(marker='o', markerfacecolor='red', markersize=5, alpha=0.5)
    )
    axes[i].set_title(col, fontsize=13, fontweight='bold')
    axes[i].grid(False)

for j in range(i + 1, len(axes)):
    axes[j].set_visible(False)

fig.suptitle('Detección de outliers — Variables numéricas', fontsize=16, fontweight='bold', y=1.01)
plt.tight_layout()
plt.show()
```

```python
# Boxplot individual por categoría — cuando querés ver outliers por grupo
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
plt.grid(False)
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
    square=True,
)
plt.title('Correlación entre variables numéricas')
plt.tight_layout()
plt.grid(False)
plt.show()
```

> **Cómo interpretar:**
> - Cercano a `1.0` → correlación positiva fuerte
> - Cercano a `-1.0` → correlación negativa fuerte
> - Cercano a `0.0` → sin correlación
> - Rojo o azul intenso → relación que vale investigar con un scatterplot

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
plt.grid(False)
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
plt.grid(False)
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
    hue='COLUMNA_CATEGORICA', # ← reemplazar (opcional, para ver grupos)
    alpha=0.6
)
plt.title('Relación entre variables')
plt.grid(False)
plt.tight_layout()
plt.show()
```

### 8.2 Numérica vs categórica → Barplot de métrica

**Cuándo usarlo:** cuando querés comparar suma o promedio entre categorías.

```python
orden = df_clean.groupby('COLUMNA_CATEGORICA')['COLUMNA_NUMERICA'] \
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
plt.grid(False)
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

## 9. Gráficos que marcan la diferencia

Estos gráficos no son los que todos usan. Los agregás cuando el dataset y el objetivo lo justifican — no como decoración, sino porque responden preguntas que los convencionales no pueden.

---

### 9.1 Violinplot — distribución + densidad por grupo

**Cuándo:** tenés una variable numérica y querés ver cómo se distribuye dentro de cada categoría, incluyendo la forma completa (no solo la caja).  
**Qué aporta sobre el boxplot:** el boxplot muestra los cuartiles; el violín muestra dónde se concentran realmente los datos — si hay un solo pico, dos, si está sesgado.

```python
plt.figure(figsize=(14, 6))
sns.violinplot(
    data=df_clean,
    x='COLUMNA_CATEGORICA',   # ← reemplazar
    y='COLUMNA_NUMERICA',     # ← reemplazar
    hue='COLUMNA_CATEGORICA',
    legend=False,
    palette='Set2',
    inner='box'               # muestra la caja del boxplot adentro del violín
)
plt.title('Distribución por categoría — Violinplot')
plt.xticks(rotation=45, ha='right')
plt.grid(False)
plt.tight_layout()
plt.show()
```

---

### 9.2 Stripplot sobre Boxplot — ver los puntos reales

**Cuándo:** el dataset no es muy grande (menos de 2000 filas) y querés mostrar cada dato real encima del boxplot.  
**Qué aporta:** hace visible si hay clusters de datos, si los outliers son aislados o si hay muchos, y cuántos registros hay realmente en cada grupo.

```python
plt.figure(figsize=(14, 6))
sns.boxplot(
    data=df_clean,
    x='COLUMNA_CATEGORICA',   # ← reemplazar
    y='COLUMNA_NUMERICA',     # ← reemplazar
    hue='COLUMNA_CATEGORICA',
    legend=False,
    palette='pastel',
    flierprops={'marker': ''}  # oculta outliers del boxplot porque el strip los muestra
)
sns.stripplot(
    data=df_clean,
    x='COLUMNA_CATEGORICA',   # ← reemplazar
    y='COLUMNA_NUMERICA',     # ← reemplazar
    color='black',
    alpha=0.35,
    size=4,
    jitter=True               # separa los puntos para que no se apilen
)
plt.title('Distribución real por categoría')
plt.xticks(rotation=45, ha='right')
plt.grid(False)
plt.tight_layout()
plt.show()
```

---

### 9.3 KDE por grupo — comparar distribuciones entre categorías

**Cuándo:** tenés una variable numérica y querés comparar cómo se distribuye en dos o más grupos en la misma escala.  
**Qué aporta sobre el histograma:** superpone las curvas sin barras, lo que hace mucho más fácil ver si los grupos tienen distribuciones distintas, similares o si se solapan.

```python
plt.figure(figsize=(12, 6))
for grupo in df_clean['COLUMNA_CATEGORICA'].unique():    # ← reemplazar
    subset = df_clean[df_clean['COLUMNA_CATEGORICA'] == grupo]
    sns.kdeplot(
        data=subset,
        x='COLUMNA_NUMERICA',                            # ← reemplazar
        label=str(grupo),
        fill=True,
        alpha=0.25,
        linewidth=2
    )
plt.title('Distribución por grupo — KDE')
plt.legend(title='COLUMNA_CATEGORICA')                   # ← reemplazar
plt.grid(False)
plt.tight_layout()
plt.show()
```

---

### 9.4 Heatmap de tabla pivote — cruzar dos categóricas con una métrica

**Cuándo:** tenés dos variables categóricas y querés ver cómo se comporta una métrica numérica en cada combinación.  
**Qué aporta:** reemplaza una tabla de números difícil de leer por un mapa de calor donde los patrones se ven de un vistazo.

```python
pivot = df_clean.pivot_table(
    values='COLUMNA_NUMERICA',        # ← reemplazar: la métrica que querés ver
    index='COLUMNA_CATEGORICA_1',     # ← reemplazar: filas
    columns='COLUMNA_CATEGORICA_2',   # ← reemplazar: columnas
    aggfunc='mean'                    # cambiar a sum, median, count según el caso
)

plt.figure(figsize=(14, 8))
sns.heatmap(
    pivot,
    annot=True,
    fmt='.1f',
    cmap='YlOrRd',
    linewidths=0.5
)
plt.title('Métrica promedio por combinación de categorías')
plt.tight_layout()
plt.show()
```

---

### 9.5 Barras apiladas porcentuales — composición dentro de cada grupo

**Cuándo:** tenés dos variables categóricas y querés ver la proporción de una dentro de la otra.  
**Qué aporta:** mientras el countplot muestra volumen absoluto, las barras apiladas al 100% muestran composición — cuánto representa cada subcategoría dentro de cada grupo.

```python
# Tabla de frecuencias cruzadas normalizada por fila
tabla = pd.crosstab(
    df_clean['COLUMNA_CATEGORICA_1'],   # ← reemplazar: eje X
    df_clean['COLUMNA_CATEGORICA_2'],   # ← reemplazar: subcategorías (colores)
    normalize='index'                   # normaliza por fila → porcentajes
) * 100

tabla.plot(
    kind='bar',
    stacked=True,
    figsize=(14, 6),
    colormap='Set2',
    edgecolor='white',
    width=0.7
)
plt.title('Composición porcentual por grupo')
plt.ylabel('Porcentaje (%)')
plt.xlabel('')
plt.xticks(rotation=45, ha='right')
plt.legend(title='COLUMNA_CATEGORICA_2', bbox_to_anchor=(1.05, 1))  # ← reemplazar
plt.grid(False)
plt.tight_layout()
plt.show()
```

---

### 9.6 Scatterplot con regresión — tendencia entre dos numéricas

**Cuándo:** el heatmap muestra correlación fuerte entre dos variables y querés mostrar la tendencia con una línea.  
**Qué aporta:** el scatterplot muestra los puntos; `regplot` agrega la línea de tendencia y la banda de confianza al 95% — comunica correlación de forma mucho más clara.

```python
plt.figure(figsize=(12, 6))
sns.regplot(
    data=df_clean,
    x='COLUMNA_NUMERICA_1',   # ← reemplazar
    y='COLUMNA_NUMERICA_2',   # ← reemplazar
    scatter_kws={'alpha': 0.4, 'color': 'steelblue', 's': 40},
    line_kws={'color': 'darkred', 'linewidth': 2},
    ci=95                     # banda de confianza al 95%
)
plt.title('Tendencia: COLUMNA_1 vs COLUMNA_2')   # ← reemplazar
plt.grid(False)
plt.tight_layout()
plt.show()
```

---

## 10. Serie temporal (si hay fecha)

**Cuándo usarlo:** solo si el dataset tiene una columna de fecha.

```python
# Asegurar tipo datetime
df_clean['COLUMNA_FECHA'] = pd.to_datetime(df_clean['COLUMNA_FECHA'])  # ← reemplazar

# Agrupar por mes
serie = df_clean.groupby(
    df_clean['COLUMNA_FECHA'].dt.to_period('M')
)['COLUMNA_NUMERICA'].sum().reset_index()   # ← reemplazar columnas
serie['COLUMNA_FECHA'] = serie['COLUMNA_FECHA'].astype(str)

plt.figure(figsize=(14, 6))
sns.lineplot(data=serie, x='COLUMNA_FECHA', y='COLUMNA_NUMERICA', marker='o')
plt.title('Evolución temporal')
plt.xticks(rotation=45, ha='right')
plt.grid(False)
plt.tight_layout()
plt.show()
```

---

## 11. Qué gráfico usar según el caso

| Tenés... | Querés saber... | Usá |
|----------|-----------------|-----|
| 1 variable numérica | Cómo se distribuye | Histograma (grilla) |
| 1 variable numérica | Si hay outliers | Boxplot (grilla) |
| 2+ variables numéricas | Si están relacionadas | Heatmap de correlación |
| 2 numéricas específicas | Cómo se relacionan visualmente | Scatterplot |
| 2 numéricas con tendencia | Confirmar y mostrar correlación | Regplot |
| Muchas numéricas | Ver todas las relaciones de una | Pairplot |
| 1 variable categórica | Cuántos hay por categoría | Countplot |
| 1 categórica + 1 numérica | Métrica por categoría (suma, media) | Barplot |
| 1 numérica + 1 categórica | Distribución completa por grupo | Violinplot |
| 1 numérica + 1 categórica | Outliers + puntos reales por grupo | Boxplot + Stripplot |
| 1 numérica + 1 categórica | Comparar forma de distribución entre grupos | KDE por grupo |
| 2 categóricas + 1 numérica | Cruzar dos dimensiones | Heatmap de pivot |
| 2 categóricas | Ver composición proporcional | Barras apiladas % |
| 1 fecha + 1 numérica | Cómo evoluciona en el tiempo | Lineplot |

---

*Para variantes avanzadas de cada gráfico → `visualizacion_datos.md`*
