# 📊 Guía Completa de Visualización de Datos en Python

> **Librerías:** Matplotlib · Seaborn  
> **Variable de referencia:** `df_clean` (DataFrame ya limpiado)  
> **Nivel:** Básico → Senior

---

## Tabla de Contenidos

1. [Setup inicial](#1-setup-inicial)
2. [Anatomía de un gráfico](#2-anatomía-de-un-gráfico)
3. [¿Qué gráfico usar según el objetivo?](#3-qué-gráfico-usar-según-el-objetivo)
4. [Gráfico de barras (countplot y barplot)](#4-gráfico-de-barras-countplot-y-barplot)
5. [Histograma](#5-histograma)
6. [Boxplot](#6-boxplot)
7. [Gráfico de dispersión (scatterplot)](#7-gráfico-de-dispersión-scatterplot)
8. [Gráfico de líneas](#8-gráfico-de-líneas)
9. [Mapa de calor (heatmap)](#9-mapa-de-calor-heatmap)
10. [Gráfico de torta](#10-gráfico-de-torta)
11. [Pairplot](#11-pairplot)
12. [Formato profesional de ejes](#12-formato-profesional-de-ejes)
13. [Paletas de colores](#13-paletas-de-colores)
14. [Subplots — múltiples gráficos en un lienzo](#14-subplots--múltiples-gráficos-en-un-lienzo)
15. [Guardar gráficos](#15-guardar-gráficos)
16. [Patrones senior](#16-patrones-senior)
17. [Errores comunes](#17-errores-comunes)
18. [Cheat sheet](#18-cheat-sheet)

---

## 1. Setup inicial

Siempre la primera celda del notebook, antes de cualquier gráfico:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
import warnings

# ─── Supresión de warnings ──────────────────────────────────────────────────
warnings.filterwarnings('ignore')

# ─── Configuración global de gráficos ──────────────────────────────────────
sns.set_theme(style='whitegrid')        # estilo base (opciones: darkgrid, white, ticks)
sns.set_palette('husl')                 # paleta de colores por defecto
plt.rcParams['figure.figsize'] = (12, 6)  # tamaño por defecto de todos los gráficos
plt.rcParams['font.size'] = 12          # tamaño de fuente base
plt.rcParams['axes.titlesize'] = 16     # tamaño de títulos
plt.rcParams['axes.labelsize'] = 13     # tamaño de etiquetas de ejes
```

> 💡 `sns.set_theme()` reemplaza al `sns.set()` antiguo que genera FutureWarning en versiones recientes de Seaborn.

---

## 2. Anatomía de un gráfico

```
┌─────────────────────────────────────────┐
│              TÍTULO (title)             │
│  eje Y  │                               │
│ (ylabel)│         ÁREA DEL GRÁFICO      │
│         │                               │
│         └─────────────────────────────  │
│                    eje X (xlabel)       │
│              LEYENDA (legend)           │
└─────────────────────────────────────────┘
```

```python
# Estructura base de cualquier gráfico
plt.figure(figsize=(12, 6))           # 1. Tamaño del lienzo
sns.barplot(data=df_clean, ...)       # 2. Tipo de gráfico
plt.title('Título', fontsize=16)      # 3. Título
plt.xlabel('Etiqueta eje X')          # 4. Etiqueta X
plt.ylabel('Etiqueta eje Y')          # 5. Etiqueta Y
plt.xticks(rotation=45)               # 6. Rotación de etiquetas (si se superponen)
plt.tight_layout()                    # 7. Ajusta márgenes automáticamente
plt.show()                            # 8. Mostrar
```

> ⚠️ `plt.figure(figsize=...)` siempre **antes** del gráfico. `plt.show()` siempre **al final**.

---

## 3. ¿Qué gráfico usar según el objetivo?

| Pregunta de negocio | Gráfico | Función |
|---------------------|---------|---------|
| ¿Cuántos hay por categoría? | Barras | `countplot` |
| ¿Cuánto suma/promedia cada categoría? | Barras | `barplot` |
| ¿Cómo se distribuye una variable? | Histograma | `histplot` |
| ¿Hay outliers? ¿Cómo es la dispersión? | Caja | `boxplot` |
| ¿Hay relación entre dos variables? | Dispersión | `scatterplot` |
| ¿Cómo evoluciona algo en el tiempo? | Líneas | `lineplot` |
| ¿Qué tan correlacionadas están las variables? | Calor | `heatmap` |
| ¿Qué proporción tiene cada parte? | Torta | `pie` |
| ¿Relación entre todas las variables numéricas? | Pares | `pairplot` |

---

## 4. Gráfico de barras (countplot y barplot)

### 4.1 Countplot — contar ocurrencias por categoría

```python
# ─── Básico ─────────────────────────────────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.countplot(
    data=df_clean,
    x='Rubro',
    hue='Rubro',      # evita FutureWarning en Seaborn moderno
    legend=False
)
plt.title('Cantidad de ventas por Rubro')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

```python
# ─── Horizontal (mejor cuando las etiquetas son largas) ────────────────────
plt.figure(figsize=(12, 6))
p = sns.countplot(
    data=df_clean,
    y='Rubro',
    hue='Rubro',
    legend=False,
    palette='Wistia'
)
p.axes.set_title('Ventas por Rubro', fontsize=16)

# Etiquetas en las barras
for container in p.containers:
    p.bar_label(container, label_type='edge', padding=5)

plt.tight_layout()
plt.show()
```

### 4.2 Barplot — suma, promedio u otra métrica por categoría

```python
# ─── Suma de importes por rubro (horizontal) ────────────────────────────────
plt.figure(figsize=(12, 6))
sns.barplot(
    data=df_clean,
    x='Importe',
    y='Rubro',
    estimator=sum,        # función agregadora: sum, mean, median, max
    hue='Rubro',
    legend=False,
    palette='Blues_r'
)
plt.title('Recaudación total por Rubro')
plt.ticklabel_format(style='plain', axis='x')

# Formato moneda en eje X
fmt = ticker.StrMethodFormatter('${x:,.0f}')
plt.gca().xaxis.set_major_formatter(fmt)

plt.tight_layout()
plt.show()
```

```python
# ─── Ordenado de mayor a menor ──────────────────────────────────────────────
orden = df_clean.groupby('Rubro')['Importe'].sum().sort_values(ascending=False).index

plt.figure(figsize=(12, 6))
sns.barplot(
    data=df_clean,
    x='Rubro',
    y='Importe',
    estimator=sum,
    order=orden,          # orden explícito
    hue='Rubro',
    legend=False
)
plt.title('Recaudación por Rubro (ordenado)')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

```python
# ─── Barras agrupadas por dos categorías ────────────────────────────────────
plt.figure(figsize=(14, 6))
sns.barplot(
    data=df_clean,
    x='Rubro',
    y='Importe',
    hue='Sede',           # segunda dimensión
    estimator=sum
)
plt.title('Recaudación por Rubro y Sede')
plt.xticks(rotation=45)
plt.legend(title='Sede', bbox_to_anchor=(1.05, 1))
plt.tight_layout()
plt.show()
```

> 💡 **Cuándo usar cada uno:**  
> `countplot` → contar filas por categoría  
> `barplot` → calcular una métrica numérica (suma, promedio) por categoría

---

## 5. Histograma

Muestra cómo se distribuye una variable numérica.

```python
# ─── Básico ─────────────────────────────────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.histplot(data=df_clean, x='Importe')
plt.title('Distribución de Importes')
plt.tight_layout()
plt.show()
```

```python
# ─── Con formato de moneda y configuración avanzada ─────────────────────────
plt.figure(figsize=(12, 6))
sns.histplot(
    data=df_clean,
    x='Importe',
    bins=30,              # cantidad de barras (más bins = más detalle)
    kde=True,             # curva de densidad suavizada encima
    color='steelblue',
    alpha=0.7             # transparencia
)
plt.title('Distribución de Importes de Venta', fontsize=16)
plt.xlabel('Importe ($)')

fmt = ticker.StrMethodFormatter('${x:,.0f}')
plt.gca().xaxis.set_major_formatter(fmt)
plt.xticks(rotation=45)

plt.tight_layout()
plt.show()
```

```python
# ─── Comparar distribución entre grupos ─────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.histplot(
    data=df_clean,
    x='Importe',
    hue='Sede',           # una distribución por categoría
    kde=True,
    alpha=0.5
)
plt.title('Distribución de Importes por Sede')
plt.tight_layout()
plt.show()
```

> 💡 **Cómo interpretar:**  
> - Sesgado a la izquierda → mayoría de valores bajos, pocos muy altos  
> - Simétrico (campana) → distribución normal  
> - Bimodal (dos picos) → probablemente hay dos grupos distintos en los datos

---

## 6. Boxplot

Muestra la distribución estadística y detecta outliers.

```python
# ─── Básico ─────────────────────────────────────────────────────────────────
plt.figure(figsize=(12, 8))
sns.boxplot(data=df_clean, x='Satisfaccion_Cliente', y='Sede')
plt.title('Satisfacción del cliente por Sede')
plt.tight_layout()
plt.show()
```

```python
# ─── Con colores y configuración completa ───────────────────────────────────
plt.figure(figsize=(12, 8))
sns.boxplot(
    data=df_clean,
    x='Satisfaccion_Cliente',
    y='Sede',
    hue='Sede',
    legend=False,
    palette='Set2',
    flierprops={'marker': 'o', 'markerfacecolor': 'red', 'markersize': 8}  # outliers en rojo
)
plt.title('Satisfacción por Sede — Detección de outliers', fontsize=16)
plt.xlabel('Satisfacción del cliente')
plt.tight_layout()
plt.show()
```

```python
# ─── Boxplot + swarmplot (ver puntos individuales) ──────────────────────────
plt.figure(figsize=(12, 8))
sns.boxplot(data=df_clean, x='Satisfaccion_Cliente', y='Sede', palette='pastel')
sns.swarmplot(data=df_clean, x='Satisfaccion_Cliente', y='Sede', color='black', alpha=0.4, size=3)
plt.title('Distribución de Satisfacción con puntos reales')
plt.tight_layout()
plt.show()
```

> 💡 **Cómo leer un boxplot:**
> ```
> |──── bigote ────[  Q1 │ MEDIANA │ Q3  ]──── bigote ────|  ●outlier
>                   25%              75%
> ```
> - **Caja** = donde está el 50% central de los datos
> - **Línea del medio** = mediana
> - **Bigotes** = rango normal (Q1 - 1.5×IQR hasta Q3 + 1.5×IQR)
> - **Puntos sueltos** = outliers estadísticos

---

## 7. Gráfico de dispersión (scatterplot)

Muestra la relación entre dos variables numéricas.

```python
# ─── Básico ─────────────────────────────────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.scatterplot(data=df_clean, x='Importe', y='Comision_x_venta')
plt.title('Relación entre Importe y Comisión')
plt.tight_layout()
plt.show()
```

```python
# ─── Con tercera variable como color ────────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.scatterplot(
    data=df_clean,
    x='Importe',
    y='Comision_x_venta',
    hue='Sede',           # color por categoría
    alpha=0.7,
    s=80                  # tamaño de los puntos
)
plt.title('Importe vs Comisión por Sede')
plt.legend(title='Sede', bbox_to_anchor=(1.05, 1))
plt.tight_layout()
plt.show()
```

```python
# ─── Con línea de tendencia (regresión) ────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.regplot(
    data=df_clean,
    x='Importe',
    y='Comision_x_venta',
    scatter_kws={'alpha': 0.5},
    line_kws={'color': 'red'}
)
plt.title('Correlación: Importe vs Comisión')
plt.tight_layout()
plt.show()
```

> 💡 **Cómo interpretar correlación:**  
> - Puntos en diagonal ascendente → correlación positiva (una sube, la otra sube)  
> - Puntos en diagonal descendente → correlación negativa  
> - Nube dispersa sin forma → sin correlación

---

## 8. Gráfico de líneas

Para series temporales o evolución de una métrica.

```python
# ─── Básico ─────────────────────────────────────────────────────────────────
# Primero agrupar por período
df_clean['Fecha'] = pd.to_datetime(df_clean['Fecha'])
ventas_mes = df_clean.groupby(df_clean['Fecha'].dt.to_period('M'))['Importe'].sum().reset_index()
ventas_mes['Fecha'] = ventas_mes['Fecha'].astype(str)

plt.figure(figsize=(14, 6))
sns.lineplot(data=ventas_mes, x='Fecha', y='Importe')
plt.title('Evolución de ventas por mes')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

```python
# ─── Múltiples líneas por categoría ─────────────────────────────────────────
plt.figure(figsize=(14, 6))
sns.lineplot(
    data=df_clean,
    x='Fecha',
    y='Importe',
    hue='Sede',           # una línea por sede
    marker='o'            # puntos en cada observación
)
plt.title('Evolución de ventas por Sede')
plt.xticks(rotation=45)
plt.legend(title='Sede')
plt.tight_layout()
plt.show()
```

---

## 9. Mapa de calor (heatmap)

Ideal para visualizar correlaciones entre variables numéricas.

```python
# ─── Mapa de correlación ─────────────────────────────────────────────────────
plt.figure(figsize=(10, 8))
correlacion = df_clean.select_dtypes(include='number').corr()

sns.heatmap(
    correlacion,
    annot=True,           # muestra los valores dentro de cada celda
    fmt='.2f',            # formato de 2 decimales
    cmap='coolwarm',      # rojo=positivo, azul=negativo
    center=0,             # centra la escala en 0
    linewidths=0.5,
    square=True
)
plt.title('Mapa de Correlación entre Variables Numéricas', fontsize=16)
plt.tight_layout()
plt.show()
```

```python
# ─── Heatmap de tabla pivote ─────────────────────────────────────────────────
pivot = df_clean.pivot_table(
    values='Importe',
    index='Sede',
    columns='Rubro',
    aggfunc='sum'
)

plt.figure(figsize=(14, 8))
sns.heatmap(
    pivot,
    annot=True,
    fmt=',.0f',
    cmap='YlOrRd',
    linewidths=0.5
)
plt.title('Importe total por Sede y Rubro')
plt.tight_layout()
plt.show()
```

> 💡 **Cómo interpretar correlación:**  
> - `1.0` → correlación perfecta positiva  
> - `-1.0` → correlación perfecta negativa  
> - `0.0` → sin correlación  
> - Por encima de `0.7` o debajo de `-0.7` → correlación fuerte

---

## 10. Gráfico de torta

Para proporciones. Usar con pocas categorías (máximo 6).

```python
# ─── Básico ─────────────────────────────────────────────────────────────────
conteo = df_clean['Rubro'].value_counts()

plt.figure(figsize=(10, 8))
plt.pie(
    conteo.values,
    labels=conteo.index,
    autopct='%1.1f%%',    # muestra porcentaje con 1 decimal
    startangle=90
)
plt.title('Distribución de ventas por Rubro')
plt.tight_layout()
plt.show()
```

```python
# ─── Donut (más moderno y legible) ──────────────────────────────────────────
plt.figure(figsize=(10, 8))
wedges, texts, autotexts = plt.pie(
    conteo.values,
    labels=conteo.index,
    autopct='%1.1f%%',
    startangle=90,
    pctdistance=0.85,
    wedgeprops=dict(width=0.5)   # hace el hueco del donut
)
plt.title('Distribución de ventas por Rubro')
plt.tight_layout()
plt.show()
```

> ⚠️ **No usar torta cuando hay más de 6 categorías** — se vuelve ilegible. Usá barras horizontales en su lugar.

---

## 11. Pairplot

Muestra la relación entre todas las variables numéricas de una vez. Útil al inicio del EDA.

```python
# ─── Básico ─────────────────────────────────────────────────────────────────
numericas = df_clean.select_dtypes(include='number').columns.tolist()
sns.pairplot(df_clean[numericas])
plt.suptitle('Relaciones entre variables numéricas', y=1.02)
plt.show()
```

```python
# ─── Con color por categoría ─────────────────────────────────────────────────
sns.pairplot(
    df_clean[numericas + ['Sede']],
    hue='Sede',
    diag_kind='kde',      # curva de densidad en la diagonal
    plot_kws={'alpha': 0.6}
)
plt.suptitle('Pairplot por Sede', y=1.02)
plt.show()
```

> 💡 Si el dataset tiene muchas columnas, seleccioná solo las que te interesan:
> ```python
> sns.pairplot(df_clean[['Importe', 'Comision_x_venta', 'Satisfaccion_Cliente']])
> ```

---

## 12. Formato profesional de ejes

### 12.1 Formatos numéricos

```python
import matplotlib.ticker as ticker

# ─── Moneda ──────────────────────────────────────────────────────────────────
fmt_moneda = ticker.StrMethodFormatter('${x:,.0f}')
plt.gca().xaxis.set_major_formatter(fmt_moneda)
plt.gca().yaxis.set_major_formatter(fmt_moneda)

# ─── Miles con separador ─────────────────────────────────────────────────────
fmt_miles = ticker.StrMethodFormatter('{x:,.0f}')
plt.gca().xaxis.set_major_formatter(fmt_miles)

# ─── Porcentaje ──────────────────────────────────────────────────────────────
fmt_pct = ticker.PercentFormatter(xmax=100)
plt.gca().yaxis.set_major_formatter(fmt_pct)

# ─── Apagar notación científica (1e6 → 1000000) ────────────────────────────
plt.ticklabel_format(style='plain', axis='x')
plt.ticklabel_format(style='plain', axis='both')  # ambos ejes
```

### 12.2 Rotación y tamaño de etiquetas

```python
plt.xticks(rotation=45, fontsize=11)
plt.yticks(fontsize=11)

# Rotación con alineación correcta
plt.xticks(rotation=45, ha='right')  # ha='right' alinea al punto del tick
```

### 12.3 Límites de ejes

```python
plt.xlim(0, 500000)
plt.ylim(0, 100)
```

### 12.4 Líneas de referencia

```python
# Línea horizontal de referencia (ej: objetivo o promedio)
promedio = df_clean['Importe'].mean()
plt.axhline(y=promedio, color='red', linestyle='--', linewidth=1.5, label=f'Promedio: ${promedio:,.0f}')
plt.legend()

# Línea vertical
plt.axvline(x=300000, color='orange', linestyle=':', linewidth=1.5)
```

---

## 13. Paletas de colores

```python
# ─── Paletas de Seaborn ──────────────────────────────────────────────────────
# Cualitativas (para categorías sin orden)
'husl'         # variado y accesible - recomendada
'Set1'         # colores vivos
'Set2'         # colores suaves
'Paired'       # pares de colores
'tab10'        # colores de Tableau

# Secuenciales (para valores de menor a mayor)
'Blues'        # azules
'Greens'
'YlOrRd'       # amarillo → naranja → rojo
'viridis'      # azul → verde → amarillo (accesible para daltónicos)
'magma'

# Divergentes (para valores positivos y negativos)
'coolwarm'     # azul → blanco → rojo
'RdYlGn'       # rojo → amarillo → verde
'BrBG'

# Invertir cualquier paleta agregando _r
'Blues_r'      # azul invertido
'viridis_r'

# ─── Ver todas las paletas disponibles ───────────────────────────────────────
# sns.color_palette('husl', n_colors=10)  # ver colores individuales
```

```python
# ─── Colores personalizados ──────────────────────────────────────────────────
mi_paleta = ['#2196F3', '#4CAF50', '#FF5722', '#9C27B0']
sns.barplot(data=df_clean, x='Rubro', y='Importe', palette=mi_paleta)
```

---

## 14. Subplots — múltiples gráficos en un lienzo

```python
# ─── 2 gráficos lado a lado ──────────────────────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(16, 6))
fig.suptitle('Tablero Analítico', fontsize=18, fontweight='bold')

# Gráfico 1
sns.barplot(data=df_clean, x='Rubro', y='Importe', estimator=sum, ax=axes[0])
axes[0].set_title('Recaudación por Rubro')
axes[0].tick_params(axis='x', rotation=45)

# Gráfico 2
sns.histplot(data=df_clean, x='Importe', ax=axes[1])
axes[1].set_title('Distribución de Importes')

plt.tight_layout()
plt.show()
```

```python
# ─── Dashboard completo 2x2 ──────────────────────────────────────────────────
fig, axes = plt.subplots(2, 2, figsize=(16, 12))
fig.suptitle('Dashboard Analítico de Ventas', fontsize=18, fontweight='bold')

# [0,0] Barras
sns.barplot(data=df_clean, x='Rubro', y='Importe', estimator=sum,
            hue='Rubro', legend=False, ax=axes[0, 0])
axes[0, 0].set_title('Recaudación por Rubro')
axes[0, 0].tick_params(axis='x', rotation=45)

# [0,1] Histograma
sns.histplot(data=df_clean, x='Importe', kde=True, ax=axes[0, 1])
axes[0, 1].set_title('Distribución de Importes')

# [1,0] Boxplot
sns.boxplot(data=df_clean, x='Satisfaccion_Cliente', y='Sede',
            hue='Sede', legend=False, ax=axes[1, 0])
axes[1, 0].set_title('Satisfacción por Sede')

# [1,1] Dispersión
sns.scatterplot(data=df_clean, x='Importe', y='Comision_x_venta',
                hue='Sede', alpha=0.6, ax=axes[1, 1])
axes[1, 1].set_title('Importe vs Comisión')

plt.tight_layout()
plt.show()
```

---

## 15. Guardar gráficos

```python
# ─── Guardar el último gráfico generado ──────────────────────────────────────
plt.savefig('grafico.png', dpi=150, bbox_inches='tight')

# ─── Alta resolución para presentaciones ─────────────────────────────────────
plt.savefig('grafico_hd.png', dpi=300, bbox_inches='tight')

# ─── Guardar antes de plt.show() ─────────────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.barplot(data=df_clean, x='Rubro', y='Importe', estimator=sum)
plt.title('Recaudación por Rubro')
plt.tight_layout()
plt.savefig('barras_rubro.png', dpi=150, bbox_inches='tight')  # primero guardar
plt.show()                                                       # después mostrar

# ─── Descargar en Colab ───────────────────────────────────────────────────────
from google.colab import files
files.download('grafico.png')
```

> ⚠️ `plt.savefig()` siempre **antes** de `plt.show()`. Después del show la figura se borra de memoria.

---

## 16. Patrones senior

### 16.1 Etiquetas de datos en las barras

```python
plt.figure(figsize=(12, 6))
p = sns.barplot(data=df_clean, x='Rubro', y='Importe', estimator=sum,
                hue='Rubro', legend=False)

# Etiquetas automáticas
for container in p.containers:
    p.bar_label(container, label_type='edge', padding=5,
                fmt='${:,.0f}')  # formato moneda en las etiquetas

plt.title('Recaudación por Rubro')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### 16.2 Anotaciones personalizadas

```python
plt.figure(figsize=(12, 6))
ax = sns.histplot(data=df_clean, x='Importe')

# Anotación en un punto específico
ax.annotate(
    'Valor atípico',
    xy=(0, 5),              # punto donde apunta la flecha
    xytext=(50000, 20),     # posición del texto
    arrowprops=dict(arrowstyle='->', color='red'),
    fontsize=11,
    color='red'
)
plt.tight_layout()
plt.show()
```

### 16.3 Gráfico con dos ejes Y

```python
fig, ax1 = plt.subplots(figsize=(14, 6))

# Eje izquierdo — barras
color1 = 'steelblue'
ventas = df_clean.groupby('Mes')['Importe'].sum()
ax1.bar(ventas.index, ventas.values, color=color1, alpha=0.7, label='Ventas')
ax1.set_ylabel('Importe total ($)', color=color1)
ax1.tick_params(axis='y', labelcolor=color1)

# Eje derecho — línea
ax2 = ax1.twinx()
color2 = 'coral'
satisfaccion = df_clean.groupby('Mes')['Satisfaccion_Cliente'].mean()
ax2.plot(satisfaccion.index, satisfaccion.values, color=color2, marker='o', linewidth=2, label='Satisfacción')
ax2.set_ylabel('Satisfacción promedio', color=color2)
ax2.tick_params(axis='y', labelcolor=color2)

plt.title('Ventas y Satisfacción por Mes')
fig.tight_layout()
plt.show()
```

### 16.4 Top N categorías

```python
# Solo las 5 categorías con más ventas
top5 = df_clean.groupby('Vendedor')['Importe'].sum().nlargest(5).reset_index()

plt.figure(figsize=(12, 5))
sns.barplot(data=top5, x='Importe', y='Vendedor',
            hue='Vendedor', legend=False, palette='Blues_r')
plt.title('Top 5 Vendedores por Importe')
fmt = ticker.StrMethodFormatter('${x:,.0f}')
plt.gca().xaxis.set_major_formatter(fmt)
plt.tight_layout()
plt.show()
```

### 16.5 Comparar antes/después o dos períodos

```python
# Filtrar dos grupos
grupo_a = df_clean[df_clean['Año'] == 2022]
grupo_b = df_clean[df_clean['Año'] == 2023]

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 6), sharey=True)
fig.suptitle('Comparativa 2022 vs 2023', fontsize=16)

sns.barplot(data=grupo_a, x='Rubro', y='Importe', estimator=sum, ax=ax1)
ax1.set_title('2022')
ax1.tick_params(axis='x', rotation=45)

sns.barplot(data=grupo_b, x='Rubro', y='Importe', estimator=sum, ax=ax2)
ax2.set_title('2023')
ax2.tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.show()
```

---

## 17. Errores comunes

### ❌ plt.figure() después del gráfico

```python
# Incorrecto
sns.barplot(...)
plt.figure(figsize=(12, 6))   # ya es tarde

# Correcto
plt.figure(figsize=(12, 6))   # siempre antes
sns.barplot(...)
```

### ❌ plt.show() antes de guardar

```python
# Incorrecto — guarda una imagen en blanco
plt.show()
plt.savefig('grafico.png')

# Correcto
plt.savefig('grafico.png')
plt.show()
```

### ❌ FutureWarning en Seaborn moderno

```python
# Incorrecto — genera FutureWarning
sns.countplot(data=df_clean, x='Rubro', palette='Set2')

# Correcto — Seaborn moderno requiere hue
sns.countplot(data=df_clean, x='Rubro', hue='Rubro', legend=False, palette='Set2')
```

### ❌ Notación científica en ejes numéricos

```python
# El eje muestra 1e6 en lugar de 1,000,000
# Solución:
plt.ticklabel_format(style='plain', axis='x')
# O con formato explícito:
plt.gca().xaxis.set_major_formatter(ticker.StrMethodFormatter('{x:,.0f}'))
```

### ❌ Etiquetas superpuestas en eje X

```python
# Solución: rotar + alinear
plt.xticks(rotation=45, ha='right')

# O usar gráfico horizontal
sns.barplot(data=df_clean, x='Importe', y='Rubro')  # invertir ejes
```

---

## 18. Cheat sheet

```python
# ─── Setup ───────────────────────────────────────────────────────────────────
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
sns.set_theme(style='whitegrid')

# ─── Estructura base ─────────────────────────────────────────────────────────
plt.figure(figsize=(12, 6))
sns.<tipo>(data=df_clean, x='col_x', y='col_y')
plt.title('Título')
plt.tight_layout()
plt.savefig('grafico.png', dpi=150, bbox_inches='tight')  # opcional
plt.show()

# ─── Tipos de gráficos ───────────────────────────────────────────────────────
sns.countplot(data=df_clean, x='cat', hue='cat', legend=False)
sns.barplot(data=df_clean, x='cat', y='num', estimator=sum)
sns.histplot(data=df_clean, x='num', kde=True)
sns.boxplot(data=df_clean, x='num', y='cat')
sns.scatterplot(data=df_clean, x='num1', y='num2', hue='cat')
sns.lineplot(data=df_clean, x='fecha', y='num')
sns.heatmap(df.corr(), annot=True, fmt='.2f', cmap='coolwarm')
sns.pairplot(df_clean[cols_numericas])

# ─── Formatos de ejes ────────────────────────────────────────────────────────
plt.ticklabel_format(style='plain', axis='x')
plt.gca().xaxis.set_major_formatter(ticker.StrMethodFormatter('${x:,.0f}'))
plt.xticks(rotation=45, ha='right')

# ─── Subplots ────────────────────────────────────────────────────────────────
fig, axes = plt.subplots(2, 2, figsize=(16, 12))
sns.barplot(..., ax=axes[0, 0])
sns.histplot(..., ax=axes[0, 1])
```

---

*Guía elaborada para uso personal y académico · Seaborn: [seaborn.pydata.org](https://seaborn.pydata.org) · Matplotlib: [matplotlib.org](https://matplotlib.org)*
