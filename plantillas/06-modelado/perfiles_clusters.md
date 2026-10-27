# Perfiles de Clusters

Plantilla reutilizable para describir e interpretar los clusters encontrados
por cualquier algoritmo de clustering (K-Means, DBSCAN, etc.).

**Posición en el flujo:** `06-modelado/`

---

## ¿Para qué sirve perfilar clusters?

El clustering te devuelve números (0, 1, 2, 3...). Esos números no
significan nada por sí solos. El perfil de clusters es el paso que
convierte esos números en segmentos interpretables con nombre y
descripción de negocio.

Sin perfilar: *"el cluster 2 tiene 430 clientes"*
Con perfilar: *"el cluster 2 son clientes jóvenes de alto ingreso
con pocas compras pero montos altos — posibles compradores premium
ocasionales"*

---

## BLOQUE 0 — Imports y configuración

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# ── Configuración ──────────────────────────────────────────
COL_CLUSTER     = "cluster_dbscan"   # ← CAMBIAR: nombre de la columna de clusters
EXCLUIR_RUIDO   = True               # ← CAMBIAR: True excluye el -1 de DBSCAN
RANDOM_STATE    = 42
# ───────────────────────────────────────────────────────────

# Si usás DBSCAN y querés excluir el ruido del perfilado
if EXCLUIR_RUIDO:
    df_perfil = df_clean[df_clean[COL_CLUSTER] != -1].copy()
else:
    df_perfil = df_clean.copy()

print(f"Registros para perfilar: {len(df_perfil)}")
print(f"Clusters: {sorted(df_perfil[COL_CLUSTER].unique())}")
```

---

## BLOQUE 1 — Tamaño de cada cluster

El primer paso es entender cuántos registros tiene cada cluster.
Un cluster con 3 registros probablemente no es interpretable.

```python
# ── Configuración ──────────────────────────────────────────
COLUMNAS_PERFIL = [
    "edad_recalculada",
    "ingreso_mensual",
    "cantidad_compras",
    "monto_total",
    "dias_desde_ultima_compra",
    "antiguedad_dias"
]  # ← CAMBIAR: las columnas numéricas que querés analizar
# ───────────────────────────────────────────────────────────

tamaños = df_perfil[COL_CLUSTER].value_counts().sort_index()

print("Tamaño de cada cluster:")
print(tamaños)
print(f"\nTotal: {tamaños.sum()} registros")
print(f"\nPorcentaje por cluster:")
print((tamaños / tamaños.sum() * 100).round(1).astype(str) + "%")
```

---

## BLOQUE 2 — Perfil promedio por cluster

El `groupby` + `mean()` calcula el valor promedio de cada variable
numérica para cada cluster. Es la radiografía de cada segmento.

```python
perfil = (
    df_perfil
    .groupby(COL_CLUSTER)[COLUMNAS_PERFIL]
    .mean()
    .round(2)
)

display(perfil)
```

### Cómo leer la tabla de perfil

Cada fila es un cluster. Cada columna es la media de esa variable
para los registros de ese cluster. Buscás qué hace **distinto** a
cada cluster del resto: qué columna tiene valores notablemente más
altos o más bajos que los otros clusters.

---

## BLOQUE 3 — Perfil completo con múltiples estadísticos

La media sola puede engañar si hay mucha dispersión dentro del
cluster. Agregar mediana, desvío estándar y conteo da una imagen
más completa.

```python
perfil_completo = (
    df_perfil
    .groupby(COL_CLUSTER)[COLUMNAS_PERFIL]
    .agg(["mean", "median", "std", "count"])
    .round(2)
)

display(perfil_completo)
```

---

## BLOQUE 4 — Perfil normalizado (comparación relativa)

Cuando las variables tienen escalas muy distintas (edad en decenas,
ingresos en miles), es difícil comparar clusters visualmente.
Normalizar divide cada columna por su máximo para llevarla al
rango [0, 1].

```python
# Normalizar el perfil promedio para comparación visual
perfil_norm = perfil.copy()

for col in perfil_norm.columns:
    max_val = perfil_norm[col].max()
    if max_val != 0:
        perfil_norm[col] = perfil_norm[col] / max_val

display(perfil_norm.round(3))
```

---

## BLOQUE 5 — Heatmap del perfil normalizado

El heatmap es la forma más rápida de ver qué distingue a cada
cluster. Colores más intensos = valores más altos relativos.

```python
plt.figure(figsize=(12, max(4, len(perfil_norm) * 0.8)))

sns.heatmap(
    perfil_norm.T,           # transpuesto: clusters en columnas, variables en filas
    annot=True,
    fmt=".2f",
    cmap="YlOrRd",
    linewidths=0.5,
    cbar_kws={"label": "Valor normalizado (0-1)"}
)

plt.title(f"Perfil de clusters — {COL_CLUSTER}\n(valores normalizados por variable)")
plt.xlabel("Cluster")
plt.ylabel("Variable")
plt.tight_layout()
plt.show()
```

---

## BLOQUE 6 — Distribución de variables categóricas por cluster

Si tenés variables categóricas (género, provincia, segmento, etc.),
este bloque muestra cómo se distribuyen dentro de cada cluster.

```python
# ── Configuración ──────────────────────────────────────────
COLUMNAS_CATEGORICAS = ["genero_limpio", "provincia", "segmento"]  # ← CAMBIAR
# ───────────────────────────────────────────────────────────

for col in COLUMNAS_CATEGORICAS:
    print(f"\n{'='*50}")
    print(f"Distribución de '{col}' por cluster:")
    tabla = pd.crosstab(
        df_perfil[COL_CLUSTER],
        df_perfil[col],
        normalize="index"   # porcentaje dentro de cada cluster
    ).round(3) * 100

    display(tabla.style.format("{:.1f}%"))
```

---

## BLOQUE 7 — Boxplots por cluster

Los boxplots muestran la distribución completa dentro de cada
cluster, no solo la media. Útil para detectar si un cluster tiene
mucha dispersión interna.

```python
# ── Configuración ──────────────────────────────────────────
COLS_BOXPLOT = COLUMNAS_PERFIL[:4]  # ← CAMBIAR: elegir las más importantes
# ───────────────────────────────────────────────────────────

n_cols = 2
n_rows = (len(COLS_BOXPLOT) + 1) // n_cols

fig, axes = plt.subplots(n_rows, n_cols, figsize=(14, n_rows * 4))
axes = axes.flatten()

for i, col in enumerate(COLS_BOXPLOT):
    df_perfil.boxplot(
        column=col,
        by=COL_CLUSTER,
        ax=axes[i],
        grid=False
    )
    axes[i].set_title(col)
    axes[i].set_xlabel("Cluster")

# Ocultar ejes sobrantes si el número de columnas es impar
for j in range(i + 1, len(axes)):
    axes[j].set_visible(False)

plt.suptitle("Distribución de variables por cluster", y=1.02)
plt.tight_layout()
plt.show()
```

---

## BLOQUE 8 — Asignar nombres de negocio a los clusters

Una vez que entendés cada cluster, asignale un nombre descriptivo.
Ese nombre debe reflejar qué hace especial a ese segmento.

```python
# ── Configuración ──────────────────────────────────────────
# Definir después de analizar el perfil
NOMBRES_CLUSTERS = {
    0: "Clientes premium",       # ← CAMBIAR según tu análisis
    1: "Clientes ocasionales",   # ← CAMBIAR
    2: "Clientes inactivos",     # ← CAMBIAR
    3: "Clientes nuevos",        # ← CAMBIAR
}
# ───────────────────────────────────────────────────────────

df_perfil["segmento"] = df_perfil[COL_CLUSTER].map(NOMBRES_CLUSTERS)

print("Distribución por segmento:")
print(df_perfil["segmento"].value_counts())
```

---

## BLOQUE 9 — Comparación entre dos modelos de clustering

Cuando probaste K-Means y DBSCAN sobre el mismo dataset, comparar
los segmentos que encontró cada uno ayuda a validar la consistencia.

```python
# ── Configuración ──────────────────────────────────────────
COL_CLUSTER_A = "cluster_kmeans"  # ← CAMBIAR
COL_CLUSTER_B = "cluster_dbscan"  # ← CAMBIAR
# ───────────────────────────────────────────────────────────

print("Tabla de contingencia:")
display(pd.crosstab(
    df_perfil[COL_CLUSTER_A],
    df_perfil[COL_CLUSTER_B],
    rownames=[COL_CLUSTER_A],
    colnames=[COL_CLUSTER_B]
))
```

### Cómo leer la tabla de contingencia

Cada celda muestra cuántos registros ambos modelos coincidieron en
clasificar juntos. Si una fila tiene un valor dominante en una sola
columna, significa que ese cluster de A corresponde bien a ese
cluster de B. Si los valores están repartidos entre varias columnas,
los modelos no coincidieron en ese segmento.

---

## Referencia rápida — señales al perfilar

| Señal | Qué significa |
|---|---|
| Un cluster tiene todas las variables en valores medios | Cluster "basurero" — registros que no encajan en ningún lado |
| Dos clusters tienen perfiles casi idénticos | Quizás k es muy alto — probar con menos clusters |
| Un cluster tiene muy pocos registros (<3% del total) | Puede ser ruido o un segmento real muy pequeño — investigar |
| Una variable tiene el mismo valor en todos los clusters | Esa variable no distingue entre segmentos — reconsiderar incluirla |
| El heatmap tiene filas completamente uniformes | Esa variable no aporta información de separación |

---

*Template de Perfiles de Clusters — ds-toolkit by andressonsino*
*Posición en el flujo: `06-modelado/`*
