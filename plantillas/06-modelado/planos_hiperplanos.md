# 📐 Planos e Hiperplanos en Ciencia de Datos

> **Nivel:** Conceptual → Aplicado  
> **Enfoque:** entender el concepto y reconocerlo en los algoritmos que ya usás

---

## Tabla de Contenidos

1. [Qué es un plano y qué es un hiperplano](#1-qué-es-un-plano-y-qué-es-un-hiperplano)
2. [La ecuación de un hiperplano](#2-la-ecuación-de-un-hiperplano)
3. [Dónde aparecen en ciencia de datos](#3-dónde-aparecen-en-ciencia-de-datos)
4. [SVM — el algoritmo que más los usa](#4-svm--el-algoritmo-que-más-los-usa)
5. [Regresión lineal como hiperplano](#5-regresión-lineal-como-hiperplano)
6. [Regresión logística como hiperplano](#6-regresión-logística-como-hiperplano)
7. [Redes neuronales y hiperplanos](#7-redes-neuronales-y-hiperplanos)
8. [Kernel trick — hiperplanos en dimensiones no visibles](#8-kernel-trick--hiperplanos-en-dimensiones-no-visibles)
9. [PCA y la reducción de dimensiones](#9-pca-y-la-reducción-de-dimensiones)
10. [Intuición visual por dimensión](#10-intuición-visual-por-dimensión)
11. [Resumen rápido](#11-resumen-rápido)

---

## 1. Qué es un plano y qué es un hiperplano

Empezando desde lo más básico:

- Un **punto** vive en 1 dimensión
- Una **línea** divide un plano 2D en dos mitades
- Un **plano** divide el espacio 3D en dos mitades
- Un **hiperplano** divide un espacio de N dimensiones en dos mitades

La generalización es directa: **un hiperplano en N dimensiones es un objeto de N-1 dimensiones que lo divide**.

| Espacio | Dimensiones del espacio | Cómo se llama el divisor | Dimensiones del divisor |
|---|---|---|---|
| Línea recta | 1D | Punto | 0D |
| Plano | 2D | Línea | 1D |
| Espacio 3D | 3D | Plano | 2D |
| Espacio ND | ND | Hiperplano | N-1 D |

En ciencia de datos casi nunca trabajamos con 2 o 3 dimensiones — un dataset con 20 variables vive en un espacio de 20 dimensiones. Por eso se usa el término hiperplano en lugar de línea o plano: es el caso general.

---

## 2. La ecuación de un hiperplano

La forma matemática es siempre la misma sin importar la dimensión:

```
w₁x₁ + w₂x₂ + ... + wₙxₙ + b = 0
```

O en notación vectorial:

```
w · x + b = 0
```

Donde:
- `x` → el punto en el espacio (un registro de tu dataset con N features)
- `w` → el vector normal (perpendicular al hiperplano) — define la orientación
- `b` → el bias o término independiente — desplaza el hiperplano del origen
- `w · x` → producto punto entre pesos y features

**Lo que hacen los algoritmos de ML es encontrar los valores de `w` y `b` que mejor separan o describen los datos.**

Los puntos caen en uno de tres lugares respecto al hiperplano:
```
w · x + b > 0  →  un lado (clase positiva, o por encima de la recta de regresión)
w · x + b < 0  →  el otro lado (clase negativa, o por debajo)
w · x + b = 0  →  exactamente sobre el hiperplano
```

---

## 3. Dónde aparecen en ciencia de datos

| Algoritmo | Rol del hiperplano |
|---|---|
| Regresión lineal | El hiperplano ES el modelo — la superficie que mejor predice y |
| Regresión logística | El hiperplano separa las dos clases en el espacio de features |
| SVM | Encuentra el hiperplano con mayor margen entre clases |
| Redes neuronales | Cada neurona aplica un hiperplano; la red combina muchos |
| PCA | Los componentes principales son hiperplanos de proyección |
| LDA | Encuentra el hiperplano que maximiza la separación entre clases |
| Perceptrón | El algoritmo más simple que aprende un único hiperplano separador |

---

## 4. SVM — el algoritmo que más los usa

SVM (Support Vector Machine) es el algoritmo donde el concepto de hiperplano es más explícito y central.

**La idea:**

Dado un conjunto de puntos etiquetados (clase A y clase B), SVM encuentra el hiperplano que:
1. Los separa correctamente
2. Maximiza el **margen** — la distancia entre el hiperplano y los puntos más cercanos de cada clase

```
Clase A  ●  ●  ●
                   |  ← margen
               ----+----  ← hiperplano óptimo
                   |  ← margen
Clase B  ■  ■  ■
```

Los puntos más cercanos al hiperplano se llaman **vectores de soporte** — de ahí el nombre del algoritmo.

**Por qué maximizar el margen importa:**

Un hiperplano que just barely separa las clases generalizará mal a datos nuevos. Uno con margen máximo es más robusto — pequeñas variaciones en los datos no cambian la clasificación.

**Hiperparámetro C:**

Controla el trade-off entre margen y errores de clasificación:
- C grande → margen pequeño, tolera menos errores (riesgo de overfitting)
- C chico → margen grande, tolera más errores (más generalización)

```python
from sklearn.svm import SVC

modelo = SVC(
    C=1.0,           # regularización — cuánto penalizar errores
    kernel='linear', # tipo de hiperplano (ver sección 8)
    random_state=42
)
modelo.fit(X_train, y_train)

# Ver los vectores de soporte
print(modelo.support_vectors_)

# Ver los coeficientes del hiperplano (solo para kernel='linear')
print(modelo.coef_)    # → vector w
print(modelo.intercept_)  # → b
```

**SVM para regresión (SVR):**

En lugar de separar clases, encuentra el hiperplano que contiene la mayor cantidad de puntos dentro de un margen ε (épsilon):

```python
from sklearn.svm import SVR

modelo = SVR(
    C=1.0,
    epsilon=0.1,     # margen de tolerancia alrededor del hiperplano
    kernel='rbf'
)
```

---

## 5. Regresión lineal como hiperplano

En regresión lineal, el modelo aprende un hiperplano que mejor describe la relación entre las features y el target:

```
ŷ = w₁x₁ + w₂x₂ + ... + wₙxₙ + b
```

Con 1 feature → es una recta (2D)  
Con 2 features → es un plano (3D)  
Con N features → es un hiperplano (N+1 D)

```python
from sklearn.linear_model import LinearRegression

modelo = LinearRegression()
modelo.fit(X_train, y_train)

# Los coeficientes definen el hiperplano
print(modelo.coef_)       # → vector w (uno por feature)
print(modelo.intercept_)  # → b (término independiente)
```

El algoritmo de entrenamiento (mínimos cuadrados) encuentra los valores de `w` y `b` que minimizan la distancia vertical entre el hiperplano y cada punto.

---

## 6. Regresión logística como hiperplano

Aunque el nombre dice "regresión", en la práctica es un clasificador. Aprende un hiperplano que separa las clases, y luego aplica la función sigmoide para convertir la distancia al hiperplano en una probabilidad:

```
P(clase=1) = σ(w · x + b) = 1 / (1 + e^(-(w·x+b)))
```

- Si `w · x + b` es muy positivo → probabilidad cercana a 1 (clase positiva)
- Si `w · x + b` es muy negativo → probabilidad cercana a 0 (clase negativa)
- Si `w · x + b = 0` → exactamente en el hiperplano → probabilidad = 0.5

```python
from sklearn.linear_model import LogisticRegression

modelo = LogisticRegression(random_state=42)
modelo.fit(X_train, y_train)

print(modelo.coef_)       # → vector w del hiperplano
print(modelo.intercept_)  # → b

# Distancia de cada punto al hiperplano (antes de la sigmoide)
distancias = modelo.decision_function(X_test)
```

**Diferencia clave con SVM:**

Logistic Regression minimiza el error de clasificación. SVM maximiza el margen. Con datos bien separables, SVM suele generalizar mejor; con datos ruidosos, Logistic Regression puede ser más robusta.

---

## 7. Redes neuronales y hiperplanos

Cada neurona de una red neuronal aplica un hiperplano seguido de una función de activación:

```
salida = activación(w · x + b)
```

Una red neuronal es esencialmente una **composición de múltiples hiperplanos**:
- Capa 1: aplica N₁ hiperplanos en paralelo
- Capa 2: aplica N₂ hiperplanos sobre el resultado de la capa anterior
- ...y así sucesivamente

Esto le da a la red la capacidad de modelar fronteras de decisión no lineales y complejas — algo que un solo hiperplano no puede hacer.

**El perceptrón simple** es la forma más básica: una sola neurona que aprende un único hiperplano separador. Es el ancestro directo de las redes neuronales modernas.

**Por qué las funciones de activación importan:**

Sin activación (o con activación lineal), composición de capas lineales = una sola capa lineal = un solo hiperplano. Las activaciones no lineales (ReLU, sigmoid, tanh) son lo que permite que la red construya fronteras de decisión complejas.

---

## 8. Kernel trick — hiperplanos en dimensiones no visibles

Algunos problemas no son linealmente separables en el espacio original:

```
● ● ■ ■ ● ●    ← no separable con una línea en 2D
```

El **kernel trick** proyecta los datos a un espacio de mayor dimensión donde sí son separables con un hiperplano, sin calcular explícitamente esa proyección (lo que sería muy costoso).

```
2D no separable  →  proyección implícita a 3D  →  separable con un plano
```

**Kernels comunes en SVM:**

| Kernel | Cuándo usarlo | Espacio resultante |
|---|---|---|
| `linear` | Datos linealmente separables | Igual dimensión |
| `poly` | Relaciones polinomiales entre features | Mayor dimensión |
| `rbf` | Caso general, el más usado | Dimensión infinita |
| `sigmoid` | Similar a redes neuronales | Mayor dimensión |

```python
from sklearn.svm import SVC

# Kernel lineal — hiperplano en el espacio original
modelo_lineal = SVC(kernel='linear', C=1.0)

# Kernel RBF — hiperplano en espacio de dimensión infinita
# gamma controla qué tan "local" es la influencia de cada punto
modelo_rbf = SVC(kernel='rbf', C=1.0, gamma='scale')

# Kernel polinomial de grado 3
modelo_poly = SVC(kernel='poly', degree=3, C=1.0)
```

**Cuándo elegir cada kernel:**

- Empezar siempre con `rbf` — funciona bien en la mayoría de los casos
- Probar `linear` si el dataset tiene muchas features (texto, por ejemplo)
- `poly` para relaciones explícitamente polinomiales
- Usar `GridSearchCV` para encontrar el kernel y C óptimos

---

## 9. PCA y la reducción de dimensiones

PCA (Principal Component Analysis) también trabaja con hiperplanos, pero con un objetivo distinto: no separar clases sino **proyectar datos** a una dimensión menor conservando la mayor varianza posible.

Cada componente principal es un hiperplano (en realidad un vector, pero define una dirección en el espacio) sobre el cual se proyectan los datos.

```python
from sklearn.decomposition import PCA

# Reducir a 2 dimensiones para visualizar
pca = PCA(n_components=2)
X_reducido = pca.fit_transform(X)

# Ver cuánta varianza explica cada componente
print(pca.explained_variance_ratio_)
# ej: [0.72, 0.15] → PC1 explica 72%, PC2 explica 15% → total 87%

# Los componentes son los hiperplanos de proyección
print(pca.components_)   # shape: (n_components, n_features)
```

**Relación con hiperplanos:**

PC1 es el hiperplano (dirección) de máxima varianza. PC2 es perpendicular a PC1 y captura la segunda mayor varianza. En N componentes, los datos se proyectan sobre N hiperplanos ortogonales entre sí.

---

## 10. Intuición visual por dimensión

```
1D (una feature):
    Clasificador:  punto en la recta
    |___●___|_____|  →  x < umbral = clase A; x > umbral = clase B

2D (dos features):
    Clasificador:  línea recta
    x₂ |  ● ● /
       |   ● / ■ ■
       |  / ■
       |/_________ x₁

3D (tres features):
    Clasificador:  plano
    Un plano que flota en el espacio 3D separando puntos de dos colores

ND (N features — lo que pasa en la práctica):
    Clasificador:  hiperplano
    Imposible de visualizar directamente.
    La matemática es la misma que en 2D, solo que con N coordenadas.
```

**Por qué esta intuición importa:**

Cuando un modelo de regresión logística o SVM "no converge" o tiene bajo rendimiento, muchas veces es porque los datos no son linealmente separables en el espacio actual de features — y la solución es o bien usar un kernel (SVM) o bien crear features nuevas que hagan el problema separable.

---

## 11. Resumen rápido

```
hiperplano = w · x + b = 0

w  →  pesos / coeficientes (definen la orientación)
b  →  bias / intercepto (desplaza el hiperplano del origen)
x  →  features del punto a clasificar o predecir

Regresión lineal    →  hiperplano que minimiza distancia a los puntos
Regresión logística →  hiperplano separador + sigmoide para probabilidad
SVM                 →  hiperplano separador con margen máximo
Perceptrón          →  hiperplano separador más simple (una neurona)
Red neuronal        →  composición de múltiples hiperplanos con activaciones
PCA                 →  hiperplanos de proyección para reducir dimensiones
Kernel trick        →  proyecta datos a dimensión mayor para hacer el problema separable
```

**Señales de que el concepto de hiperplano es relevante en un problema:**

- Estás usando SVM, Logistic Regression, Linear Regression o Perceptrón → siempre
- El modelo tiene bajo rendimiento y los datos podrían no ser linealmente separables → kernel trick o feature engineering
- Necesitás interpretar coeficientes del modelo → los coeficientes son el vector `w` del hiperplano
- Necesitás reducir dimensiones → PCA proyecta sobre hiperplanos ortogonales

---

*Relacionado: `plantilla_modelado.md` — implementación de SVM, regresión logística y reducción de dimensiones*
