# 🔵 Plantilla Clustering — Segmentación No Supervisada

> **Variable entrada:** `df_clean` (DataFrame limpio del notebook de limpieza)  
> **Uso:** Copiá cada bloque en una celda separada del notebook  
> **Orden:** Seguí las secciones de arriba hacia abajo  
> **Salida:** `df_clean` con columna `cluster` asignada + visualizaciones

---

## Tabla de Contenidos

1. [Setup](#1-setup)
2. [Carga del dataset limpio](#2-carga-del-dataset-limpio)
3. [Preparación de features](#3-preparación-de-features)
4. [Escalado](#4-escalado)
5. [Encontrar el número óptimo de clusters](#5-encontrar-el-número-óptimo-de-clusters)
6. [Entrenar el modelo](#6-entrenar-el-modelo)
7. [Asignar clusters al DataFrame](#7-asignar-clusters-al-dataframe)
8. [Analizar y describir los clusters](#8-analizar-y-describir-los-clusters)
9. [Visualizar los clusters](#9-visualizar-los-clusters)
10. [Reducción de dimensiones para visualización](#10-reducción-de-dimensiones-para-visualización)
11. [Otros algoritmos de clustering](#11-otros-algoritmos-de-clustering)
12. [Guardar resultados](#12-guardar-resultados)
13. [Cuándo usar cada algoritmo](#13-cuándo-usar-cada-algoritmo)

---

> **Orden fijo de clustering — nunca cambiar:**
> 1. Carga del dataset limpio
> 2. Seleccionar features relevantes
> 3. Encoding si hay categóricas
> 4. Escalado — obligatorio
> 5. Encontrar k óptimo (codo + silhouette)
> 6. Entrenar modelo
> 7. Asignar clusters al DataFrame
> 8. Analizar y describir cada cluster
> 9. Visualizar (boxplots, heatmap, PCA)
> 10. Exportar dataset con clusters

---

## 1. Setup

```python
import pandas as pd
import numpy as np
import os
import warnings
import matplotlib.pyplot as plt
import seaborn as sns

# ─── Preprocesamiento ───────────────────────────────────────────────────────
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.pipeline import Pipeline

# ─── Clustering ─────────────────────────────────────────────────────────────
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.mixture import GaussianMixture

# ─── Evaluación ─────────────────────────────────────────────────────────────
from sklearn.metrics import silhouette_score, davies_bouldin_score, calinski_harabasz_score

# ─── Reducción de dimensiones (para visualización) ──────────────────────────
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE

# Supresión de warnings
warnings.filterwarnings('ignore')
pd.options.mode.chained_assignment = None

# Configuración de gráficos
sns.set_theme(style='whitegrid')
sns.set_palette('husl')
plt.rcParams['figure.figsize'] = (12, 6)
plt.rcParams['axes.titlesize'] = 16
plt.rcParams['axes.labelsize'] = 13

# Reproducibilidad
RANDOM_STATE = 42
```

---

## 2. Carga del dataset limpio

```python
df_clean = pd.read_csv('NOMBRE_ARCHIVO_limpio.csv')     # ← reemplazar
print(f'Dataset cargado: {df_clean.shape[0]} filas x {df_clean.shape[1]} columnas')
df_clean.head()
```

---

## 3. Preparación de features

**En clustering no hay variable objetivo (y). Solo trabajás con features (X).**

```python
# ─── Seleccionar columnas para clustering ────────────────────────────────────
# Solo columnas numéricas que aporten valor al agrupamiento
# No incluir IDs, fechas, ni columnas con demasiados nulos

X = df_clean[['COL1', 'COL2', 'COL3']]                 # ← reemplazar

# Alternativa: todas las numéricas
X = df_clean.select_dtypes(include='number').copy()

# Verificar
print(f'Shape de X: {X.shape}')
print(f'Columnas: {X.columns.tolist()}')
print(f'\nNulos en X: {X.isnull().sum().sum()}')
```

```python
# ─── Encoding de categóricas si las necesitás ───────────────────────────────
# Cuándo: querés incluir variables de texto en el clustering
# Usá siempre Label Encoding para clustering (no One Hot — genera demasiadas columnas)

le = LabelEncoder()
X['COLUMNA_CATEGORICA'] = le.fit_transform(              # ← reemplazar
    df_clean['COLUMNA_CATEGORICA']
)
```

> ⚠️ **En clustering no existe train/test split.** El modelo aprende sobre todos los datos disponibles porque no hay nada que predecir — solo agrupar.

---

## 4. Escalado

**Obligatorio en clustering. Sin escalar, las columnas con valores grandes dominan el agrupamiento.**

```python
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Convertir a DataFrame para mantener nombres de columnas
X_scaled = pd.DataFrame(X_scaled, columns=X.columns)

print('Escalado aplicado')
print(X_scaled.describe().round(2))
```

> **Por qué es obligatorio:**  
> Si una columna va de 0 a 1.000.000 (importe) y otra de 1 a 5 (satisfacción), sin escalar el importe domina completamente el agrupamiento y la satisfacción no aporta nada.

---

## 5. Encontrar el número óptimo de clusters

**Hacer siempre antes de entrenar. Nunca elegir el número de clusters arbitrariamente.**

### 5.1 Método del Codo (Elbow Method)

```python
# Probar de 2 a 10 clusters y graficar la inercia
inercias = []
rango_k = range(2, 11)

for k in rango_k:
    kmeans = KMeans(n_clusters=k, random_state=RANDOM_STATE, n_init=10)
    kmeans.fit(X_scaled)
    inercias.append(kmeans.inertia_)

plt.figure(figsize=(10, 5))
plt.plot(rango_k, inercias, marker='o', color='steelblue', linewidth=2)
plt.title('Método del Codo — Número óptimo de clusters')
plt.xlabel('Número de clusters (k)')
plt.ylabel('Inercia')
plt.xticks(rango_k)
plt.tight_layout()
plt.show()
```

### 5.2 Silhouette Score

```python
# El k con mayor score es el óptimo
silhouette_scores = []

for k in rango_k:
    kmeans = KMeans(n_clusters=k, random_state=RANDOM_STATE, n_init=10)
    labels = kmeans.fit_predict(X_scaled)
    score = silhouette_score(X_scaled, labels)
    silhouette_scores.append(score)
    print(f'k={k}: Silhouette = {score:.4f}')

plt.figure(figsize=(10, 5))
plt.plot(rango_k, silhouette_scores, marker='o', color='coral', linewidth=2)
plt.title('Silhouette Score por número de clusters')
plt.xlabel('Número de clusters (k)')
plt.ylabel('Silhouette Score')
plt.xticks(rango_k)
plt.tight_layout()
plt.show()
```

```python
# Elegir k óptimo
k_optimo = rango_k[silhouette_scores.index(max(silhouette_scores))]
print(f'\nK óptimo según Silhouette: {k_optimo}')
```

> **Cómo elegir k:**
> - **Método del Codo** → buscás dónde la curva deja de bajar bruscamente (el "codo")
> - **Silhouette Score** → el k con el valor más alto. Rango: [-1, 1]. Cuanto más cerca de 1, mejor.
> - Si ambos métodos sugieren el mismo k → esa es tu respuesta
> - Si difieren → usá el que tenga más sentido para el negocio

---

## 6. Entrenar el modelo

```python
# Entrenar K-Means con el k óptimo
K = k_optimo                                            # ← o reemplazar con tu número elegido

kmeans = KMeans(
    n_clusters=K,
    random_state=RANDOM_STATE,
    n_init=10,          # repite 10 veces con distintas semillas, elige el mejor
    max_iter=300
)
kmeans.fit(X_scaled)

print(f'Modelo entrenado con k={K}')
print(f'Inercia final: {kmeans.inertia_:.2f}')
```

```python
# ─── Métricas de calidad del modelo ─────────────────────────────────────────
labels = kmeans.labels_

silhouette = silhouette_score(X_scaled, labels)
davies     = davies_bouldin_score(X_scaled, labels)
calinski   = calinski_harabasz_score(X_scaled, labels)

print(f'Silhouette Score:         {silhouette:.4f}  (más alto = mejor, máx 1)')
print(f'Davies-Bouldin Score:     {davies:.4f}    (más bajo = mejor, mín 0)')
print(f'Calinski-Harabasz Score:  {calinski:.2f} (más alto = mejor)')
```

---

## 7. Asignar clusters al DataFrame

```python
# Agregar columna de cluster al DataFrame original
df_clean['cluster'] = kmeans.labels_

# Verificar distribución de clusters
print('Distribución de clusters:')
print(df_clean['cluster'].value_counts().sort_index())
print(df_clean['cluster'].value_counts(normalize=True).sort_index().round(3) * 100)
```

---

## 8. Analizar y describir los clusters

**La parte más importante del clustering — entender qué significa cada grupo.**

```python
# Estadísticas por cluster
resumen = df_clean.groupby('cluster')[X.columns].mean().round(2)
display(resumen)
```

```python
# Comparar con los valores originales (sin escalar) para interpretación
resumen_original = df_clean.groupby('cluster')[X.columns].agg(['mean', 'median']).round(2)
display(resumen_original)
```

```python
# Tamaño de cada cluster
print(df_clean.groupby('cluster').size().rename('Cantidad de registros'))
```

```python
# Heatmap de centroides — muestra el perfil de cada cluster
plt.figure(figsize=(12, 6))
sns.heatmap(
    resumen,
    annot=True,
    fmt='.2f',
    cmap='RdYlGn',
    linewidths=0.5
)
plt.title('Perfil promedio por cluster')
plt.ylabel('Cluster')
plt.tight_layout()
plt.show()
```

> **Cómo describir cada cluster:**  
> Buscás qué hace diferente a cada grupo. Ejemplo:
> - Cluster 0 → alto importe, baja satisfacción → "clientes de alto valor insatisfechos"
> - Cluster 1 → bajo importe, alta satisfacción → "clientes frecuentes y leales"
> - Cluster 2 → bajo importe, baja satisfacción → "clientes en riesgo de abandono"

---

## 9. Visualizar los clusters

### 9.1 Distribución por variable categórica

```python
# Ver cómo se distribuyen los clusters en una columna categórica del dataset original
plt.figure(figsize=(12, 6))
sns.countplot(
    data=df_clean,
    x='COLUMNA_CATEGORICA',         # ← reemplazar
    hue='cluster',
    palette='husl'
)
plt.title('Distribución de clusters por categoría')
plt.xticks(rotation=45, ha='right')
plt.legend(title='Cluster')
plt.tight_layout()
plt.show()
```

### 9.2 Boxplot de variable numérica por cluster

```python
plt.figure(figsize=(12, 6))
sns.boxplot(
    data=df_clean,
    x='cluster',
    y='COLUMNA_NUMERICA',           # ← reemplazar
    hue='cluster',
    legend=False,
    palette='husl'
)
plt.title('Distribución de variable por cluster')
plt.tight_layout()
plt.show()
```

### 9.3 Barplot de medias por cluster

```python
plt.figure(figsize=(12, 6))
resumen.T.plot(kind='bar', figsize=(14, 6), colormap='Set2', edgecolor='white')
plt.title('Media de cada variable por cluster')
plt.xlabel('Variable')
plt.ylabel('Valor promedio')
plt.xticks(rotation=45, ha='right')
plt.legend(title='Cluster')
plt.tight_layout()
plt.show()
```

---

## 10. Reducción de dimensiones para visualización

**Usar cuando tenés más de 2 features y querés ver los clusters en un plano 2D.**

### 10.1 PCA (más rápido)

```python
pca = PCA(n_components=2, random_state=RANDOM_STATE)
X_pca = pca.fit_transform(X_scaled)

varianza_explicada = pca.explained_variance_ratio_
print(f'Varianza explicada: {varianza_explicada.sum()*100:.1f}%')
print(f'  PC1: {varianza_explicada[0]*100:.1f}%')
print(f'  PC2: {varianza_explicada[1]*100:.1f}%')
```

```python
plt.figure(figsize=(10, 7))
scatter = plt.scatter(
    X_pca[:, 0], X_pca[:, 1],
    c=df_clean['cluster'],
    cmap='tab10',
    alpha=0.6,
    s=50
)
# Centroides
centroides_pca = pca.transform(kmeans.cluster_centers_)
plt.scatter(
    centroides_pca[:, 0], centroides_pca[:, 1],
    c='black', marker='X', s=200, zorder=5, label='Centroides'
)
plt.colorbar(scatter, label='Cluster')
plt.title('Clusters visualizados con PCA')
plt.xlabel(f'PC1 ({varianza_explicada[0]*100:.1f}% varianza)')
plt.ylabel(f'PC2 ({varianza_explicada[1]*100:.1f}% varianza)')
plt.legend()
plt.tight_layout()
plt.show()
```

### 10.2 t-SNE (mejor visualización, más lento)

```python
# Usar cuando PCA no muestra separación clara entre clusters
# Solo para visualización — no para el modelo

tsne = TSNE(n_components=2, random_state=RANDOM_STATE, perplexity=30)
X_tsne = tsne.fit_transform(X_scaled)

plt.figure(figsize=(10, 7))
scatter = plt.scatter(
    X_tsne[:, 0], X_tsne[:, 1],
    c=df_clean['cluster'],
    cmap='tab10',
    alpha=0.6,
    s=50
)
plt.colorbar(scatter, label='Cluster')
plt.title('Clusters visualizados con t-SNE')
plt.xlabel('t-SNE 1')
plt.ylabel('t-SNE 2')
plt.tight_layout()
plt.show()
```

> **PCA vs t-SNE:**
> - **PCA** → rápido, lineal, muestra cuánta varianza captura. Usarlo primero siempre.
> - **t-SNE** → lento, no lineal, mejor para visualizar estructuras complejas. Usar si PCA no separa bien.

---

## 11. Otros algoritmos de clustering

```python
# ─── DBSCAN ──────────────────────────────────────────────────────────────────
# Cuándo: clusters de forma irregular, detección de outliers
# No necesita definir k — lo encuentra solo
# -1 = outlier (no pertenece a ningún cluster)

dbscan = DBSCAN(
    eps=0.5,            # radio máximo para considerar vecinos — ajustar según datos
    min_samples=5       # mínimo de puntos para formar un cluster
)
df_clean['cluster_dbscan'] = dbscan.fit_predict(X_scaled)

print(f'Clusters encontrados: {df_clean["cluster_dbscan"].nunique() - 1}')
print(f'Outliers detectados: {(df_clean["cluster_dbscan"] == -1).sum()}')
```

```python
# ─── Agglomerative Clustering ────────────────────────────────────────────────
# Cuándo: cuando querés clustering jerárquico, datasets pequeños (< 10k filas)

agg = AgglomerativeClustering(n_clusters=K)
df_clean['cluster_agg'] = agg.fit_predict(X_scaled)

score_agg = silhouette_score(X_scaled, df_clean['cluster_agg'])
print(f'Silhouette Agglomerative: {score_agg:.4f}')
```

```python
# ─── Gaussian Mixture Models ─────────────────────────────────────────────────
# Cuándo: cuando los clusters tienen forma elíptica, no esférica

gmm = GaussianMixture(n_components=K, random_state=RANDOM_STATE)
df_clean['cluster_gmm'] = gmm.fit_predict(X_scaled)

score_gmm = silhouette_score(X_scaled, df_clean['cluster_gmm'])
print(f'Silhouette GMM: {score_gmm:.4f}')
```

```python
# ─── Comparar algoritmos ─────────────────────────────────────────────────────
algoritmos = {
    'K-Means':              kmeans.labels_,
    'Agglomerative':        df_clean['cluster_agg'],
    'Gaussian Mixture':     df_clean['cluster_gmm']
}

print('Comparativa de algoritmos:')
for nombre, labels in algoritmos.items():
    score = silhouette_score(X_scaled, labels)
    print(f'  {nombre}: Silhouette = {score:.4f}')
```

---

## 12. Guardar resultados

```python
import joblib

# Guardar modelo y scaler
joblib.dump(kmeans, 'modelo_kmeans.pkl')
joblib.dump(scaler, 'scaler_clustering.pkl')
print('Modelo y scaler guardados')
```

```python
# Exportar dataset con clusters asignados
nombre_salida = 'NOMBRE_DATASET_con_clusters.csv'        # ← reemplazar
df_clean.to_csv(nombre_salida, index=False)
print(f'Dataset exportado: {nombre_salida}')
```

```python
# Solo en Colab — descargar
# from google.colab import files
# files.download(nombre_salida)
```

---

## 13. Cuándo usar cada algoritmo

| Algoritmo | Cuándo usarlo | No usarlo cuando |
|-----------|--------------|-----------------|
| **K-Means** | Primera opción siempre. Clusters esféricos, datasets grandes | Clusters de forma irregular |
| **DBSCAN** | Hay outliers, clusters de forma arbitraria, no sabés cuántos clusters hay | Datos de alta dimensión, clusters de densidad muy distinta |
| **Agglomerative** | Dataset pequeño (< 10k), querés jerarquía de grupos | Dataset grande (muy lento) |
| **Gaussian Mixture** | Clusters elípticos, necesitás probabilidad de pertenencia | Clusters muy irregulares |

### Métricas de evaluación

| Métrica | Mejor valor | Qué mide |
|---------|------------|---------|
| **Silhouette Score** | Cercano a `1.0` | Qué tan separados y compactos son los clusters |
| **Davies-Bouldin** | Cercano a `0.0` | Similitud entre clusters (menor = mejor separados) |
| **Calinski-Harabasz** | Cuanto más alto | Ratio entre dispersión inter e intra cluster |

---

*Para limpieza del dataset → `plantilla_limpieza.md`  
Para análisis exploratorio → `plantilla_eda.md`  
Para modelado supervisado → `plantilla_modelado_supervisado.md`*
