# Hiperparámetros en Ciencia de Datos
## Reglas generales, rangos de referencia y criterios de calibración

---

## ¿Qué es un hiperparámetro?

Un **parámetro** es algo que el modelo aprende solo de los datos (los coeficientes de una regresión, los pesos de una red neuronal).

Un **hiperparámetro** es algo que vos definís antes de entrenar. El modelo no lo aprende: vos lo elegís y ese valor determina cómo va a aprender el modelo. Calibrarlos bien es la diferencia entre un modelo mediocre y uno útil.

---

## La regla más importante antes de calibrar cualquier cosa

> **Primero diagnosticá. Después calibrá.**

Antes de tocar un hiperparámetro, respondé estas dos preguntas:

| Pregunta | Señal | Diagnóstico | Qué hacer |
|---|---|---|---|
| ¿El modelo falla en train Y en test? | Error alto en ambos | **Underfitting** (sesgo alto) | Hacer el modelo más complejo |
| ¿El modelo acierta en train pero falla en test? | Error bajo en train, alto en test | **Overfitting** (varianza alta) | Hacer el modelo más simple o regularizar |
| ¿El modelo acierta en ambos? | Error similar en train y test | **Buen ajuste** | Calibrar fino para mejorar métricas |

Nunca calibrés a ciegas. Cada cambio de hiperparámetro debe responder a un diagnóstico.

---

## Reglas universales (aplican a casi todos los modelos)

### 1. Complejidad del modelo
Existe un trade-off entre complejidad y generalización. Más complejidad = mejor en train, peor en datos nuevos (si se pasa).

- **Underfitting** → aumentar complejidad (más profundidad, más estimadores, menos regularización)
- **Overfitting** → reducir complejidad (menos profundidad, más regularización, menos features)

### 2. Regularización
La regularización penaliza la complejidad para evitar overfitting. Cuanto mayor el coeficiente de regularización, más simple el modelo.

- Regularización **baja** → modelo más libre, más riesgo de overfitting
- Regularización **alta** → modelo más restringido, más riesgo de underfitting

### 3. Tamaño del dataset vs. complejidad
- Datasets **pequeños** (<1000 filas) → modelos simples, regularización alta, evitar modelos con muchos parámetros
- Datasets **medianos** (1k–100k filas) → modelos de complejidad media, calibración estándar
- Datasets **grandes** (>100k filas) → modelos complejos pueden funcionar bien, cuidado con el tiempo de entrenamiento

### 4. Datos desbalanceados
Si una clase aparece mucho más que otra (ej: 95% negativo, 5% positivo), la mayoría de los modelos tienen un hiperparámetro `class_weight` o `scale_pos_weight`. Ignorarlo lleva a modelos que siempre predicen la clase mayoritaria.

---

## Hiperparámetros por modelo

---

### Árboles de Decisión (`DecisionTreeClassifier / Regressor`)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `max_depth` | Profundidad máxima del árbol | 3–20 | Empezar con 3–5. Si underfitting, subir. Si overfitting, bajar |
| `min_samples_split` | Mínimo de muestras para dividir un nodo | 2–50 | Valores altos simplifican el árbol |
| `min_samples_leaf` | Mínimo de muestras en una hoja | 1–50 | Valores altos evitan hojas con muy pocos datos |
| `max_features` | Features consideradas en cada división | `"sqrt"`, `"log2"`, float | Menos features = más aleatorio = menos overfitting |
| `criterion` | Función de impureza | `"gini"`, `"entropy"` (clf) / `"squared_error"` (reg) | Generalmente `gini` para clasificación |

**Regla práctica:** un árbol profundo sin límite de `max_depth` casi siempre sobreajusta. Empezar siempre con `max_depth=5`.

---

### Random Forest (`RandomForestClassifier / Regressor`)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `n_estimators` | Cantidad de árboles | 100–1000 | Más árboles = más estable. 100 para explorar, 300–500 para producción |
| `max_depth` | Profundidad de cada árbol | 5–30 o `None` | `None` por defecto funciona bien. Restringir si hay overfitting |
| `max_features` | Features consideradas en cada división | `"sqrt"`, `"log2"`, float | `"sqrt"` para clasificación, `1/3` del total para regresión |
| `min_samples_leaf` | Mínimo de muestras por hoja | 1–20 | Subir si overfitting |
| `bootstrap` | Usar muestreo bootstrap | `True` / `False` | Casi siempre `True` |
| `class_weight` | Peso por clase | `"balanced"` o dict | Usar `"balanced"` con datos desbalanceados |

**Regla práctica:** Random Forest es robusto con sus valores por defecto. Antes de tocar hiperparámetros, probar con `n_estimators=100` y ver el resultado.

---

### Gradient Boosting (`GradientBoostingRegressor / Classifier`)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `n_estimators` | Cantidad de árboles (boosting rounds) | 100–3000 | Más rondas con learning rate bajo |
| `learning_rate` | Paso de cada árbol | 0.01–0.3 | **Relación inversa con n_estimators**: bajar uno implica subir el otro |
| `max_depth` | Profundidad de cada árbol | 3–8 | En boosting, árboles **chicos** (3–5) funcionan mejor que árboles grandes |
| `subsample` | Fracción del dataset por árbol | 0.6–1.0 | Valores < 1 introducen aleatoriedad (como stochastic GB), reduce overfitting |
| `min_samples_leaf` | Mínimo de muestras por hoja | 1–50 | Subir si overfitting |

**Regla del learning rate:** si bajás `learning_rate` de 0.1 a 0.01, necesitás ~10 veces más `n_estimators` para el mismo resultado.

---

### XGBoost (`XGBClassifier / XGBRegressor`)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `n_estimators` | Rondas de boosting | 100–3000 | Usar con `early_stopping_rounds` para que pare solo |
| `learning_rate` (`eta`) | Tasa de aprendizaje | 0.01–0.3 | Igual que Gradient Boosting |
| `max_depth` | Profundidad | 3–10 | Default es 6, empezar ahí |
| `min_child_weight` | Mínimo de peso en hojas | 1–10 | Subir si overfitting |
| `gamma` | Regularización L0 (poda de árboles) | 0–5 | 0 = sin poda. Subir si overfitting |
| `subsample` | Fracción del dataset por árbol | 0.5–1.0 | 0.8 es un buen punto de partida |
| `colsample_bytree` | Fracción de features por árbol | 0.5–1.0 | Igual que `max_features` en RF |
| `reg_alpha` (L1) | Regularización Lasso | 0–10 | Útil para eliminar features irrelevantes |
| `reg_lambda` (L2) | Regularización Ridge | 1–10 | Default 1, subir si overfitting |
| `scale_pos_weight` | Balance de clases | ratio negativo/positivo | Usar con datos desbalanceados |

**Regla práctica de XGBoost:** la combinación más común que funciona bien como punto de partida:
```python
learning_rate=0.05, n_estimators=500, max_depth=5,
subsample=0.8, colsample_bytree=0.8
```

---

### LightGBM (`LGBMClassifier / LGBMRegressor`)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `n_estimators` | Rondas de boosting | 100–3000 | Igual que XGBoost |
| `learning_rate` | Tasa de aprendizaje | 0.01–0.3 | Igual que XGBoost |
| `num_leaves` | Hojas por árbol (parámetro más importante de LGBM) | 20–300 | Default 31. **Más importante que max_depth en LGBM**. Subir si underfitting |
| `max_depth` | Profundidad máxima | -1 a 20 | -1 = sin límite (LGBM usa `num_leaves` como control principal) |
| `min_child_samples` | Mínimo de muestras por hoja | 10–200 | Subir si overfitting |
| `subsample` | Fracción del dataset | 0.5–1.0 | Igual que XGBoost |
| `colsample_bytree` | Fracción de features | 0.5–1.0 | Igual que XGBoost |
| `reg_alpha` (L1) | Regularización Lasso | 0–10 | Igual que XGBoost |
| `reg_lambda` (L2) | Regularización Ridge | 0–10 | Igual que XGBoost |
| `class_weight` | Balance de clases | `"balanced"` o dict | Datos desbalanceados |

**Diferencia clave LGBM vs XGBoost:** LGBM usa `num_leaves` como control principal de complejidad, no `max_depth`. Un árbol con `num_leaves=127` puede ser muy profundo aunque `max_depth` sea pequeño.

---

### Regresión Lineal con Regularización (Ridge / Lasso / ElasticNet)

| Modelo | Hiperparámetro | Qué controla | Regla |
|---|---|---|---|
| Ridge | `alpha` | Fuerza de regularización L2 | Mayor alpha = modelo más simple |
| Lasso | `alpha` | Fuerza de regularización L1 | Mayor alpha = más coeficientes forzados a 0 (selección de features) |
| ElasticNet | `alpha` + `l1_ratio` | Mezcla L1+L2 | `l1_ratio=1` = Lasso, `l1_ratio=0` = Ridge |

**Cuándo usar cada uno:**
- **Ridge:** cuando creés que todos los features aportan algo (ninguno es totalmente irrelevante)
- **Lasso:** cuando sospechás que muchos features son irrelevantes (Lasso los elimina automáticamente)
- **ElasticNet:** cuando tenés muchos features correlacionados entre sí

---

### K-Nearest Neighbors (`KNeighborsClassifier / Regressor`)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `n_neighbors` (k) | Cantidad de vecinos a considerar | 3–30 | k pequeño = más complejo, más overfitting. k grande = más simple |
| `weights` | Cómo ponderar los vecinos | `"uniform"`, `"distance"` | `"distance"` da más peso a vecinos más cercanos |
| `metric` | Distancia para medir cercanía | `"euclidean"`, `"manhattan"` | Euclidean es estándar |
| `p` | Parámetro de la métrica Minkowski | 1 (Manhattan), 2 (Euclidean) | |

**Regla clave de KNN:** KNN es muy sensible a la escala. **Siempre escalar antes** con StandardScaler. Sin escalado, una variable en miles domina sobre una en unidades y el modelo falla.

**Regla para k:** empezar con `k = sqrt(n_muestras)` como punto de referencia.

---

### Support Vector Machine (SVM)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `C` | Regularización (inversa): cuánto penalizar errores | 0.001–1000 | C alto = menos regularización, más overfitting |
| `kernel` | Tipo de transformación del espacio | `"linear"`, `"rbf"`, `"poly"` | `"rbf"` (radial) es el más versátil |
| `gamma` | Alcance de la influencia de cada punto (solo `rbf`) | `"scale"`, `"auto"`, float | gamma alto = cada punto influye poco = modelo más complejo |
| `degree` | Grado del polinomio (solo `poly`) | 2–5 | |

**Nota:** SVM es lento en datasets grandes (>50k filas). Para esos casos, XGBoost o LightGBM son mejores alternativas.

---

### K-Means (`KMeans`)

| Hiperparámetro | Qué controla | Rango típico | Regla |
|---|---|---|---|
| `n_clusters` (k) | Cantidad de grupos | 2–20 | Usar método del codo + Silhouette Score para elegir |
| `n_init` | Inicializaciones aleatorias | 10–20 | Más inicializaciones = más estable. Default 10 en sklearn |
| `max_iter` | Iteraciones máximas | 300–500 | Si no converge, subir |
| `random_state` | Semilla de aleatoriedad | cualquier int | **Siempre fijar** para reproducibilidad |
| `init` | Método de inicialización | `"k-means++"`, `"random"` | `"k-means++"` casi siempre es mejor |

**Cómo elegir k:**
1. **Método del codo:** graficar inercia vs k. El punto donde la curva "dobla" es k óptimo
2. **Silhouette Score:** valores entre -1 y 1. Cuanto más cerca de 1, mejor la separación. Elegir k con score máximo
3. **Regla de negocio:** a veces el negocio impone el k (ej: "queremos 3 segmentos de clientes")

---

## Métodos para calibrar (buscar el mejor hiperparámetro)

### GridSearchCV — búsqueda exhaustiva
Prueba todas las combinaciones posibles de los valores que vos definís.

```python
from sklearn.model_selection import GridSearchCV

grilla = {
    "max_depth": [3, 5, 7],
    "n_estimators": [100, 200, 300]
}
# Prueba 3 × 3 = 9 combinaciones

buscador = GridSearchCV(modelo, grilla, cv=5, scoring="r2")
buscador.fit(X_train, y_train)
print(buscador.best_params_)
```

**Cuándo usarlo:** cuando el espacio de búsqueda es chico (pocos hiperparámetros, pocos valores).
**Desventaja:** escala mal. 5 hiperparámetros con 5 valores cada uno = 3125 combinaciones.

---

### RandomizedSearchCV — búsqueda aleatoria
Prueba una cantidad fija de combinaciones elegidas al azar dentro de los rangos que definís.

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

espacio = {
    "max_depth": randint(3, 20),
    "n_estimators": randint(100, 500),
    "learning_rate": uniform(0.01, 0.3)
}

buscador = RandomizedSearchCV(
    modelo, espacio,
    n_iter=50,       # prueba 50 combinaciones aleatorias
    cv=5,
    scoring="r2",
    random_state=42
)
buscador.fit(X_train, y_train)
print(buscador.best_params_)
```

**Cuándo usarlo:** cuando el espacio de búsqueda es grande. Es más eficiente que GridSearch y generalmente encuentra resultados similares en menos tiempo.

---

### Early Stopping (XGBoost / LightGBM)
Deja de entrenar cuando la métrica de validación deja de mejorar. Sirve para encontrar el `n_estimators` óptimo automáticamente.

```python
modelo = XGBRegressor(
    learning_rate=0.05,
    n_estimators=3000,   # máximo posible
    early_stopping_rounds=50  # para si en 50 rondas no mejora
)
modelo.fit(
    X_train, y_train,
    eval_set=[(X_test, y_test)],
    verbose=False
)
print(f"Mejor n_estimators: {modelo.best_iteration}")
```

---

## Validación cruzada (Cross-Validation)

Nunca evalúes el modelo solo una vez con un split fijo. La validación cruzada divide el dataset en k partes y entrena/evalúa k veces, rotando la parte de test.

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(modelo, X, y, cv=5, scoring="r2")
print(f"R² promedio: {scores.mean():.4f} ± {scores.std():.4f}")
```

**Regla:** si el desvío estándar de los scores es alto, el modelo es inestable (sensible a qué datos usa para entrenar). Eso indica que necesita más regularización o más datos.

---

## Señales de alerta al calibrar

| Señal | Qué significa | Qué hacer |
|---|---|---|
| Score en train muy alto, en test muy bajo | Overfitting severo | Regularizar más, reducir complejidad |
| Score en train y test bajos | Underfitting | Aumentar complejidad, revisar features |
| Score muy variable entre folds de CV | Modelo inestable | Más datos, más regularización |
| El mejor modelo tiene hiperparámetros en el borde del rango | El rango de búsqueda es chico | Ampliar el rango y volver a buscar |
| Agregar más árboles no mejora nada | Ya convergió | No tiene sentido seguir subiendo `n_estimators` |
| El modelo tarda mucho en entrenar | Modelo o dataset demasiado grande | Reducir complejidad, usar subsampling |

---

## Regla de oro para el orden de calibración

Cuando no sabés por dónde empezar, seguí este orden:

1. **Fijar `random_state`** en todo — reproducibilidad ante todo
2. **Entrenar con valores por defecto** — es tu baseline
3. **Diagnosticar overfitting/underfitting** — antes de tocar nada
4. **Ajustar el hiperparámetro de complejidad principal** — `max_depth`, `num_leaves`, `C`
5. **Ajustar la regularización** — `alpha`, `reg_lambda`, `min_samples_leaf`
6. **Ajustar el learning rate con `n_estimators`** (solo boosting)
7. **Usar RandomizedSearchCV o GridSearchCV** para calibración fina
8. **Evaluar con validación cruzada** — no con un solo split

---

*Referencia de hiperparámetros — ds-toolkit by andressonsino*
*Posición en el flujo: `06-modelado/`*
