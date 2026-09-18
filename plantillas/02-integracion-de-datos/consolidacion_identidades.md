# 🔗 Consolidación de Identidades — Identity Resolution

> **Objetivo:** Construir una tabla maestra de identidades únicas a partir de múltiples fuentes de datos  
> **Uso:** Copiá cada bloque en una celda separada del notebook  
> **Orden:** Seguí las secciones de arriba hacia abajo  
> **Salida:** `df_maestra` con una fila por persona única identificada

---

## Tabla de Contenidos

1. [Concepto y reglas de matching](#1-concepto-y-reglas-de-matching)
2. [Setup](#2-setup)
3. [Carga de fuentes](#3-carga-de-fuentes)
4. [Funciones de normalización](#4-funciones-de-normalización)
5. [Enfoque A — Fuente por fuente](#5-enfoque-a--fuente-por-fuente)
6. [Enfoque B — Concat primero](#6-enfoque-b--concat-primero)
7. [Verificación del resultado final](#7-verificación-del-resultado-final)
8. [Cuándo usar cada enfoque](#8-cuándo-usar-cada-enfoque)

---

## 1. Concepto y reglas de matching

**¿Qué es Identity Resolution?**

Es el proceso de consolidar registros de personas que vienen de múltiples fuentes en una sola tabla maestra sin duplicados. La misma persona puede aparecer en varias fuentes con datos inconsistentes — nombre escrito diferente, DNI con puntos, email en mayúscula. El objetivo es responder: **¿cuántas personas únicas hay en total?**

---

**Definir las reglas de matching ANTES de codear**

Esta es la decisión más importante del proceso. Las reglas determinan cuándo dos registros se consideran la misma persona.

```
Regla 1: Mismo DNI → misma persona
         (aunque el nombre esté escrito diferente o tenga emails distintos)

Regla 2: Sin DNI + mismo email → misma persona

Regla 3: Sin DNI + sin email → NO identificable → se descarta
         (solo nombre no es suficiente evidencia)
```

> ⚠️ **Documentar las reglas antes de empezar.** Cambiarlas a mitad del proceso obliga a rehacer todo desde el principio.

**Campos de identificación por jerarquía:**

| Prioridad | Campo | Motivo |
|---|---|---|
| 1 | DNI | Único por persona, no cambia |
| 2 | Email | Puede cambiar o haber varios por persona |
| 3 | Nombre | Ambiguo — puede repetirse entre personas distintas |

---

## 2. Setup

```python
import pandas as pd
import numpy as np
import unicodedata
import re
import warnings
from pathlib import Path

warnings.filterwarnings('ignore')
pd.options.mode.chained_assignment = None
pd.set_option('display.max_columns', None)
```

---

## 3. Carga de fuentes

### Desde Google Colab

```python
from google.colab import files

archivos_subidos = files.upload()
```

```python
# Función para buscar archivos por nombre parcial
def buscar_csv(comienzo_del_nombre):
    encontrados = list(Path('.').glob(comienzo_del_nombre + '*.csv'))
    if len(encontrados) == 0:
        raise FileNotFoundError('No se encontró: ' + comienzo_del_nombre)
    return encontrados[0]
```

```python
# Cargar cada fuente — reemplazar nombres según el proyecto
fuente_1 = pd.read_csv(buscar_csv('NOMBRE_FUENTE_1'))   # ← reemplazar
fuente_2 = pd.read_csv(buscar_csv('NOMBRE_FUENTE_2'))   # ← reemplazar
fuente_3 = pd.read_csv(buscar_csv('NOMBRE_FUENTE_3'))   # ← reemplazar
fuente_4 = pd.read_csv(buscar_csv('NOMBRE_FUENTE_4'))   # ← reemplazar

print('Archivos cargados correctamente.')
```

### Desde archivo local (Jupyter Lab / Miniconda)

```python
fuente_1 = pd.read_csv('NOMBRE_FUENTE_1.csv')           # ← reemplazar
fuente_2 = pd.read_csv('NOMBRE_FUENTE_2.csv')
fuente_3 = pd.read_csv('NOMBRE_FUENTE_3.csv')
fuente_4 = pd.read_csv('NOMBRE_FUENTE_4.csv')
```

### Exploración inicial de cada fuente

```python
# Ejecutar para cada fuente antes de normalizar
for nombre, df in [('fuente_1', fuente_1), ('fuente_2', fuente_2),
                   ('fuente_3', fuente_3), ('fuente_4', fuente_4)]:
    print(f'\n=== {nombre} ===')
    print(f'Shape: {df.shape}')
    print(f'Columnas: {df.columns.tolist()}')
    print(f'Nulos:\n{df.isnull().sum()}')
```

---

## 4. Funciones de normalización

> Normalizar SIEMPRE antes de comparar. Sin normalización, "Juan Pérez" y "juan perez" son strings distintos para Python.

```python
def limpiar_dni(valor):
    """
    Extrae exactamente 8 dígitos del DNI.
    Descarta puntos, guiones, espacios y cualquier formato no estándar.
    Si no tiene exactamente 8 dígitos → pd.NA
    """
    if pd.isna(valor):
        return pd.NA

    cont = 0
    valor = str(valor)
    new_valor = ""

    for i in valor:
        if i.isdigit():
            cont += 1
            new_valor += i

    if cont == 8:
        return new_valor
    else:
        return pd.NA
```

```python
def limpiar_email(valor):
    """
    Convierte a minúsculas y elimina espacios.
    Si no tiene @ → pd.NA (no es un email válido)
    """
    if pd.isna(valor):
        return pd.NA

    email = str(valor).strip().lower()

    if '@' not in email:
        return pd.NA
    else:
        return email
```

```python
def limpiar_nombre(nombre):
    """
    Convierte a minúsculas, elimina tildes y normaliza espacios múltiples.
    Permite comparar nombres con diferentes acentuaciones o formatos.
    """
    if pd.isna(nombre):
        return None

    nombre = nombre.lower().strip()

    # Eliminar tildes y diacríticos
    nombre = unicodedata.normalize("NFD", nombre)
    nombre = "".join(
        letra for letra in nombre
        if unicodedata.category(letra) != "Mn"
    )

    # Normalizar espacios múltiples
    nombre = re.sub(r"\s+", " ", nombre)

    return nombre
```

```python
# Verificar que las funciones funcionan correctamente
assert limpiar_dni("12.345.678") == "12345678"
assert limpiar_dni("1234567") is pd.NA       # menos de 8 dígitos
assert limpiar_email("USER@Mail.COM") == "user@mail.com"
assert limpiar_email("no-es-email") is pd.NA
assert limpiar_nombre("María Pérez") == "maria perez"
print("Funciones de normalización verificadas correctamente.")
```

---

## 5. Enfoque A — Fuente por fuente

**Cuándo usarlo:** cuando las fuentes son muy diferentes entre sí, cuando necesitás control total sobre el origen de cada registro, o cuando el dataset es grande y necesitás trazabilidad completa.

### 5.1 Fuente base (la más confiable — generalmente el CRM)

```python
# Exploración
fuente_1.head()
fuente_1.shape
```

```python
# Eliminar duplicados exactos
fuente_1 = fuente_1.drop_duplicates()
print(f'Shape tras drop_duplicates: {fuente_1.shape}')
```

```python
# Normalizar campos identificadores
fuente_1['dni_limpio']    = fuente_1['COL_DNI'].apply(limpiar_dni)       # ← reemplazar COL_DNI
fuente_1['email_limpio']  = fuente_1['COL_EMAIL'].apply(limpiar_email)   # ← reemplazar COL_EMAIL
fuente_1['nombre_limpio'] = fuente_1['COL_NOMBRE'].apply(limpiar_nombre) # ← reemplazar COL_NOMBRE
```

```python
# Ver nulos DESPUÉS de normalizar — pueden aparecer nuevos (DNIs con 7 dígitos, etc.)
print('Nulos post-normalización:')
print(fuente_1[['dni_limpio', 'email_limpio', 'nombre_limpio']].isnull().sum())
```

```python
# Detectar DNIs duplicados (misma persona con distintas filas)
fuente_1[fuente_1['dni_limpio'].notna() & fuente_1['dni_limpio'].duplicated(keep=False)]
```

```python
# Inspección manual: cuántos nombres distintos tiene cada DNI
nombres_por_dni = fuente_1.groupby('dni_limpio')['nombre_limpio'].nunique()
print(f'DNIs con más de un nombre: {nombres_por_dni[nombres_por_dni > 1].count()}')
fuente_1[fuente_1['dni_limpio'].isin(
    nombres_por_dni[nombres_por_dni > 1].index
)].groupby('dni_limpio')['nombre_limpio'].unique()
```

```python
# Si hay un registro duplicado específico a eliminar por su ID
# fuente_1 = fuente_1.drop(fuente_1[fuente_1['ID_COL'] == 'VALOR_A_ELIMINAR'].index)
```

```python
# Construir tabla maestra inicial con columnas clave
df_maestra = fuente_1[['ID_COL', 'dni_limpio', 'email_limpio', 'nombre_limpio']]  # ← reemplazar ID_COL
print(f'Maestra inicial: {df_maestra.shape[0]} registros')
```

---

### 5.2 Fuentes adicionales (ventas, soporte, newsletter, etc.)

> Repetir este bloque para cada fuente adicional. El patrón es siempre el mismo.

```python
# Normalizar
fuente_2['dni_limpio']    = fuente_2['COL_DNI'].apply(limpiar_dni)       # ← reemplazar
fuente_2['email_limpio']  = fuente_2['COL_EMAIL'].apply(limpiar_email)   # ← reemplazar
fuente_2['nombre_limpio'] = fuente_2['COL_NOMBRE'].apply(limpiar_nombre) # ← reemplazar
```

```python
# Quedarse solo con identificables
fuente_2_id = fuente_2[
    fuente_2['dni_limpio'].notna() | fuente_2['email_limpio'].notna()
]
print(f'Identificables: {fuente_2_id.shape[0]}')
```

```python
# Eliminar duplicados por par (dni + email juntos)
fuente_2_id = fuente_2_id.drop_duplicates(subset=['dni_limpio', 'email_limpio'])
print(f'Tras drop_duplicates por par: {fuente_2_id.shape[0]}')
```

```python
# Seleccionar columnas necesarias
fuente_2_id = fuente_2_id[['dni_limpio', 'email_limpio', 'nombre_limpio']]
```

```python
# ── Procesar los CON DNI ─────────────────────────────────────────────────────
fuente_2_con_dni = fuente_2_id[fuente_2_id['dni_limpio'].notna()].copy()

# Mapear todos los emails asociados a cada DNI antes de deduplicar
emails_por_dni = fuente_2_con_dni.groupby('dni_limpio')['email_limpio'].apply(list)
fuente_2_con_dni['emails_asociados'] = fuente_2_con_dni['dni_limpio'].map(emails_por_dni)

# Deduplicar por DNI (ahora sin perder emails)
fuente_2_con_dni = fuente_2_con_dni.drop_duplicates(subset=['dni_limpio'])
print(f'DNIs únicos en fuente_2: {fuente_2_con_dni.shape[0]}')
```

```python
# Actualizar emails_asociados en maestra para DNIs que ya existen
# (agrega emails nuevos encontrados en esta fuente a DNIs ya conocidos)
for _, fila in fuente_2_con_dni.iterrows():
    dni = fila['dni_limpio']
    emails_nuevos = fila['emails_asociados']

    if dni in df_maestra['dni_limpio'].values:
        idx = df_maestra[df_maestra['dni_limpio'] == dni].index[0]
        emails_actuales = df_maestra.loc[idx, 'emails_asociados']

        if isinstance(emails_actuales, list):
            df_maestra.at[idx, 'emails_asociados'] = list(
                set(emails_actuales + emails_nuevos)
            )
        else:
            df_maestra.at[idx, 'emails_asociados'] = emails_nuevos
```

```python
# Filtrar solo los DNIs que NO están ya en la maestra
fuente_2_nuevos_dni = fuente_2_con_dni[
    ~fuente_2_con_dni['dni_limpio'].isin(df_maestra['dni_limpio'].dropna())
]
print(f'DNIs nuevos para agregar: {fuente_2_nuevos_dni.shape[0]}')
```

```python
# Concatenar nuevos a la maestra
df_maestra = pd.concat([df_maestra, fuente_2_nuevos_dni], ignore_index=True)
print(f'Maestra tras agregar fuente_2 (DNI): {df_maestra.shape[0]}')
```

```python
# ── Procesar los SIN DNI pero CON EMAIL ──────────────────────────────────────
fuente_2_sin_dni = fuente_2_id[fuente_2_id['dni_limpio'].isna()].copy()

# Verificar que el email no esté en email_limpio de la maestra
fuente_2_sin_dni = fuente_2_sin_dni[
    ~fuente_2_sin_dni['email_limpio'].isin(df_maestra['email_limpio'].dropna())
]

# Verificar que el email no esté en emails_asociados de la maestra
emails_asociados_maestra = df_maestra['emails_asociados'].dropna().explode()
fuente_2_sin_dni = fuente_2_sin_dni[
    ~fuente_2_sin_dni['email_limpio'].isin(emails_asociados_maestra)
]
print(f'Sin DNI, email nuevo no visto antes: {fuente_2_sin_dni.shape[0]}')
```

```python
# Concatenar a la maestra
df_maestra = pd.concat([df_maestra, fuente_2_sin_dni], ignore_index=True)
print(f'Maestra tras agregar fuente_2 (email): {df_maestra.shape[0]}')
```

> 🔁 **Repetir el bloque 5.2 completo para cada fuente adicional** (fuente_3, fuente_4, etc.)  
> El patrón siempre es: normalizar → con DNI → sin DNI pero con email → concatenar

---

### 5.3 Limpieza final de la maestra

```python
# Eliminar registros con DNI nulo cuyo email ya aparece en emails_asociados
# (pueden haber quedado del primer concat)
emails_asociados_maestra = df_maestra['emails_asociados'].dropna().explode()

df_maestra = df_maestra[
    df_maestra['dni_limpio'].notna() |
    (
        df_maestra['dni_limpio'].isna() &
        ~df_maestra['email_limpio'].isin(emails_asociados_maestra)
    )
]
print(f'Maestra final: {df_maestra.shape[0]} identidades únicas')
```

---

## 6. Enfoque B — Concat primero

**Cuándo usarlo:** cuando las fuentes son similares entre sí, cuando la velocidad de desarrollo importa más que la trazabilidad, o cuando el dataset es pequeño-mediano.

```python
# Renombrar columnas de cada fuente para que coincidan
fuente_1_xa_concat = fuente_1[['ID_COL', 'COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']]      # ← reemplazar

fuente_2_xa_concat = fuente_2[['COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']]                 # ← reemplazar
fuente_2_xa_concat.columns = ['nombre', 'dni', 'email']

fuente_3_xa_concat = fuente_3[['COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']]                 # ← reemplazar
fuente_3_xa_concat.columns = ['nombre', 'dni', 'email']

fuente_4_xa_concat = fuente_4[['COL_NOMBRE', 'COL_EMAIL']]                            # ← reemplazar (si no tiene DNI)
```

```python
# Concatenar todo de una vez
df_total = pd.concat(
    [fuente_1_xa_concat, fuente_2_xa_concat, fuente_3_xa_concat, fuente_4_xa_concat],
    ignore_index=True
)
print(f'Total combinado: {df_total.shape[0]} filas')
```

```python
# Normalizar sobre el df combinado
df_total['nombre_limpio'] = df_total['nombre'].apply(limpiar_nombre)
df_total['dni_limpio']    = df_total['dni'].apply(limpiar_dni)
df_total['email_limpio']  = df_total['email'].apply(limpiar_email)
```

```python
# Eliminar duplicados exactos por los tres campos limpios
df_total = df_total.drop_duplicates(subset=['dni_limpio', 'email_limpio', 'nombre_limpio'])
print(f'Tras eliminar duplicados exactos: {df_total.shape[0]}')
```

```python
# Separar los CON DNI
df_total_con_dni = df_total[df_total['dni_limpio'].notna()].copy()

# Mapear nombres y emails asociados a cada DNI
nombres_por_dni = df_total_con_dni.groupby('dni_limpio')['nombre_limpio'].apply(list)
df_total_con_dni['nombres_asociados'] = df_total_con_dni['dni_limpio'].map(nombres_por_dni)

emails_por_dni = df_total_con_dni.groupby('dni_limpio')['email_limpio'].apply(list)
df_total_con_dni['emails_asociados'] = df_total_con_dni['dni_limpio'].map(emails_por_dni)

# Deduplicar por DNI
df_total_con_dni = df_total_con_dni.drop_duplicates(subset=['dni_limpio'])
print(f'DNIs únicos: {df_total_con_dni.shape[0]}')
```

```python
# Separar los SIN DNI pero CON EMAIL
df_total_sin_dni = df_total[
    df_total['dni_limpio'].isna() & df_total['email_limpio'].notna()
].copy()

# Filtrar emails ya presentes en email_limpio de los con DNI
df_total_sin_dni = df_total_sin_dni[
    ~df_total_sin_dni['email_limpio'].isin(df_total_con_dni['email_limpio'].dropna())
]

# Filtrar emails ya presentes en emails_asociados
emails_asociados_explode = df_total_con_dni['emails_asociados'].dropna().explode()
df_total_sin_dni = df_total_sin_dni[
    ~df_total_sin_dni['email_limpio'].isin(emails_asociados_explode.dropna())
]
print(f'Sin DNI, email nuevo: {df_total_sin_dni.shape[0]}')
```

```python
# Tabla maestra final
df_maestra = pd.concat([df_total_con_dni, df_total_sin_dni], ignore_index=True)
print(f'Maestra final: {df_maestra.shape[0]} identidades únicas')
```

```python
# Caso borde: registros sin DNI y sin email — solo nombre
# Por defecto se descartan, pero se puede intentar matching por nombre
df_sin_dni_sin_email = df_total[
    df_total['dni_limpio'].isna() & df_total['email_limpio'].isna()
].copy()
df_sin_dni_sin_email = df_sin_dni_sin_email.drop_duplicates(subset=['nombre_limpio'])

# Filtrar nombres que ya están en la maestra
nombres_maestra = df_maestra['nombre_limpio'].dropna()
nombres_asociados_explode = df_maestra['nombres_asociados'].dropna().explode() \
    if 'nombres_asociados' in df_maestra.columns else pd.Series(dtype=str)

df_sin_dni_sin_email = df_sin_dni_sin_email[
    ~df_sin_dni_sin_email['nombre_limpio'].isin(nombres_maestra) &
    ~df_sin_dni_sin_email['nombre_limpio'].isin(nombres_asociados_explode)
]
print(f'Sin DNI ni email, nombre nuevo: {df_sin_dni_sin_email.shape[0]}')
# Si decidís agregarlos: df_maestra = pd.concat([df_maestra, df_sin_dni_sin_email])
```

---

## 7. Verificación del resultado final

```python
# Resumen de la maestra
print('=== RESULTADO FINAL ===')
print(f'Total identidades únicas: {df_maestra.shape[0]}')
print(f'Columnas: {df_maestra.columns.tolist()}')
print(f'\nNulos:')
print(df_maestra.isnull().sum())
```

```python
# Verificar que no haya DNIs duplicados
assert df_maestra['dni_limpio'].dropna().duplicated().sum() == 0, \
    "ERROR: hay DNIs duplicados en la maestra"
print("DNIs únicos: OK")
```

```python
# Verificar que no haya emails duplicados en email_limpio
assert df_maestra['email_limpio'].dropna().duplicated().sum() == 0, \
    "ERROR: hay emails duplicados en la maestra"
print("Emails únicos: OK")
```

```python
# Verificar que no haya registros sin ningún identificador
sin_identificador = df_maestra[
    df_maestra['dni_limpio'].isna() & df_maestra['email_limpio'].isna()
]
print(f'Registros sin DNI ni email: {sin_identificador.shape[0]}')
```

```python
# Muestra del resultado final
df_maestra.sample(10, random_state=42)
```

```python
# Exportar la maestra
df_maestra.to_csv('maestra_identidades.csv', index=False)
print('Maestra exportada.')
```

---

## 8. Cuándo usar cada enfoque

| | Enfoque A — Fuente por fuente | Enfoque B — Concat primero |
|---|---|---|
| **Control** | Total — sabés el origen de cada registro | Menor — los orígenes se mezclan |
| **Trazabilidad** | Alta | Baja |
| **Complejidad del código** | Mayor | Menor |
| **Velocidad de desarrollo** | Más lenta | Más rápida |
| **Recomendado cuando** | Fuentes muy distintas, dataset grande, cliente real | Fuentes similares, exploración, prototipado |
| **Resultado** | Igual en ambos casos si las reglas son las mismas | |

---

> **Regla de oro:** definir las reglas de matching antes de escribir una línea de código.  
> Cambiarlas a mitad del proceso obliga a rehacer todo desde el principio.

---

*Basado en clase de Minería de Datos — IFTS 33 (2026)*
