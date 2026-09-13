# 🗺️ Hoja de Ruta — Proyecto de Ciencia de Datos

> **Uso:** Copiá este archivo al iniciar cualquier proyecto nuevo y completá cada sección  
> **Reemplazá** todo lo que está entre `[corchetes]` con información real del proyecto  
> **Principio guía:** el proceso importa tanto como el resultado técnico

---

## 0. Ficha del proyecto

| Campo | Detalle |
|---|---|
| **Nombre del proyecto** | [nombre-descriptivo-del-proyecto] |
| **Cliente / Organización** | [nombre del cliente o "proyecto personal"] |
| **Tipo de problema** | [clasificación / regresión / clustering / EDA / dashboard / otro] |
| **Fecha de inicio** | [DD/MM/AAAA] |
| **Fecha estimada de entrega** | [DD/MM/AAAA] |
| **Repositorio** | [github.com/andressonsino/nombre-repo] |
| **Estado** | [En planificación / En desarrollo / Entregado / En portfolio] |

---

## 1. FASE ESTRATÉGICA

> Completar esta fase **antes de tocar una línea de código**.  
> Un proyecto sin pregunta de negocio clara no tiene dirección — y eso se nota en el resultado.

---

### 1.1 Problema de negocio

**Pregunta principal que el proyecto debe responder:**
```
[Escribir UNA sola pregunta concreta. Ej: "¿Qué alumnos tienen riesgo de abandonar 
en los próximos 30 días?" No: "Analizar los datos de alumnos".]
```

**Por qué importa este problema:**
```
[Qué impacto tiene resolverlo para el cliente. Qué pasa si no se resuelve.]
```

**Alcance — qué SÍ entra:**
- [ ] [entregable 1]
- [ ] [entregable 2]
- [ ] [entregable 3]

**Alcance — qué NO entra (igual de importante):**
- [ ] [exclusión 1]
- [ ] [exclusión 2]

---

### 1.2 Datos disponibles

> Completar el inventario completo en `inventario.md` antes de continuar.  
> Registrar acá solo el resumen ejecutivo una vez terminado el relevamiento.

**Fuentes identificadas:** [cantidad]  
**Período cubierto:** [desde — hasta]  
**Clave identificadora principal:** [columna]  
**Anomalías críticas detectadas:** [resumen de 1-2 líneas]

---

### 1.3 Entregables acordados

| Entregable | Para quién | Formato | Fecha |
|---|---|---|---|
| [Informe ejecutivo] | [Cliente] | [PDF/PPT] | [fecha] |
| [Dashboard] | [Equipo operativo] | [Streamlit/PowerBI] | [fecha] |
| [Modelo en producción] | [Área técnica] | [API/script] | [fecha] |
| [Repo documentado] | [Portfolio propio] | [GitHub público] | [fecha] |

---

### 1.4 Gobernanza de datos y ética

**Datos que NO se van a usar aunque estén disponibles:**
```
[Ej: datos de salud, situaciones familiares, información financiera personal]
```

**Estrategia de anonimización:**
- [ ] Hash de IDs personales con `hashlib`
- [ ] Eliminación de nombres, correos, teléfonos desde el día 1
- [ ] Solo trabajar con datos agregados en entorno de desarrollo
- [ ] Nunca subir `data/raw/` a GitHub

**Acuerdo de confidencialidad:**
- [ ] Confirmar verbalmente con el cliente
- [ ] Mail de confirmación de alcance y uso del proyecto
- [ ] NDA formal (si el cliente lo requiere)

> Documentar esto en `DATA_GOVERNANCE.md` dentro del repo.  
> Cuando lo mostrés en entrevistas, demuestra criterio profesional real.

---

## 2. FASE TÉCNICA

---

### 2.1 Estructura del repositorio

```
[nombre-del-proyecto]/
├── README.md                        # resumen ejecutivo + cómo replicar
├── DATA_GOVERNANCE.md               # qué datos, anonimización, límites éticos
├── data/
│   ├── raw/                         # ← nunca subir a git (.gitignore)
│   └── processed/                   # datos anonimizados, sí versionables
├── notebooks/
│   ├── 01_limpieza.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_feature_engineering.ipynb
│   ├── 04_modelado.ipynb
│   └── 05_evaluacion.ipynb
├── src/                             # funciones reutilizables (no lógica en notebooks)
│   ├── data_processing.py
│   ├── features.py
│   ├── model.py
│   └── utils.py
├── reports/
│   ├── informe_ejecutivo.pdf
│   └── figures/
├── dashboard/                       # si aplica
├── requirements.txt
├── .gitignore
└── environment.yml                  # para reproducibilidad del entorno
```

---

### 2.2 Pipeline técnico

| Etapa | Qué hacer | Herramientas | Estado |
|---|---|---|---|
| **Ingesta** | Extracción desde la fuente, script reproducible | pandas, SQLAlchemy | ⬜ |
| **Anonimización** | Hash de IDs, eliminación de campos identificables | pandas, hashlib | ⬜ |
| **Limpieza** | Nulos, duplicados, outliers, tipos, validación | pandas | ⬜ |
| **EDA** | Distribuciones, correlaciones, hipótesis | matplotlib, seaborn | ⬜ |
| **Feature engineering** | Variables derivadas relevantes al problema | pandas, scikit-learn | ⬜ |
| **Modelado** | Baseline → 2-3 modelos → comparación | scikit-learn, XGBoost | ⬜ |
| **Evaluación** | Métricas alineadas al problema, interpretabilidad | scikit-learn, SHAP | ⬜ |
| **Entrega** | Informe ejecutivo + dashboard opcional | Streamlit, Quarto | ⬜ |
| **Documentación** | README con storytelling completo | Markdown | ⬜ |

> Cambiar ⬜ por ✅ a medida que completás cada etapa.

---

### 2.3 Decisiones técnicas clave

**Métrica de evaluación principal:**
```
[Ej: F1-Score porque hay desbalance de clases. NO accuracy.]
[Ej: RMSE porque el costo de errores grandes es mayor.]
[Justificar la elección antes de modelar.]
```

**Modelo baseline elegido:**
```
[Siempre empezar con el más simple posible. Ej: Regresión Logística para 
clasificación, Regresión Lineal para regresión, K-Means para clustering.]
```

**Modelos a comparar:**
- [ ] [Modelo 1 — baseline]
- [ ] [Modelo 2]
- [ ] [Modelo 3]

**Criterio de éxito del modelo:**
```
[Ej: "El modelo es aceptable si supera 0.75 de F1-Score en el set de validación".]
[Sin criterio definido antes, no sabés cuándo terminaste.]
```

---

### 2.4 Buenas prácticas a mantener en todo el proyecto

- [ ] Control de versiones desde el día 1, commits chicos con mensajes claros
- [ ] `data/raw/` en `.gitignore` desde el inicio — nunca subir datos crudos
- [ ] Notebooks numerados y ejecutables en orden de arriba hacia abajo
- [ ] Funciones reutilizables en `src/`, no copiar código entre notebooks
- [ ] `requirements.txt` actualizado en cada sesión de trabajo
- [ ] Al menos un test básico por función en `src/`

---

## 3. CRONOGRAMA

| Semana | Foco | Entregable interno |
|---|---|---|
| 1 | Propuesta, definición del problema, acceso a datos, governance doc | `DATA_GOVERNANCE.md` + propuesta firmada |
| 2 | Ingesta, anonimización, limpieza | `01_limpieza.ipynb` finalizado |
| 3 | EDA + feature engineering | `02_eda.ipynb` + hipótesis documentadas |
| 4-5 | Modelado + evaluación (iterar) | `04_modelado.ipynb` + comparativa de modelos |
| 6 | Informe ejecutivo + entrega al cliente | `reports/informe_ejecutivo.pdf` |
| 7 | Packaging del portfolio | README final + repo público |

> Ajustar semanas según la complejidad real del proyecto.

---

## 4. PACKAGING PARA EL PORTFOLIO

> Completar esta sección al cerrar el proyecto con el cliente.

**Checklist antes de hacer el repo público:**
- [ ] Ningún dato identificable en el repo (nombres, correos, teléfonos, DNI)
- [ ] `data/raw/` fuera del historial de git
- [ ] README cuenta la historia completa: contexto → problema → proceso → resultado → impacto
- [ ] 2-3 visualizaciones fuertes elegidas (no 20 gráficos sueltos)
- [ ] Métricas e impacto expresados con números cuando es posible (aunque sean aproximados)
- [ ] Mencionado explícitamente que son datos reales con autorización del cliente
- [ ] Demo o capturas si hay dashboard

**Storytelling del README (estructura sugerida):**

```markdown
## Contexto
[2 líneas: quién es el cliente, cuál es su actividad]

## Problema de negocio
[1 pregunta clara que el proyecto responde]

## Proceso
[Lista de etapas con lo más interesante de cada una]

## Resultado
[Métrica principal lograda + qué significa en términos de negocio]

## Impacto
[Qué cambió o puede cambiar para el cliente gracias a este análisis]

## Cómo replicar
[Instrucciones para correr el proyecto localmente]
```

**Testimonio del cliente (opcional pero muy valioso):**
```
[Pedirle al supervisor/cliente una línea para LinkedIn antes de cerrar el proyecto]
```

---

## 5. REGISTRO DE DECISIONES

> Documentar acá las decisiones importantes tomadas durante el proyecto y por qué.  
> Esto vale oro en una entrevista — podés explicar tus elecciones con fundamento.

| Fecha | Decisión | Alternativa descartada | Motivo |
|---|---|---|---|
| [fecha] | [decisión tomada] | [qué no se hizo] | [por qué] |
| [fecha] | | | |

---

## 6. NOTAS Y PENDIENTES

```
[Espacio libre para anotar cosas durante el proyecto]
- 
- 
- 
```

---

*Basado en el proyecto Dante Alighieri (2026) — replicable para cualquier cliente o caso de estudio*
