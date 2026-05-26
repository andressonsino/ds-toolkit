# 📊 Guía Completa de `loc` e `iloc` en Pandas

> **Nivel:** Básico → Avanzado  
> **Requisitos:** Python 3.x, pandas instalado (`pip install pandas`)

---

## Tabla de Contenidos

1. [¿Qué son loc e iloc?](#1-qué-son-loc-e-iloc)
2. [Diferencia clave entre los dos](#2-diferencia-clave-entre-los-dos)
3. [Preparación: el DataFrame de práctica](#3-preparación-el-dataframe-de-práctica)
4. [Uso de `loc` — básico a avanzado](#4-uso-de-loc--básico-a-avanzado)
5. [Uso de `iloc` — básico a avanzado](#5-uso-de-iloc--básico-a-avanzado)
6. [Comparativa directa loc vs iloc](#6-comparativa-directa-loc-vs-iloc)
7. [Selección con condiciones booleanas](#7-selección-con-condiciones-booleanas)
8. [Modificar datos con loc e iloc](#8-modificar-datos-con-loc-e-iloc)
9. [Casos avanzados y patrones útiles](#9-casos-avanzados-y-patrones-útiles)
10. [Errores comunes y cómo evitarlos](#10-errores-comunes-y-cómo-evitarlos)
11. [Resumen rápido (cheat sheet)](#11-resumen-rápido-cheat-sheet)

---

## 1. ¿Qué son loc e iloc?

En pandas, un DataFrame es una tabla con filas y columnas. Para acceder a partes específicas de esa tabla, pandas ofrece dos indexadores principales:

| Indexador | Nombre completo | ¿Qué usa para seleccionar? |
|-----------|-----------------|---------------------------|
| `.loc`    | Label-based Location | **Etiquetas** (nombres de índice y columnas) |
| `.iloc`   | Integer-based Location | **Posiciones enteras** (0, 1, 2, ...) |

Ambos siguen la misma sintaxis base:

```python
df.loc[filas, columnas]
df.iloc[filas, columnas]
```

---

## 2. Diferencia clave entre los dos

```
DataFrame con índice personalizado:

   nombre  edad  ciudad
A  Ana      28   Madrid
B  Bruno    34   Lima
C  Carla    22   Buenos Aires
D  Diego    45   México

                 ↑ índice (etiquetas: A, B, C, D)
```

- **`.loc`** → usa `A`, `B`, `C`, `D` para las filas y `"nombre"`, `"edad"` para las columnas
- **`.iloc`** → usa `0`, `1`, `2`, `3` para las filas y `0`, `1`, `2` para las columnas

> 💡 **Regla de oro:**  
> Si pensás en *nombres*, usá `loc`.  
> Si pensás en *posiciones numéricas*, usá `iloc`.

---

## 3. Preparación: el DataFrame de práctica

Usaremos este DataFrame a lo largo de toda la guía. Copialo y ejecutalo antes de los ejemplos:

```python
import pandas as pd
import numpy as np

data = {
    "nombre":   ["Ana",    "Bruno",  "Carla",       "Diego", "Elena",  "Franco"],
    "edad":     [28,       34,       22,             45,      31,       27],
    "ciudad":   ["Madrid", "Lima",   "Buenos Aires", "México","Bogotá", "Santiago"],
    "salario":  [3500,     4200,     2800,           6100,    3900,     3100],
    "activo":   [True,     False,    True,           True,    False,    True],
}

# Índice personalizado con letras
df = pd.DataFrame(data, index=["A", "B", "C", "D", "E", "F"])

print(df)
```

**Salida:**

```
   nombre  edad        ciudad  salario  activo
A     Ana    28        Madrid     3500    True
B   Bruno    34          Lima     4200   False
C   Carla    22  Buenos Aires     2800    True
D   Diego    45        México     6100    True
E   Elena    31        Bogotá     3900   False
F  Franco    27      Santiago     3100    True
```

---

## 4. Uso de `loc` — básico a avanzado

### 4.1 Seleccionar una fila por su etiqueta

```python
# Una sola fila → devuelve una Serie
df.loc["A"]
```
```
nombre         Ana
edad            28
ciudad      Madrid
salario       3500
activo        True
Name: A, dtype: object
```

```python
# Varias filas → devuelve un DataFrame
df.loc[["A", "C", "E"]]
```
```
  nombre  edad        ciudad  salario  activo
A    Ana    28        Madrid     3500    True
C  Carla    22  Buenos Aires     2800    True
E  Elena    31        Bogotá     3900   False
```

---

### 4.2 Seleccionar un rango de filas (INCLUSIVO en ambos extremos)

```python
# Desde "B" hasta "D" — ⚠️ incluye "D"
df.loc["B":"D"]
```
```
  nombre  edad        ciudad  salario  activo
B  Bruno    34          Lima     4200   False
C  Carla    22  Buenos Aires     2800    True
D  Diego    45        México     6100    True
```

> ⚠️ **Importante:** A diferencia de los slices de Python, `.loc` incluye el extremo final del rango.

---

### 4.3 Seleccionar columnas específicas

```python
# Una sola columna → devuelve Serie
df.loc["A", "nombre"]    # 'Ana' # devuelve un solo dato

# Varias columnas → devuelve DataFrame
df.loc["A", ["nombre", "salario"]] # devuelve varias columnas de una fila

# Rango de columnas
df.loc["A", "edad":"ciudad"] # devuelve rango de columnas de una fila
```

---

### 4.4 Seleccionar filas y columnas al mismo tiempo

```python
# Filas A, C, E — columnas nombre y salario
df.loc[["A", "C", "E"], ["nombre", "salario"]]
```
```
  nombre  salario
A    Ana     3500
C  Carla     2800
E  Elena     3900
```

```python
# Rango de filas + rango de columnas
df.loc["B":"D", "nombre":"salario"]
```
```
  nombre  edad        ciudad  salario
B  Bruno    34          Lima     4200
C  Carla    22  Buenos Aires     2800
D  Diego    45        México     6100
```

---

### 4.5 Seleccionar todas las filas o todas las columnas con `:`

```python
# Todas las filas, solo columnas nombre y edad
df.loc[:, ["nombre", "edad"]]

# Solo filas A y B, todas las columnas
df.loc[["A", "B"], :]
```

---

### 4.6 Uso con índice numérico (por defecto)

Cuando el DataFrame tiene índice numérico (0, 1, 2...), `loc` usa esos números como etiquetas:

```python
df2 = pd.DataFrame({"x": [10, 20, 30]})  # índice: 0, 1, 2

df2.loc[0]      # fila con etiqueta 0
df2.loc[0:1]    # filas 0 y 1 (¡ambos incluidos!)
```

---

## 5. Uso de `iloc` — básico a avanzado

### 5.1 Seleccionar una fila por posición

```python
# Primera fila (posición 0)
df.iloc[0]
```
```
nombre         Ana
edad            28
ciudad      Madrid
salario       3500
activo        True
Name: A, dtype: object
```

```python
# Última fila
df.iloc[-1]

# Penúltima fila
df.iloc[-2]
```

---

### 5.2 Seleccionar varias filas por posición

```python
# Filas en posición 0, 2 y 4
df.iloc[[0, 2, 4]]
```
```
  nombre  edad        ciudad  salario  activo
A    Ana    28        Madrid     3500    True
C  Carla    22  Buenos Aires     2800    True
E  Elena    31        Bogotá     3900   False
```

---

### 5.3 Slicing con iloc (NO inclusivo en el extremo final)

```python
# Filas 1, 2, 3 (posición 4 NO incluida)
df.iloc[1:4]
```
```
  nombre  edad        ciudad  salario  activo
B  Bruno    34          Lima     4200   False
C  Carla    22  Buenos Aires     2800    True
D  Diego    45        México     6100    True
```

```python
# Cada 2 filas (step)
df.iloc[::2]

# Filas en orden inverso
df.iloc[::-1]
```

---

### 5.4 Seleccionar columnas por posición

```python
# Primera columna (posición 0)
df.iloc[0, 0]      # 'Ana'

# Varias columnas
df.iloc[0, [0, 3]]    # nombre y salario de la primera fila
```

---

### 5.5 Seleccionar filas y columnas al mismo tiempo

```python
# Filas 0 a 2, columnas 0 a 2
df.iloc[0:3, 0:3]
```
```
  nombre  edad        ciudad
A    Ana    28        Madrid
B  Bruno    34          Lima
C  Carla    22  Buenos Aires
```

```python
# Filas 1, 3 — columnas 1 y 3
df.iloc[[1, 3], [1, 3]]
```
```
   edad  salario
B    34     4200
D    45     6100
```

```python
# Todas las filas, últimas 2 columnas
df.iloc[:, -2:]

# Todas las columnas, últimas 3 filas
df.iloc[-3:, :]
```

---

### 5.6 Slicing avanzado con step

```python
# Filas de 0 a 5, columnas de 2 en 2
df.iloc[0:6:2, 0::2]
```
```
  nombre        ciudad  activo
A    Ana        Madrid    True
C  Carla  Buenos Aires    True
E  Elena        Bogotá   False
```

---

## 6. Comparativa directa loc vs iloc

```python
# Mismo resultado, distinta forma de pedirlo

# Con loc (etiquetas)
df.loc["B":"D", "nombre":"ciudad"]

# Con iloc (posiciones)
df.iloc[1:4, 0:3]
```

| Situación | Usar |
|-----------|------|
| Conocés el nombre del índice o columna | `loc` |
| Trabajás con posiciones numéricas | `iloc` |
| El índice es una fecha, string, etc. | `loc` |
| Querés slicing estilo Python (0, 1, 2) | `iloc` |
| Necesitás el último elemento con `-1` | `iloc` |
| Filtrás con condiciones booleanas | `loc` |

---

## 7. Selección con condiciones booleanas

Esta es una de las funcionalidades más poderosas de `loc`.

### 7.1 Filtro simple

```python
# Personas con salario mayor a 3500
df.loc[df["salario"] > 3500]
```
```
  nombre  edad  ciudad  salario  activo
B  Bruno    34    Lima     4200   False
D  Diego    45  México     6100    True
E  Elena    31  Bogotá     3900   False
```

---

### 7.2 Filtro con múltiples condiciones

```python
# Activos Y con salario > 3000
df.loc[(df["activo"] == True) & (df["salario"] > 3000)]

# Mayores de 30 O salario mayor a 5000
df.loc[(df["edad"] > 30) | (df["salario"] > 5000)]

# Personas NO activas
df.loc[~df["activo"]]
```

> ⚠️ **Siempre usá paréntesis** alrededor de cada condición cuando combinás con `&`, `|` o `~`.

---

### 7.3 Filtro con condición + selección de columnas

```python
# De las personas activas, mostrar solo nombre y salario
df.loc[df["activo"] == True, ["nombre", "salario"]]
```
```
   nombre  salario
A     Ana     3500
C   Carla     2800
D   Diego     6100
F  Franco     3100
```

---

### 7.4 Uso de isin() con loc

```python
# Personas de ciudades específicas
ciudades_target = ["Madrid", "Lima", "Bogotá"]
df.loc[df["ciudad"].isin(ciudades_target)]
```

---

### 7.5 Uso de str con loc

```python
# Nombres que empiezan con "C"
df.loc[df["nombre"].str.startswith("C")]

# Ciudades que contienen "a"
df.loc[df["ciudad"].str.contains("a", case=False)]
```

---

## 8. Modificar datos con loc e iloc

### 8.1 Modificar una celda

```python
# Con loc
df.loc["A", "salario"] = 4000

# Con iloc
df.iloc[0, 3] = 4000
```

---

### 8.2 Modificar una fila completa

```python
df.loc["F"] = ["Franco", 28, "Montevideo", 3200, True]
```

---

### 8.3 Modificar varias celdas en un rango

```python
# Aumentar 10% el salario de las filas B a D
df.loc["B":"D", "salario"] = df.loc["B":"D", "salario"] * 1.10
```

---

### 8.4 Modificar con condición booleana

```python
# Desactivar a todos con salario menor a 3000
df.loc[df["salario"] < 3000, "activo"] = False

# Subir salario de inactivos
df.loc[~df["activo"], "salario"] += 500
```

---

### 8.5 Crear nuevas columnas con loc

```python
# Nueva columna "categoria" basada en condición
df.loc[df["salario"] >= 4000, "categoria"] = "senior"
df.loc[df["salario"] < 4000, "categoria"] = "junior"
```

---

### 8.6 Modificar con iloc

```python
# Cambiar toda la primera columna
df.iloc[:, 0] = df.iloc[:, 0].str.upper()

# Cambiar un bloque de valores
df.iloc[0:3, 3] = [9999, 9999, 9999]
```

---

## 9. Casos avanzados y patrones útiles

### 9.1 Encadenamiento con loc

```python
# Filtrar y luego seleccionar columnas
resultado = (
    df
    .loc[df["edad"] > 25]
    .loc[:, ["nombre", "ciudad", "salario"]]
)
```

---

### 9.2 loc con MultiIndex (índices jerárquicos)

```python
arrays = [
    ["Argentina", "Argentina", "Chile", "Chile"],
    ["Ana",       "Bruno",     "Carla", "Diego"]
]
index = pd.MultiIndex.from_arrays(arrays, names=["pais", "nombre"])

df_multi = pd.DataFrame(
    {"salario": [3500, 4200, 2800, 6100]},
    index=index
)

# Seleccionar todo un país
df_multi.loc["Argentina"]

# Seleccionar una persona específica
df_multi.loc[("Chile", "Carla")]

# Usar pd.IndexSlice para más control
idx = pd.IndexSlice
df_multi.loc[idx["Argentina":"Chile", :], :]
```

---

### 9.3 Combinar loc con numpy

```python
# Asignar NaN a valores que cumplen una condición
df.loc[df["salario"] < 3000, "salario"] = np.nan

# Rellenar NaN con un valor
df.loc[df["salario"].isna(), "salario"] = df["salario"].median()
```

---

### 9.4 Obtener índices booleanos para reusar

```python
# Guardá la máscara para usarla varias veces
mask_senior = df["salario"] >= 4000
mask_activo = df["activo"] == True

df.loc[mask_senior & mask_activo]
df.loc[mask_senior, "nombre"]
df.loc[~mask_activo, "salario"] += 300
```

---

### 9.5 Uso de .at y .iat (versiones optimizadas para una celda)

Cuando solo necesitás acceder a **una sola celda**, `.at` y `.iat` son más rápidos que `.loc` e `.iloc`:

```python
# .at → equivalente de loc para una celda
df.at["A", "nombre"]         # 'Ana'
df.at["A", "nombre"] = "Ana María"

# .iat → equivalente de iloc para una celda
df.iat[0, 0]                 # 'Ana'
df.iat[0, 0] = "Ana María"
```

> 💡 En loops o funciones que acceden a millones de celdas individuales, `.at` e `.iat` son notablemente más rápidos.

---

### 9.6 loc con query() como alternativa más legible

```python
# Equivalente a df.loc[(df["edad"] > 25) & (df["activo"] == True)]
df.query("edad > 25 and activo == True")

# Usar variables externas con @
edad_minima = 25
df.query("edad > @edad_minima")
```

---

### 9.7 Aplicar funciones con loc

```python
# Aplicar una función solo a un subconjunto
df.loc[df["activo"], "salario"] = df.loc[df["activo"], "salario"].apply(lambda x: round(x * 1.05, 2))
```

---

## 10. Errores comunes y cómo evitarlos

### ❌ Error 1: SettingWithCopyWarning

```python
# ❌ Incorrecto — genera advertencia o no modifica el original
subset = df[df["activo"] == True]
subset["salario"] = 9999   # ⚠️ SettingWithCopyWarning

# ✅ Correcto — usá loc directamente
df.loc[df["activo"] == True, "salario"] = 9999
```

---

### ❌ Error 2: Confundir iloc con loc en índice numérico

```python
df2 = pd.DataFrame({"val": [10, 20, 30]}, index=[5, 10, 15])

df2.loc[5]    # ✅ Fila con etiqueta 5 → val = 10
df2.iloc[0]   # ✅ Primera fila (posición 0) → val = 10

df2.loc[0]    # ❌ KeyError — no existe etiqueta 0 en este índice
```

---

### ❌ Error 3: Olvidar que loc incluye el final del slice

```python
# iloc: [1:4] → posiciones 1, 2, 3 (4 NO incluido) ← comportamiento Python estándar
df.iloc[1:4]   # filas en posición 1, 2, 3

# loc: ["B":"D"] → etiquetas B, C, D (D SÍ incluida)
df.loc["B":"D"]  # filas B, C, D
```

---

### ❌ Error 4: Condiciones sin paréntesis

```python
# ❌ Incorrecto — genera error de operador
df.loc[df["edad"] > 25 & df["activo"] == True]

# ✅ Correcto — paréntesis en cada condición
df.loc[(df["edad"] > 25) & (df["activo"] == True)]
```

---

### ❌ Error 5: Usar iloc con strings o loc con enteros de posición

```python
# ❌ iloc NO acepta strings
df.iloc["A"]         # TypeError

# ❌ Si el índice no tiene esa etiqueta, loc falla
df.loc[0]            # KeyError (si el índice no tiene "0")
```

---

## 11. Resumen rápido (cheat sheet)

### `loc` — Por etiquetas

```python
df.loc["A"]                          # Una fila
df.loc[["A", "C"]]                   # Varias filas
df.loc["A":"C"]                      # Rango (INCLUSIVO)
df.loc["A", "nombre"]                # Una celda
df.loc["A", ["nombre", "edad"]]      # Fila, varias columnas
df.loc[:, "nombre"]                  # Todas las filas, una columna
df.loc[condicion]                    # Filtro booleano
df.loc[condicion, ["col1", "col2"]]  # Filtro + columnas
df.loc["A", "salario"] = 5000        # Modificar celda
```

### `iloc` — Por posición

```python
df.iloc[0]                           # Primera fila
df.iloc[-1]                          # Última fila
df.iloc[[0, 2, 4]]                   # Filas en posición 0, 2, 4
df.iloc[1:4]                         # Rango (posición 4 NO incluida)
df.iloc[::2]                         # Una de cada dos filas
df.iloc[::-1]                        # Filas en orden inverso
df.iloc[0, 0]                        # Primera celda
df.iloc[0, [0, 2]]                   # Fila 0, columnas 0 y 2
df.iloc[0:3, 0:2]                    # Rango filas + rango columnas
df.iloc[0, 3] = 5000                 # Modificar celda
```

---

## Tips finales

- 🔍 Usá `.loc` por defecto cuando trabajes con nombres: es más legible y menos propenso a errores al reordenar columnas. Es más expresivo y evita errores cuando cambia el orden de las columnas.
- ⚡ Usá `.iloc` cuando necesités posiciones relativas (primeras N filas, últimas N, cada N), cuando la lógica sea puramente posicional o estés automatizando algo con números.
- 🛡️ Siempre que modifiques datos con una condición, preferí `.loc` directamente sobre el DataFrame original para evitar el `SettingWithCopyWarning`.
- 🧪 Ante la duda, probá con `.head()` o `print()` antes de modificar para asegurarte de estar seleccionando lo que querés.

---

*Guía elaborada para uso personal y académico · Pandas Documentation: [pandas.pydata.org](https://pandas.pydata.org/docs/)*
