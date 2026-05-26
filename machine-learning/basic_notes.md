# Machine Learning: Guía de Referencia para Ciencia de Datos

## ¿Qué es Machine Learning?

Machine Learning (ML) es una rama de la inteligencia artificial que le permite a una computadora **aprender patrones a partir de datos históricos** para hacer predicciones o tomar decisiones sobre datos nuevos, sin que el programador le escriba las reglas explícitamente.

> En vez de programar "si X entonces Y", le mostrás ejemplos y el algoritmo deduce las reglas solo.

---

## ¿Por qué es central en Ciencia de Datos?

La ciencia de datos sin ML se limita a describir lo que ya pasó. ML agrega la capacidad de **predecir lo que va a pasar** y de encontrar patrones que el ojo humano no puede detectar en grandes volúmenes de datos.

| Sin ML | Con ML |
|---|---|
| ¿Cuántos clientes se fueron el mes pasado? | ¿Qué clientes se van a ir el mes que viene? |

---

## Tipos de Machine Learning

### 1. Aprendizaje Supervisado
El modelo aprende a partir de datos **etiquetados** (con respuesta conocida).

- **Clasificación**: la salida es una categoría.
  - Ejemplo: ¿este mail es spam o no? ¿el tumor es benigno o maligno?
- **Regresión**: la salida es un número.
  - Ejemplo: ¿cuánto va a valer esta casa? ¿cuál será la temperatura mañana?

**Algoritmos comunes:** Regresión Lineal, Regresión Logística, Árboles de Decisión, Random Forest, SVM, Redes Neuronales.

---

### 2. Aprendizaje No Supervisado
El modelo trabaja con datos **sin etiquetar** y busca estructura por sí mismo.

- **Clustering**: agrupa datos similares.
  - Ejemplo: segmentar clientes por comportamiento de compra.
- **Reducción de dimensionalidad**: simplifica datos complejos sin perder información relevante.
  - Ejemplo: PCA.

**Algoritmos comunes:** K-Means, DBSCAN, PCA, Autoencoders.

---

### 3. Aprendizaje por Refuerzo
El modelo aprende por **prueba y error**, recibiendo recompensas o penalizaciones. Se usa en robótica, videojuegos y sistemas de recomendación avanzados.

---

## El Proceso Completo de un Proyecto de ML

```
1. Definir el problema
        ↓
2. Recolectar y explorar los datos (EDA)
        ↓
3. Preprocesar los datos (limpiar, transformar, escalar)
        ↓
4. Elegir y entrenar el modelo
        ↓
5. Evaluar el modelo
        ↓
6. Ajustar (tuning) e iterar
        ↓
7. Poner en producción (deploy)
```

---

## Conceptos Clave

### Train/Test Split
Los datos se dividen en dos partes:
- **Train set**: el modelo aprende con estos datos.
- **Test set**: se evalúa qué tan bien generaliza a datos que nunca vio.

> Si solo evaluás con los datos de entrenamiento, el modelo puede "memorizar" en vez de aprender. Eso se llama **overfitting**.

---

### Overfitting y Underfitting

- **Overfitting**: el modelo aprendió demasiado bien los datos de entrenamiento (incluyendo el ruido) y falla con datos nuevos. Es como estudiar las respuestas de memoria sin entender.
- **Underfitting**: el modelo es demasiado simple y no captura los patrones. Es como no estudiar nada.

El objetivo es el punto medio: un modelo que **generaliza** bien.

---

### Métricas de Evaluación

| Tipo de problema | Métricas |
|---|---|
| Clasificación | Accuracy, Precisión, Recall, F1-Score, AUC-ROC |
| Regresión | MAE, MSE, RMSE, R² |

> No alcanza con el accuracy: si el 95% de los mails no son spam, un modelo que diga "nunca es spam" tiene 95% de accuracy pero es inútil.

---

### Features e Ingeniería de Features

Los **features** son las variables de entrada del modelo. La **ingeniería de features** es el arte de crear, transformar o seleccionar las variables que más información le aportan al modelo. Es una de las habilidades más importantes y menos automatizables en ML.

---

## Cuándo Usar y Cuándo No Usar ML

| ✅ Usar ML | ❌ No usar ML |
|---|---|
| El problema es demasiado complejo para reglas manuales | El problema tiene reglas simples y claras |
| Tenés grandes volúmenes de datos históricos | Tenés pocos datos |
| Necesitás predicciones | Solo necesitás describir el pasado |
| Los patrones cambian con el tiempo | Necesitás explicabilidad total (legal, médico) |
| La precisión puede ser imperfecta | No podés tolerar errores |

---

## Bibliotecas Esenciales en Python

| Biblioteca | Para qué sirve |
|---|---|
| `NumPy` | Operaciones numéricas y arreglos |
| `Pandas` | Manipulación y análisis de datos |
| `Matplotlib` / `Seaborn` | Visualización |
| `Scikit-learn` | Algoritmos de ML clásicos, pipelines, métricas |
| `TensorFlow` / `PyTorch` | Redes neuronales y Deep Learning |
| `XGBoost` / `LightGBM` | Gradient boosting (muy usados en competencias) |

---

## ML vs Estadística Clásica

Una confusión común es pensar que ML reemplaza a la estadística. No es así:

- La **estadística clásica** busca entender y explicar relaciones entre variables. El modelo importa, la interpretabilidad es clave.
- El **ML** busca principalmente predecir con la mayor precisión posible. A veces el modelo es una "caja negra".

En ciencia de datos se usan los dos. La estadística para entender los datos y el negocio, ML para construir sistemas predictivos.

---

## Hoja de Ruta para Profundizar

1. **Regresión Lineal y Logística** — la base matemática de todo.
2. **Árboles de Decisión y Random Forest** — intuitivos y muy usados.
3. **Validación cruzada (Cross-Validation)** — evaluar modelos de forma robusta.
4. **Regularización (L1/L2)** — técnicas para evitar overfitting.
5. **Gradient Boosting (XGBoost)** — uno de los algoritmos más poderosos en la práctica.
6. **Redes Neuronales y Deep Learning** — para problemas complejos como imágenes y texto.
7. **Explicabilidad de modelos (SHAP, LIME)** — cada vez más importante en contextos reales.

---

## Recursos Recomendados

- 📘 **Libro gratuito**: *Hands-On Machine Learning with Scikit-Learn, Keras & TensorFlow* — Aurélien Géron
- 🎓 **Curso gratuito**: Machine Learning de Andrew Ng en Coursera (el más recomendado a nivel mundial)
- 💻 **Práctica**: [Kaggle.com](https://www.kaggle.com) — plataforma con datasets reales y competencias de ML
- 📖 **Documentación**: [scikit-learn.org](https://scikit-learn.org) — tutoriales claros con ejemplos en Python

---

*Documento elaborado como complemento de estudio para la Tecnicatura Superior en Ciencia de Datos e IA — IFTS 33 UOCRA*
