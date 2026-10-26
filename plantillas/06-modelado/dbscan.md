# DBSCAN — Template de Clustering por Densidad

Plantilla reutilizable para aplicar DBSCAN de básico a avanzado.
Incluye selección de parámetros con K-Distance Plot, grid search con
Silhouette Score y filtro por porcentaje de ruido.

**Posición en el flujo:** `06-modelado/`

---

## ¿Qué es DBSCAN?

DBSCAN (Density-Based Spatial Clustering of Applications with Noise)
es un algoritmo de clustering que agrupa puntos según su densidad en
el espacio de features. A diferencia de K-Means, **no requiere que
definas la cantidad de clusters de antemano** y puede encontrar
clusters de cualquier forma geométrica.

### Conceptos fundamentales

**Epsilon (eps):** radio de vecindad alrededor de cada punto. Define
qué tan lejos puede estar un vecino para ser considerado "cercano".

**min_samples:** cantidad mínima de puntos que deben existir dentro
del radio eps para que un punto sea considerado "núcleo".

### Los tres tipos de puntos en DBSCAN

| Tipo | Descripción | Etiqueta |
|---|---|---|
| **Punto núcleo** | Tiene al menos `min_samples` vecinos dentro de `eps` | Pertenece a un cluster |
| **Punto borde** | Está dentro del radio de un núcleo pero no tiene suficientes vecinos propios | Pertenece al cluster del núcleo más cercano |
| **Ruido** | No pertenece a ningún núcleo ni está en radio de ninguno | Etiqueta `-1` |

### Diferencias clave con K-Means

| | K-Means | DBSCAN |
|---|---|---|
| Definir k | Sí, obligatorio | No |
| Forma de clusters | Solo convexos (esféricos) | Cualquier forma |
| Outliers | Se asignan al cluster más cercano | Se identifican como ruido (-1) |
| Escalado requerido | Sí | **Sí — crítico** |
| Parámetros | `n_clusters` | `eps` y `min_samples` |

---

## BLOQUE 0 — Imports y configuración

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import NearestNeighbors
from sklearn.metrics import silhouette_score
from sklearn.decomposition import PCA

# ── Configuración ──────────────────────────────────────────
RANDOM_STATE    = 42
MAX_RUIDO_PCT   = 10      # ← CAMBIAR: porcentaje máximo de ruido aceptable
TOP_N_COMBOS    = 15      # ← CAMBIAR: cuántas mejores combinaciones mostrar
# ───────────────────────────────────────────────────────────
```

---

## BLOQUE 1 — Preparación de datos (siempre antes de DBSCAN)

DBSCAN usa distancias entre puntos. Si las variables tienen escalas
distintas (una en miles, otra en unidades), las de mayor escala
dominan el cálculo de distancia y el modelo falla.
**El escalado es obligatorio.**

```python
# ── Configuración ──────────────────────────────────────────
COLUMNAS_DBSCAN = ['col_1', 'col_2', 'col_3']  # ← CAMBIAR
# ───────────────────────────────────────────────────────────

df_dbscan = df_clean[COLUMNAS_DBSCAN].copy()

scaler = StandardScaler()
df_scaled = scaler.fit_transform(df_dbscan)

print(f"Shape: {df_scaled.shape}")
print(f"Media por columna (debe ser ~0): {df_scaled.mean(axis=0).round(2)}")
print(f"Std por columna (debe ser ~1):   {df_scaled.std(axis=0).round(2)}")
```

---

## BLOQUE 2 — K-Distance Plot para elegir eps

### ¿Por qué se usa?

Elegir `eps` a mano es difícil. El K-Distance Plot te muestra la
distribución de distancias en los datos y te ayuda a encontrar el
"codo" natural: el punto donde las distancias empiezan a crecer
rápido. Ese punto es un buen valor de `eps`.

### ¿Cómo funciona?

1. Para cada punto, calcula la distancia a su k-ésimo vecino más
   cercano (donde k = `min_samples`)
2. Ordena todas esas distancias de menor a mayor
3. Grafica la curva — el "codo" indica dónde los puntos dejan de
   tener vecinos cercanos (la transición entre puntos densos y ruido)

### Cómo leer el gráfico

- La parte **plana** al inicio = zona densa (clusters)
- El **codo** = umbral natural entre densidad y dispersión → ese es tu `eps`
- La parte **empinada** al final = zona de ruido

```python
# ── Configuración ──────────────────────────────────────────
MIN_SAMPLES_REF = 5   # ← CAMBIAR: usar el min_samples que pensás usar
# ───────────────────────────────────────────────────────────

vecinos = NearestNeighbors(n_neighbors=MIN_SAMPLES_REF)
vecinos.fit(df_scaled)

distancias, indices = vecinos.kneighbors(df_scaled)

# Distancia al k-ésimo vecino (último de la fila)
distancias_ordenadas = np.sort(distancias[:, MIN_SAMPLES_REF - 1])

plt.figure(figsize=(10, 5))
plt.plot(distancias_ordenadas)
plt.xlabel("Puntos ordenados por distancia")
plt.ylabel(f"Distancia al vecino {MIN_SAMPLES_REF}")
plt.title(f"K-Distance Plot — min_samples={MIN_SAMPLES_REF}")
plt.grid(alpha=0.3)
plt.axhline(
    y=distancias_ordenadas[int(len(distancias_ordenadas) * 0.9)],
    color='red', linestyle='--', alpha=0.5,
    label='Referencia percentil 90'
)
plt.legend()
plt.show()

print(f"Percentil 80: {np.percentile(distancias_ordenadas, 80):.2f}")
print(f"Percentil 90: {np.percentile(distancias_ordenadas, 90):.2f}")
print(f"Percentil 95: {np.percentile(distancias_ordenadas, 95):.2f}")
```

**Cómo usar el resultado:** buscá visualmente el codo en la curva.
Los percentiles impresos te dan referencias numéricas.
El valor en el codo es tu punto de partida para `eps`.

---

## BLOQUE 3 — DBSCAN básico (exploración inicial)

Antes del grid search, probá una combinación inicial basada en lo
que viste en el K-Distance Plot. Sirve para entender qué devuelve
el modelo y calibrar expectativas.

```python
# ── Configuración ──────────────────────────────────────────
EPS_INICIAL         = 1.5  # ← CAMBIAR: valor del codo del K-Distance Plot
MIN_SAMPLES_INICIAL = 5    # ← CAMBIAR
# ───────────────────────────────────────────────────────────

dbscan = DBSCAN(eps=EPS_INICIAL, min_samples=MIN_SAMPLES_INICIAL)
etiquetas = dbscan.fit_predict(df_scaled)

n_clusters = len(set(etiquetas) - {-1})
n_ruido    = np.sum(etiquetas == -1)
pct_ruido  = round(n_ruido / len(etiquetas) * 100, 2)

print(f"Clusters encontrados: {n_clusters}")
print(f"Puntos de ruido:      {n_ruido} ({pct_ruido}%)")
print(f"\nDistribución de clusters:")
print(pd.Series(etiquetas).value_counts().sort_index())
```

---

## BLOQUE 4 — Grid search manual con Silhouette Score

### ¿Por qué no existe un GridSearchCV para DBSCAN?

DBSCAN no tiene una función de loss que sklearn pueda optimizar
automáticamente. Hay que recorrer las combinaciones a mano y
calcular Silhouette Score para cada una.

### Cómo funciona el filtro de ruido

El Silhouette Score no incluye los puntos de ruido (-1) porque no
pertenecen a ningún cluster. Si hubiera demasiado ruido, el score
sería artificialmente bueno pero el modelo estaría ignorando gran
parte de los datos.

**Regla:** filtrar combinaciones donde el ruido supere
`MAX_RUIDO_PCT` (definido en la configuración). Así garantizás que
el modelo clasifique la mayoría de los puntos.

### Condiciones para calcular Silhouette

El Silhouette Score solo se puede calcular cuando:
- Hay **al menos 2 clusters** (con 1 solo cluster no hay comparación)
- Los puntos clasificados son **más que los clusters** (n > k)

El código verifica ambas condiciones antes de calcular.

```python
resultados = []

# ── Configuración ──────────────────────────────────────────
EPS_VALORES         = np.arange(1.0, 6.1, 0.2)  # ← CAMBIAR: rango a explorar
MIN_SAMPLES_VALORES = [3, 4, 5, 7, 10, 15, 20]  # ← CAMBIAR
# ───────────────────────────────────────────────────────────

for min_s in MIN_SAMPLES_VALORES:
    for eps in EPS_VALORES:

        dbscan   = DBSCAN(eps=eps, min_samples=min_s)
        etiquetas = dbscan.fit_predict(df_scaled)

        # Máscara para excluir ruido
        mascara   = etiquetas != -1
        n_clusters = len(set(etiquetas) - {-1})
        n_clientes = mascara.sum()

        # Silhouette solo si hay al menos 2 clusters y
        # más puntos que clusters
        if 2 <= n_clusters < n_clientes:

            silhouette = silhouette_score(
                df_scaled[mascara],
                etiquetas[mascara]
            )

            ruido = (~mascara).sum()

            resultados.append({
                "eps":              round(eps, 2),
                "min_samples":      min_s,
                "clusters":         n_clusters,
                "silhouette":       round(silhouette, 4),
                "ruido":            ruido,
                "porcentaje_ruido": round(ruido / len(etiquetas) * 100, 2)
            })

df_resultados = pd.DataFrame(resultados)
print(f"Combinaciones evaluadas: {len(df_resultados)}")
print(f"Combinaciones con ruido ≤ {MAX_RUIDO_PCT}%: "
      f"{(df_resultados['porcentaje_ruido'] <= MAX_RUIDO_PCT).sum()}")
```

---

## BLOQUE 5 — Selección de la mejor combinación

Filtrá por porcentaje de ruido y ordená por Silhouette Score.
Las primeras filas son las mejores combinaciones candidatas.

```python
mejores = (
    df_resultados[df_resultados["porcentaje_ruido"] <= MAX_RUIDO_PCT]
    .sort_values("silhouette", ascending=False)
    .head(TOP_N_COMBOS)
)

display(mejores)
```

### Cómo leer la tabla de resultados

| Columna | Qué significa | Qué buscar |
|---|---|---|
| `eps` | Radio de vecindad | El valor del codo del K-Distance Plot |
| `min_samples` | Densidad mínima | Mayor = clusters más densos, más ruido |
| `clusters` | Clusters encontrados | Depende del negocio. 3–6 suelen ser interpretables |
| `silhouette` | Separación entre clusters | Más alto = mejor separación (máximo 1) |
| `ruido` | Puntos no clasificados | Cuanto menos, mejor (salvo que sean outliers reales) |
| `porcentaje_ruido` | % de puntos sin cluster | Filtrado a ≤ 10% |

**No elegir siempre el mayor Silhouette.** Evaluar también:
- ¿Cuántos clusters devuelve? ¿Son interpretables para el negocio?
- ¿Cómo están distribuidos los tamaños de clusters?
- Un Silhouette de 0.45 con 3 clusters interpretables puede ser mejor
  que 0.60 con 8 clusters imposibles de nombrar.

---

## BLOQUE 6 — Distribución de clusters en las mejores combinaciones

Antes de elegir, revisá cómo está distribuido el tamaño de clusters
en cada combinación candidata. Un cluster con 2 puntos y otro con
490 no es útil aunque tenga buen Silhouette.

```python
distribuciones = []
cluster_min_list = []
cluster_max_list = []

for _, fila in mejores.iterrows():

    dbscan = DBSCAN(
        eps=float(fila["eps"]),
        min_samples=int(fila["min_samples"])
    )

    etiquetas = dbscan.fit_predict(df_scaled)

    # Tamaño de cada cluster (sin ruido)
    tamaños = (
        pd.Series(etiquetas[etiquetas != -1])
        .value_counts()
        .sort_index()
    )

    distribuciones.append(tamaños.to_dict())
    cluster_min_list.append(tamaños.min())
    cluster_max_list.append(tamaños.max())

mejores = mejores.copy()
mejores["distribucion"]  = distribuciones
mejores["cluster_min"]   = cluster_min_list
mejores["cluster_max"]   = cluster_max_list

display(mejores[["eps", "min_samples", "clusters", "silhouette",
                  "porcentaje_ruido", "cluster_min", "cluster_max",
                  "distribucion"]])
```

---

## BLOQUE 7 — Entrenar el modelo final y visualizar

Una vez elegidos los hiperparámetros, entrenar el modelo final
y visualizar con PCA reducido a 2 dimensiones.

```python
# ── Configuración ──────────────────────────────────────────
EPS_FINAL         = 4.0  # ← CAMBIAR: el elegido en el grid search
MIN_SAMPLES_FINAL = 3    # ← CAMBIAR
# ───────────────────────────────────────────────────────────

dbscan_final = DBSCAN(eps=EPS_FINAL, min_samples=MIN_SAMPLES_FINAL)
clusters_final = dbscan_final.fit_predict(df_scaled)

# Guardar en el DataFrame original
df_clean["cluster_dbscan"] = clusters_final

# Resumen
n_clusters = len(set(clusters_final) - {-1})
n_ruido    = np.sum(clusters_final == -1)
print(f"Clusters encontrados: {n_clusters}")
print(f"Puntos de ruido:      {n_ruido} ({n_ruido/len(clusters_final)*100:.1f}%)")
print(f"\nDistribución:")
print(pd.Series(clusters_final).value_counts().sort_index())
```

### Visualización con PCA

```python
pca = PCA(n_components=2, random_state=RANDOM_STATE)
X_pca = pca.fit_transform(df_scaled)

df_pca = pd.DataFrame(X_pca, columns=["PC1", "PC2"])
df_pca["cluster"] = clusters_final

plt.figure(figsize=(10, 6))

for cluster in sorted(df_pca["cluster"].unique()):

    datos = df_pca[df_pca["cluster"] == cluster]

    plt.scatter(
        datos["PC1"],
        datos["PC2"],
        label="Ruido" if cluster == -1 else f"Cluster {cluster}",
        alpha=0.6,
        color="black" if cluster == -1 else None,
        s=20 if cluster == -1 else 40
    )

varianza = pca.explained_variance_ratio_.sum() * 100

plt.xlabel(f"PC1")
plt.ylabel(f"PC2")
plt.title(f"DBSCAN — eps={EPS_FINAL}, min_samples={MIN_SAMPLES_FINAL}\n"
          f"Varianza explicada por PCA: {varianza:.1f}%")
plt.legend()
plt.grid(alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## BLOQUE 8 — Perfil de clusters (interpretación)

El resultado del clustering solo tiene valor si podés explicar
qué significa cada cluster en términos de negocio.

```python
# ── Configuración ──────────────────────────────────────────
COLUMNAS_PERFIL = ['col_1', 'col_2', 'col_3']  # ← CAMBIAR: columnas a analizar
# ───────────────────────────────────────────────────────────

# Agregar etiquetas al DataFrame completo (sin escalado)
df_perfil = df_clean.copy()
df_perfil["cluster"] = clusters_final

# Perfil por cluster (sin incluir el ruido)
perfil = (
    df_perfil[df_perfil["cluster"] != -1]
    .groupby("cluster")[COLUMNAS_PERFIL]
    .mean()
    .round(2)
)

display(perfil)

# Tamaño de cada cluster
print("\nTamaño de clusters:")
print(df_perfil["cluster"].value_counts().sort_index())
```

---

## BLOQUE 9 — Comparación DBSCAN vs K-Means (avanzado)

Cuando tenés ambos modelos entrenados, comparar qué tan parecidos
son los segmentos que encontraron.

```python
# Requiere haber entrenado ambos modelos previamente
df_comparacion = pd.DataFrame({
    "cluster_kmeans": clusters_kmeans,   # ← CAMBIAR: tu variable de K-Means
    "cluster_dbscan": clusters_final
})

print("Tabla de contingencia KMeans vs DBSCAN:")
display(pd.crosstab(
    df_comparacion["cluster_kmeans"],
    df_comparacion["cluster_dbscan"]
))
```

---

## Referencia rápida — ¿qué hacer cuando DBSCAN da resultados malos?

| Problema | Síntoma | Solución |
|---|---|---|
| Un solo cluster gigante | Todos en cluster 0, sin ruido | `eps` muy alto — bajarlo |
| Todo es ruido (-1) | Casi nada clasificado | `eps` muy bajo o `min_samples` muy alto — ajustar |
| Demasiados clusters pequeños | 20+ clusters de 2-3 puntos | `min_samples` muy bajo — subirlo |
| Silhouette Score muy bajo | <0.2 | Los datos no tienen estructura de densidad clara — considerar K-Means |
| Buenos scores pero clusters no interpretables | Silhouette alto, clusters sin sentido | Revisar qué features se incluyeron |

---

## Cuándo usar DBSCAN vs K-Means

| Usar DBSCAN | Usar K-Means |
|---|---|
| Los clusters no son de forma esférica | Los clusters son compactos y separados |
| Querés detectar outliers automáticamente | No necesitás detección de outliers |
| No sabés cuántos clusters hay | Tenés una idea del k esperado |
| Los datos tienen zonas de densidad variable | Los datos tienen densidad relativamente uniforme |
| Tenés datasets con formas complejas | El dataset es de alta dimensionalidad (DBSCAN sufre en >10 dims) |

---

*Template DBSCAN — ds-toolkit by andressonsino*
*Posición en el flujo: `06-modelado/`*
