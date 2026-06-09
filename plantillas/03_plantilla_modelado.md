# 🤖 Plantilla Modelado — Machine Learning Estándar

> **Variable entrada:** `df_clean` (DataFrame limpio del notebook anterior)  
> **Uso:** Copiá cada bloque en una celda separada del notebook  
> **Orden:** Seguí las secciones de arriba hacia abajo  
> **Salida:** modelo entrenado + reporte de métricas

---

## Tabla de Contenidos

1. [Setup](#1-setup)
2. [Carga del dataset limpio](#2-carga-del-dataset-limpio)
3. [Preparación de features](#3-preparación-de-features)
4. [Encoding de variables categóricas](#4-encoding-de-variables-categóricas)
5. [Escalado de variables numéricas](#5-escalado-de-variables-numéricas)
6. [División train / test](#6-división-train--test)
7. [Entrenamiento del modelo](#7-entrenamiento-del-modelo)
8. [Evaluación del modelo](#8-evaluación-del-modelo)
9. [Comparar múltiples modelos](#9-comparar-múltiples-modelos)
10. [Desbalance de clases](#10-desbalance-de-clases)
11. [Guardar y cargar el modelo](#11-guardar-y-cargar-el-modelo)
12. [Qué modelo usar según el caso](#12-qué-modelo-usar-según-el-caso)

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
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler, LabelEncoder, OneHotEncoder
from sklearn.pipeline import Pipeline

# ─── Modelos de clasificación ───────────────────────────────────────────────
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import SVC

# ─── Modelos de regresión ───────────────────────────────────────────────────
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor

# ─── Métricas ───────────────────────────────────────────────────────────────
from sklearn.metrics import (
    # Clasificación
    accuracy_score, classification_report,
    confusion_matrix, roc_auc_score, roc_curve,
    # Regresión
    mean_absolute_error, mean_squared_error, r2_score
)

# ─── Desbalance de clases ───────────────────────────────────────────────────
from sklearn.utils.class_weight import compute_class_weight

# Supresión de warnings
warnings.filterwarnings('ignore')
pd.options.mode.chained_assignment = None

# Reproducibilidad — siempre fijar semilla
RANDOM_STATE = 42
```

---

## 2. Carga del dataset limpio

```python
df_clean = pd.read_csv('NOMBRE_ARCHIVO_limpio.csv')  # ← reemplazar
print(f'Dataset cargado: {df_clean.shape[0]} filas x {df_clean.shape[1]} columnas')
df_clean.head()
```

```python
# Revisar tipos y distribución de la variable objetivo
print(df_clean.dtypes)
print(f'\nDistribución de la variable objetivo:')
print(df_clean['TARGET'].value_counts())                    # ← reemplazar TARGET
print(df_clean['TARGET'].value_counts(normalize=True).round(3) * 100)
```

---

## 3. Preparación de features

**Definir X (features) e y (target) — el paso más importante del modelado.**

```python
# ─── Definir variable objetivo ───────────────────────────────────────────────
TARGET = 'COLUMNA_TARGET'                                   # ← reemplazar

# ─── Definir features ────────────────────────────────────────────────────────
# Opción A: todas las columnas menos el target
X = df_clean.drop(columns=[TARGET])
y = df_clean[TARGET]

# Opción B: selección manual de columnas
X = df_clean[['COL1', 'COL2', 'COL3']]                    # ← reemplazar
y = df_clean[TARGET]

# ─── Verificar ───────────────────────────────────────────────────────────────
print(f'Features (X): {X.shape}')
print(f'Target (y):   {y.shape}')
print(f'\nColumnas de X:\n{X.columns.tolist()}')
```

```python
# ─── Separar numéricas y categóricas dentro de X ────────────────────────────
cols_numericas = X.select_dtypes(include='number').columns.tolist()
cols_categoricas = X.select_dtypes(include='object').columns.tolist()

print(f'Numéricas:    {cols_numericas}')
print(f'Categóricas:  {cols_categoricas}')
```

---

## 4. Encoding de variables categóricas

**Hacer antes de escalar y antes de dividir. Los modelos no aceptan texto.**

```python
# ─── Label Encoding ──────────────────────────────────────────────────────────
# Cuándo: variable categórica ORDINAL (Baja=0, Media=1, Alta=2)
# o variable objetivo binaria (Sí/No, Fraude/No fraude)

le = LabelEncoder()
X['COLUMNA_CATEGORICA'] = le.fit_transform(X['COLUMNA_CATEGORICA'])    # ← reemplazar

# Para el target
y = le.fit_transform(y)
print(f'Clases: {le.classes_}')
```

```python
# ─── One Hot Encoding ────────────────────────────────────────────────────────
# Cuándo: variable categórica NOMINAL sin orden (ciudad, rubro, marca)

X = pd.get_dummies(X, columns=['COLUMNA_CATEGORICA'], drop_first=True)  # ← reemplazar
print(f'Shape después de OHE: {X.shape}')
print(X.head())
```

```python
# ─── Encoding de múltiples columnas categóricas ──────────────────────────────
# Cuándo: tenés varias columnas categóricas nominales

X = pd.get_dummies(X, columns=cols_categoricas, drop_first=True)
print(f'Shape después de encoding: {X.shape}')
```

> **Label Encoding vs One Hot Encoding:**
> - **Label Encoding** → la columna queda como 0, 1, 2... El modelo puede interpretar orden donde no hay. Usarlo solo para variables ordinales o el target.
> - **One Hot Encoding** → crea una columna por cada valor único. Sin orden implícito. Usarlo para variables nominales.

---

## 5. Escalado de variables numéricas

**Hacer después del encoding y antes de dividir. Obligatorio para modelos sensibles a escala.**

```python
# StandardScaler — media 0, desvío estándar 1
# Cuándo: regresión logística, SVM, KNN, redes neuronales

scaler = StandardScaler()
X[cols_numericas] = scaler.fit_transform(X[cols_numericas])
print('Escalado aplicado')
print(X[cols_numericas].describe().round(2))
```

> **¿Siempre hay que escalar?**
> - **Sí:** Logistic Regression, SVM, KNN, redes neuronales
> - **No necesario:** Decision Tree, Random Forest, Gradient Boosting
> - **Nunca escalar el target (y)** en clasificación

---

## 6. División train / test

**Hacer siempre después del encoding y escalado.**

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,              # 80% entrenamiento, 20% test
    random_state=RANDOM_STATE,
    stratify=y                  # mantiene proporción de clases — usar siempre en clasificación
)

print(f'Train: {X_train.shape[0]} filas')
print(f'Test:  {X_test.shape[0]} filas')
print(f'\nDistribución en train:\n{pd.Series(y_train).value_counts()}')
print(f'\nDistribución en test:\n{pd.Series(y_test).value_counts()}')
```

> **Sobre `stratify=y`:**  
> Sin stratify, el split aleatorio puede generar un test sin casos de una clase. Con `stratify=y` se garantiza que la proporción de clases es igual en train y test. Siempre usarlo en clasificación.

---

## 7. Entrenamiento del modelo

### 7.1 Clasificación

```python
# ─── Regresión Logística ─────────────────────────────────────────────────────
# Cuándo: baseline, datos linealmente separables, interpretabilidad importante
modelo = LogisticRegression(random_state=RANDOM_STATE, max_iter=1000)
modelo.fit(X_train, y_train)
print('Modelo entrenado')
```

```python
# ─── Decision Tree ───────────────────────────────────────────────────────────
# Cuándo: necesitás interpretabilidad total, visualizar reglas de decisión
modelo = DecisionTreeClassifier(
    max_depth=5,                # limitar profundidad evita overfitting
    random_state=RANDOM_STATE
)
modelo.fit(X_train, y_train)
```

```python
# ─── Random Forest ───────────────────────────────────────────────────────────
# Cuándo: datasets medianos/grandes, buen balance entre precisión y velocidad
# Es el modelo más usado como primera opción en clasificación
modelo = RandomForestClassifier(
    n_estimators=100,           # cantidad de árboles
    max_depth=10,
    random_state=RANDOM_STATE,
    n_jobs=-1                   # usa todos los núcleos disponibles
)
modelo.fit(X_train, y_train)
```

```python
# ─── Gradient Boosting ───────────────────────────────────────────────────────
# Cuándo: cuando Random Forest no alcanza la precisión necesaria
modelo = GradientBoostingClassifier(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=3,
    random_state=RANDOM_STATE
)
modelo.fit(X_train, y_train)
```

### 7.2 Regresión

```python
# ─── Regresión Lineal ────────────────────────────────────────────────────────
# Cuándo: baseline, relación lineal entre features y target
modelo = LinearRegression()
modelo.fit(X_train, y_train)
```

```python
# ─── Random Forest Regressor ─────────────────────────────────────────────────
# Cuándo: relación no lineal, mejor performance que regresión lineal
modelo = RandomForestRegressor(
    n_estimators=100,
    random_state=RANDOM_STATE,
    n_jobs=-1
)
modelo.fit(X_train, y_train)
```

---

## 8. Evaluación del modelo

### 8.1 Clasificación

```python
# Predicciones
y_pred = modelo.predict(X_test)
y_prob = modelo.predict_proba(X_test)[:, 1]  # probabilidad de clase positiva

# ─── Accuracy ────────────────────────────────────────────────────────────────
print(f'Accuracy: {accuracy_score(y_test, y_pred):.4f}')
```

```python
# ─── Reporte completo ────────────────────────────────────────────────────────
print(classification_report(y_test, y_pred))
```

```python
# ─── Matriz de confusión ─────────────────────────────────────────────────────
plt.figure(figsize=(8, 6))
cm = confusion_matrix(y_test, y_pred)
sns.heatmap(
    cm, annot=True, fmt='d',
    cmap='Blues',
    xticklabels=['Negativo', 'Positivo'],   # ← reemplazar con tus clases
    yticklabels=['Negativo', 'Positivo']
)
plt.title('Matriz de Confusión')
plt.ylabel('Real')
plt.xlabel('Predicho')
plt.tight_layout()
plt.show()
```

```python
# ─── Curva ROC ───────────────────────────────────────────────────────────────
# Cuándo: clasificación binaria, evaluar performance más allá del accuracy
fpr, tpr, _ = roc_curve(y_test, y_prob)
auc = roc_auc_score(y_test, y_prob)

plt.figure(figsize=(8, 6))
plt.plot(fpr, tpr, color='steelblue', linewidth=2, label=f'ROC (AUC = {auc:.4f})')
plt.plot([0, 1], [0, 1], color='gray', linestyle='--', label='Clasificador aleatorio')
plt.xlabel('Tasa de Falsos Positivos')
plt.ylabel('Tasa de Verdaderos Positivos')
plt.title('Curva ROC')
plt.legend()
plt.tight_layout()
plt.show()
```

```python
# ─── Importancia de features ─────────────────────────────────────────────────
# Cuándo: modelo basado en árboles (Decision Tree, Random Forest, Gradient Boosting)
importancias = pd.Series(
    modelo.feature_importances_,
    index=X.columns
).sort_values(ascending=False)

plt.figure(figsize=(12, 6))
sns.barplot(x=importancias.values[:15], y=importancias.index[:15],   # top 15
            hue=importancias.index[:15], legend=False, palette='Blues_r')
plt.title('Importancia de Features (Top 15)')
plt.xlabel('Importancia')
plt.tight_layout()
plt.show()
```

### 8.2 Regresión

```python
y_pred = modelo.predict(X_test)

mae  = mean_absolute_error(y_test, y_pred)
rmse = np.sqrt(mean_squared_error(y_test, y_pred))
r2   = r2_score(y_test, y_pred)

print(f'MAE:  {mae:.4f}')
print(f'RMSE: {rmse:.4f}')
print(f'R²:   {r2:.4f}')
```

```python
# Gráfico real vs predicho
plt.figure(figsize=(8, 6))
plt.scatter(y_test, y_pred, alpha=0.5, color='steelblue')
plt.plot([y_test.min(), y_test.max()],
         [y_test.min(), y_test.max()],
         color='red', linestyle='--', linewidth=1.5)
plt.xlabel('Valor real')
plt.ylabel('Valor predicho')
plt.title('Real vs Predicho')
plt.tight_layout()
plt.show()
```

> **Cómo interpretar métricas de clasificación:**
> - **Accuracy** → % de predicciones correctas. Engañoso con clases desbalanceadas.
> - **Precision** → de los que predije positivo, ¿cuántos eran realmente positivos?
> - **Recall** → de los positivos reales, ¿cuántos detecté?
> - **F1-Score** → balance entre Precision y Recall. Mejor métrica para clases desbalanceadas.
> - **AUC-ROC** → performance global del modelo. 0.5 = aleatorio, 1.0 = perfecto.

---

## 9. Comparar múltiples modelos

**Usar cuando no sabés qué modelo elegir. Entrenás varios y comparás.**

```python
modelos = {
    'Logistic Regression': LogisticRegression(random_state=RANDOM_STATE, max_iter=1000),
    'Decision Tree':        DecisionTreeClassifier(random_state=RANDOM_STATE),
    'Random Forest':        RandomForestClassifier(random_state=RANDOM_STATE, n_jobs=-1),
    'Gradient Boosting':    GradientBoostingClassifier(random_state=RANDOM_STATE),
    'KNN':                  KNeighborsClassifier()
}

resultados = []

for nombre, m in modelos.items():
    m.fit(X_train, y_train)
    y_pred = m.predict(X_test)
    resultados.append({
        'Modelo':    nombre,
        'Accuracy':  accuracy_score(y_test, y_pred),
        'F1-Score':  classification_report(y_test, y_pred, output_dict=True)['weighted avg']['f1-score'],
        'AUC-ROC':   roc_auc_score(y_test, m.predict_proba(X_test)[:, 1])
    })

df_resultados = pd.DataFrame(resultados).sort_values('AUC-ROC', ascending=False)
display(df_resultados.round(4))
```

```python
# Visualizar comparativa
plt.figure(figsize=(12, 5))
df_resultados_melted = df_resultados.melt(
    id_vars='Modelo',
    value_vars=['Accuracy', 'F1-Score', 'AUC-ROC']
)
sns.barplot(data=df_resultados_melted, x='Modelo', y='value', hue='variable')
plt.title('Comparativa de modelos')
plt.ylim(0, 1)
plt.xticks(rotation=15)
plt.legend(title='Métrica')
plt.tight_layout()
plt.show()
```

---

## 10. Desbalance de clases

**Usar cuando una clase tiene muchos más ejemplos que la otra (fraude, enfermedades, etc.).**

```python
# Detectar desbalance
print(pd.Series(y).value_counts())
print(pd.Series(y).value_counts(normalize=True).round(3) * 100)
```

```python
# ─── Opción A: class_weight='balanced' ──────────────────────────────────────
# Cuándo: desbalance moderado. El modelo penaliza más los errores en la clase minoritaria.
# Disponible en: LogisticRegression, RandomForest, DecisionTree, GradientBoosting

modelo = RandomForestClassifier(
    n_estimators=100,
    class_weight='balanced',    # ← agrega esta línea
    random_state=RANDOM_STATE,
    n_jobs=-1
)
modelo.fit(X_train, y_train)
```

```python
# ─── Opción B: pesos manuales ────────────────────────────────────────────────
# Cuándo: querés controlar exactamente cuánto pesa cada clase
clases = np.unique(y_train)
pesos = compute_class_weight('balanced', classes=clases, y=y_train)
dict_pesos = dict(zip(clases, pesos))
print(f'Pesos por clase: {dict_pesos}')

modelo = RandomForestClassifier(
    class_weight=dict_pesos,
    random_state=RANDOM_STATE
)
modelo.fit(X_train, y_train)
```

> **Cuándo hay desbalance:**  
> Con clases desbalanceadas el accuracy miente. Si el 98% son "no fraude", un modelo que predice siempre "no fraude" tiene 98% de accuracy pero es inútil. Mirá siempre **F1-Score** y **AUC-ROC** en estos casos.

---

## 11. Guardar y cargar el modelo

```python
import joblib

# Guardar
joblib.dump(modelo, 'modelo_entrenado.pkl')
print('Modelo guardado como modelo_entrenado.pkl')

# Guardar también el scaler si escalaste
joblib.dump(scaler, 'scaler.pkl')
```

```python
# Cargar en otro notebook o en producción
modelo_cargado = joblib.load('modelo_entrenado.pkl')
scaler_cargado = joblib.load('scaler.pkl')

# Predecir con datos nuevos
# X_nuevo = scaler_cargado.transform(X_nuevo)
# prediccion = modelo_cargado.predict(X_nuevo)
```

---

## 12. Qué modelo usar según el caso

### Por tipo de problema

| Problema | Modelos a probar (en orden) |
|----------|----------------------------|
| Clasificación binaria | Random Forest → Gradient Boosting → Logistic Regression |
| Clasificación multiclase | Random Forest → Gradient Boosting → Decision Tree |
| Regresión | Random Forest Regressor → Gradient Boosting → Linear Regression |

### Por prioridad

| Prioridad | Modelo |
|-----------|--------|
| Interpretabilidad (explicar al cliente) | Logistic Regression, Decision Tree |
| Mejor performance | Random Forest, Gradient Boosting |
| Dataset pequeño (< 1000 filas) | Logistic Regression, KNN |
| Dataset grande (> 100k filas) | Random Forest, Gradient Boosting |
| Baseline rápido | Logistic Regression |

### Encoding

| Tipo de variable | Encoding |
|-----------------|---------|
| Ordinal (Baja/Media/Alta) | Label Encoding |
| Nominal (ciudad, rubro) | One Hot Encoding |
| Variable objetivo binaria | Label Encoding |

### Escalado

| Escalar | No escalar |
|---------|-----------|
| Logistic Regression | Decision Tree |
| SVM | Random Forest |
| KNN | Gradient Boosting |
| Redes neuronales | — |

---

> **Orden fijo de modelado — nunca cambiar:**
> 1. Carga del dataset limpio
> 2. Definir X e y
> 3. Encoding de categóricas
> 4. Escalado de numéricas
> 5. División train/test con stratify
> 6. Entrenamiento
> 7. Evaluación con métricas + gráficos
> 8. Comparar modelos si es necesario
> 9. Guardar modelo final

---

*Para limpieza del dataset → `plantilla_limpieza.md`  
Para análisis exploratorio → `plantilla_eda.md`*
