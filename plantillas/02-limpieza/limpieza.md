# 🧹 Plantilla Limpieza de Datos — Estándar de Industria

> **Variable:** `df_raw` → `df_clean` (entrada sucia, salida limpia)  
> **Uso:** Copiá cada bloque en una celda separada del notebook  
> **Orden:** Seguí las secciones de arriba hacia abajo  
> **Salida:** archivo `_clean.csv` listo para el notebook de EDA

---

## Tabla de Contenidos

1. [Diagnóstico inicial](#1-diagnóstico-inicial)
2. [Duplicados](#2-duplicados)
3. [Valores nulos (NaN)](#3-valores-nulos-nan)
4. [Tipos de datos](#4-tipos-de-datos)
5. [Inconsistencias en texto](#5-inconsistencias-en-texto)
6. [Outliers](#6-outliers)
7. [Reset de índice](#7-reset-de-índice)
8. [Verificación final](#8-verificación-final)
9. [Exportar dataset limpio](#9-exportar-dataset-limpio)
10. [Cuándo usar cada estrategia](#10-cuándo-usar-cada-estrategia)

---

> **Orden fijo de limpieza — nunca cambiar:**
> 1. Diagnóstico inicial ← fotografía del estado original
> 2. Duplicados ← primero, antes de contar nulos
> 3. Nulos ← después de sacar duplicados
> 4. Tipos de datos ← después de nulos (un NaN impide convertir a int)
> 5. Inconsistencias de texto ← después de tipos
> 6. Outliers ← cuando el dataset ya está estructuralmente limpio
> 7. Reset de índice ← siempre al final
> 8. Verificación y exportación

---

## 1. Diagnóstico inicial

**Ejecutar completo antes de tocar cualquier dato. Nunca saltear esta sección.**

```python
# ─── Primeras y últimas filas ──────────────────────────────────────────────
print('\n--- Primeras 5 filas ---')
display(df_raw.head())

print('\n--- Últimas 5 filas ---')
display(df_raw.tail())

# ─── Dimensiones ───────────────────────────────────────────────────────────
print(f'Filas: {df_raw.shape[0]} | Columnas: {df_raw.shape[1]}')

# ─── Tipos de datos ────────────────────────────────────────────────────────
print('\n--- Info general ---')
df_raw.info()

# ─── Valores nulos por columna ─────────────────────────────────────────────
print('\n--- Nulos por columna ---')
# Verificamos si hay algún nulo en toda la tabla
if df_raw.isnull().sum().sum() == 0:
    print('✅ Sin valores nulos')
else:
    # Sino mostramos tabla filtrada de nulos
    nulos = pd.DataFrame({
        'Nulos': df_raw.isnull().sum(),
        'Porcentaje': (df_raw.isnull().sum() / len(df_raw) * 100).round(2)
    })
    display(nulos[nulos['Nulos'] > 0])

# ─── Duplicados ────────────────────────────────────────────────────────────
print(f'\nFilas duplicadas: {df_raw.duplicated().sum()}')

# ─── Estadísticas descriptivas ─────────────────────────────────────────────
print('\n--- Estadísticas descriptivas ---')
display(df_raw.describe())
```

---

## 2. Duplicados

**Hacer siempre. Antes de cualquier otra limpieza.**

```python
# Ver cuántos hay
print(f'Duplicados antes: {df_raw.duplicated().sum()}')

# Ver las filas duplicadas (opcional)
display(df_raw[df_raw.duplicated(keep=False)].head(10))
```

```python
# Eliminar duplicados
df_raw = df_raw.drop_duplicates()

# Verificar
print(f'Duplicados después: {df_raw.duplicated().sum()}')
print(f'Filas restantes: {df_raw.shape[0]}')
```

> **Variantes:**
> ```python
> # Duplicados por columnas específicas (no fila completa)
> df_raw = df_raw.drop_duplicates(subset=['COL1', 'COL2'])  # ← reemplazar
>
> # Mantener última ocurrencia en lugar de primera
> df_raw = df_raw.drop_duplicates(keep='last')
> ```

---

## 3. Valores nulos (NaN)

**La estrategia depende del porcentaje de nulos y del tipo de columna.**

```python
# Ver estado actual de nulos
nulos = pd.DataFrame({
    'Nulos': df_raw.isnull().sum(),
    'Porcentaje': (df_raw.isnull().sum() / len(df_raw) * 100).round(2),
    'Tipo': df_raw.dtypes
})
display(nulos[nulos['Nulos'] > 0].sort_values('Porcentaje', ascending=False))
```

```python
# ── Imputar con MEDIANA ────────────────────────────────────────────────────
# Cuándo: columnas numéricas con outliers (precios, salarios, satisfacción)
mediana = df_raw['COLUMNA'].median()                          # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna(mediana)
print(f'Mediana usada: {mediana}')
```

```python
# ── Imputar con MEDIA ──────────────────────────────────────────────────────
# Cuándo: columnas numéricas sin outliers (temperatura, peso, edad)
media = df_raw['COLUMNA'].mean()                              # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna(media)
print(f'Media usada: {media:.2f}')
```

```python
# ── Imputar con MODA ───────────────────────────────────────────────────────
# Cuándo: columnas categóricas (ciudad, categoría, estado)
moda = df_raw['COLUMNA'].mode()[0]                            # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna(moda)
print(f'Moda usada: {moda}')
```

```python
# ── Imputar con valor fijo ─────────────────────────────────────────────────
# Cuándo: cuando "sin valor" tiene un significado específico
df_raw['COLUMNA'] = df_raw['COLUMNA'].fillna('Desconocido')   # ← reemplazar
df_raw['COLUMNA_NUM'] = df_raw['COLUMNA_NUM'].fillna(0)       # ← reemplazar
```

```python
# ── Eliminar filas con nulos ───────────────────────────────────────────────
# Cuándo: menos del 5% de nulos en una columna crítica
df_raw = df_raw.dropna(subset=['COLUMNA_CRITICA'])            # ← reemplazar
```

```python
# ── Eliminar columna entera ────────────────────────────────────────────────
# Cuándo: más del 50% de nulos y la columna no aporta valor
df_raw = df_raw.drop(columns=['COLUMNA_INUTILIZABLE'])        # ← reemplazar
```

```python
# Verificar que no quedan nulos
print(df_raw.isnull().sum())
```

---

## 4. Tipos de datos

**Hacer después de limpiar nulos. Un nulo en una columna impide convertir a int.**

```python
# Ver tipos actuales
print(df_raw.dtypes)
```

```python
# ── Fechas ────────────────────────────────────────────────────────────────
# Cuándo: columnas que son fechas pero llegaron como texto (object)
df_raw['COLUMNA_FECHA'] = pd.to_datetime(df_raw['COLUMNA_FECHA'])              # ← reemplazar
df_raw['COLUMNA_FECHA'] = pd.to_datetime(df_raw['COLUMNA_FECHA'],
                                          format='%d/%m/%Y')   # formato específico
df_raw['COLUMNA_FECHA'] = pd.to_datetime(df_raw['COLUMNA_FECHA'],
                                          errors='coerce')     # NaN si no puede parsear
```

```python
# ── Numéricos ────────────────────────────────────────────────────────────
df_raw['COLUMNA'] = pd.to_numeric(df_raw['COLUMNA'], errors='coerce')  # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].astype(int)
df_raw['COLUMNA'] = df_raw['COLUMNA'].astype(float)
```

```python
# ── Numérico con formato texto ($, puntos, comas) ────────────────────────
# Cuándo: columna tiene '$1.234,56' en lugar de 1234.56
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.replace('$', '', regex=False)  # ← reemplazar
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.replace('.', '', regex=False)
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.replace(',', '.', regex=False)
df_raw['COLUMNA'] = pd.to_numeric(df_raw['COLUMNA'], errors='coerce')
```

```python
# ── Categórico ───────────────────────────────────────────────────────────
# Cuándo: columna tiene pocos valores únicos repetidos (ahorra memoria)
df_raw['COLUMNA'] = df_raw['COLUMNA'].astype('category')                # ← reemplazar

# Categórico con orden lógico (Baja < Media < Alta)
df_raw['COLUMNA'] = pd.Categorical(
    df_raw['COLUMNA'],
    categories=['Baja', 'Media', 'Alta'],                               # ← reemplazar
    ordered=True
)
```

```python
# Verificar tipos después de convertir
print(df_raw.dtypes)
```
---

## 4.1 Normalizar fechas con formato mixto (previo a `pd.to_datetime`)

**Hacer antes del bloque "Fechas" de la sección 4. Síntoma: la misma columna mezcla formatos (`'1/15/06'`, `'11-02/05'`, `'02-01/06'`) y `pd.to_datetime()` falla o produce fechas incorrectas.**

```python
# ── Diagnóstico: ver todos los patrones de formato presentes ─────────────
import re

def detectar_patrones_fecha(df, columna):
    patrones = df[columna].astype(str).apply(
        lambda x: re.sub(r'\d+', '#', x)  # reemplaza números por # para ver el patrón
    )
    print(patrones.value_counts())

detectar_patrones_fecha(df_raw, 'COLUMNA_FECHA')                         # ← reemplazar
```

```python
# ── Probar conversión con múltiples formatos candidatos ───────────────────
formatos_candidatos = ['%m/%d/%y', '%d-%m/%y', '%m-%d/%y']               # ← ajustar según diagnóstico

def parsear_fecha_multiformato(valor, formatos):
    for fmt in formatos:
        try:
            return pd.to_datetime(valor, format=fmt)
        except (ValueError, TypeError):
            continue
    return pd.NaT  # si ningún formato funcionó

df_raw['COLUMNA_FECHA'] = df_raw['COLUMNA_FECHA'].apply(
    lambda x: parsear_fecha_multiformato(x, formatos_candidatos)
)
```

```python
# Verificar cuántas fechas no se pudieron parsear
no_parseadas = df_raw['COLUMNA_FECHA'].isnull().sum()                    # ← reemplazar
print(f'Fechas sin parsear: {no_parseadas} ({no_parseadas/len(df_raw)*100:.1f}%)')
display(df_raw[df_raw['COLUMNA_FECHA'].isnull()])
```

> Si quedan muchas sin parsear, revisar manualmente esos casos — puede haber un tercer formato no contemplado.
---

## 5. Inconsistencias en texto

**Hacer en columnas categóricas antes de análisis o agrupaciones.**

```python
# ── Espacios extra ────────────────────────────────────────────────────────
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.strip()                       # ← reemplazar

# ── Normalizar mayúsculas ────────────────────────────────────────────────
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.title()   # Primera Letra Mayúscula
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.upper()   # TODO MAYÚSCULAS
df_raw['COLUMNA'] = df_raw['COLUMNA'].str.lower()   # todo minúsculas
```

```python
# ── Reemplazar valores mal escritos ─────────────────────────────────────
reemplazos = {
    'VALOR_MAL': 'VALOR_CORRECTO',    # ← reemplazar
    'VALOR_MAL_2': 'VALOR_CORRECTO',
}
df_raw['COLUMNA'] = df_raw['COLUMNA'].replace(reemplazos)               # ← reemplazar
```

```python
# ── Verificar resultado ──────────────────────────────────────────────────
print(df_raw['COLUMNA'].value_counts())                                 # ← reemplazar
```
---

## 5.1 Encoding roto (mojibake)

**Hacer en la sección 5, después de "Espacios extra". Síntoma: letras como `ñ`, `é`, `á` aparecen como `�`, `Ã±`, o se pierden directamente (`'Coruaa'` en vez de `'Coruña'`).**

```python
# ── Detectar encoding sospechoso en UNA columna ───────────────────────────
import re

def detectar_encoding_roto(df, columna):
    patron = re.compile(r'[ï¿½�]|[A-Za-z]{2}aa[A-Za-z]?\b')
    sospechosos = df[df[columna].astype(str).str.contains(patron, na=False, regex=True)]
    return sospechosos
```

```python
# ── Recorrer TODAS las columnas de texto (no adivinar a mano) ────────────
columnas_texto = df_raw.select_dtypes(include='object').columns

for col in columnas_texto:
    resultado = detectar_encoding_roto(df_raw, col)
    if len(resultado) > 0:
        print(f'⚠️  {col}: {len(resultado)} valores sospechosos')
        display(resultado[[col]].drop_duplicates())
```

```python
# ── Opción A: Re-leer el CSV con el encoding correcto ────────────────────
# Cuándo: el archivo completo está mal — la solución de raíz
df_raw = pd.read_csv('archivo.csv', encoding='latin-1')                 # ← reemplazar
# Probar también: 'utf-8', 'cp1252', 'iso-8859-1'
```

```python
# ── Opción B: Reemplazo manual puntual ────────────────────────────────────
# Cuándo: son pocos valores y ya identificaste cuáles con el loop anterior
reemplazos_encoding = {
    'Coruaa': 'Coruña',                                                  # ← reemplazar
    'Troph�e des Champions': 'Trophée des Champions',
}
df_raw['COLUMNA'] = df_raw['COLUMNA'].replace(reemplazos_encoding)       # ← reemplazar
```

```python
# Verificar que no quedan residuos — repetir el loop completo
for col in columnas_texto:
    resultado = detectar_encoding_roto(df_raw, col)
    if len(resultado) > 0:
        print(f'⚠️  {col}: todavía quedan {len(resultado)} valores')
```
---

## 5.2 Caracteres invisibles (tabs, saltos de línea ocultos)

**Hacer junto con "Espacios extra". `str.strip()` no siempre los detecta — usar regex para limpiar cualquier whitespace no estándar.**

```python
# ── Detectar valores con solo caracteres invisibles ──────────────────────
sospechosos = df_raw[df_raw['COLUMNA'].astype(str).str.fullmatch(r'\s+')]  # ← reemplazar
print(f'Valores que son solo espacios/tabs: {len(sospechosos)}')
display(sospechosos)
```

```python
# ── Limpiar todo whitespace no estándar (tabs, \n, espacios dobles) ──────
df_raw['COLUMNA'] = (
    df_raw['COLUMNA']
    .astype(str)
    .str.replace(r'\s+', ' ', regex=True)   # colapsa tabs/saltos a un espacio
    .str.strip()
    .replace('', np.nan)                     # si quedó vacío, convertir a NaN
)
```
---
## 5.3 Separar columnas combinadas

**Hacer cuando una columna contiene dos datos distintos fusionados en el mismo campo.**
**Síntoma: `'Sebastián Meza Portero'` en vez de nombre y posición por separado.**

### Caso A — Separar texto con patrón conocido al final del string (nombre + posición)

```python
# ── Definir los valores posibles de la parte a extraer ───────────────────
# Listar de más largo a más corto para que el regex priorice el match completo
valores_a_extraer = [
    'Mediocentro ofensivo',     # ← reemplazar con los valores del dataset
    'Lateral izquierdo',
    'Lateral derecho',
    'Interior derecho',
    'Interior izquierdo',
    'Extremo izquierdo',
    'Extremo derecho',
    'Delantero centro',
    'Defensa central',
    'Mediocentro',
    'Portero',
    'Pivote',
]

patron = '(' + '|'.join(valores_a_extraer) + ')'

# ── Extraer en columna nueva ──────────────────────────────────────────────
df_raw['posicion'] = df_raw['Jugadores'].str.extract(patron)             # ← reemplazar nombre columna origen

# ── Limpiar la columna original dejando solo el nombre ───────────────────
df_raw['jugador'] = df_raw['Jugadores'].str.replace(patron, '', regex=True).str.strip()

# ── Verificar que no quedaron NaN en posicion ────────────────────────────
print(f"NaN en posicion: {df_raw['posicion'].isnull().sum()}")
display(df_raw[['jugador', 'posicion']].head(10))

# ── Eliminar columna original una vez verificado ─────────────────────────
df_raw = df_raw.drop(columns=['Jugadores'])                              # ← reemplazar
```

### Caso B — Separar texto con delimitador fijo (fecha + edad entre paréntesis)

```python
# ── Separar en dos columnas usando regex con grupos ──────────────────────
# Ejemplo: '14/03/2000 (26)' → fecha_nacimiento='14/03/2000', edad=26
df_raw['fecha_nacimiento'] = df_raw['F. Nacim./Edad'].str.extract(r'(\d{2}/\d{2}/\d{4})')   # ← reemplazar
df_raw['edad'] = df_raw['F. Nacim./Edad'].str.extract(r'\((\d+)\)').astype(float)            # ← reemplazar

# ── Convertir fecha a tipo datetime ──────────────────────────────────────
df_raw['fecha_nacimiento'] = pd.to_datetime(df_raw['fecha_nacimiento'],
                                             format='%d/%m/%Y',
                                             errors='coerce')

# ── Verificar ────────────────────────────────────────────────────────────
print(f"NaN en fecha_nacimiento: {df_raw['fecha_nacimiento'].isnull().sum()}")
print(f"NaN en edad: {df_raw['edad'].isnull().sum()}")
display(df_raw[['F. Nacim./Edad', 'fecha_nacimiento', 'edad']].head(5))

# ── Eliminar columna original una vez verificado ─────────────────────────
df_raw = df_raw.drop(columns=['F. Nacim./Edad'])                         # ← reemplazar
```

> **Regla:** siempre verificar NaN en las columnas nuevas antes de eliminar la original.
> Si quedan NaN inesperados, hay un patrón no contemplado en el regex — revisar con `df_raw[df_raw['columna_nueva'].isnull()]`

---

## 6. Outliers

**Detectar siempre. Tratar según contexto de negocio.**

```python
# Detectar con IQR
def detectar_outliers(df, columna):
    Q1 = df[columna].quantile(0.25)
    Q3 = df[columna].quantile(0.75)
    IQR = Q3 - Q1
    lim_inf = Q1 - 1.5 * IQR
    lim_sup = Q3 + 1.5 * IQR
    outliers = df[(df[columna] < lim_inf) | (df[columna] > lim_sup)]
    print(f'{columna}:')
    print(f'  Rango normal: [{lim_inf:.2f}, {lim_sup:.2f}]')
    print(f'  Outliers: {len(outliers)} ({len(outliers)/len(df)*100:.1f}%)\n')
    return lim_inf, lim_sup

lim_inf, lim_sup = detectar_outliers(df_raw, 'COLUMNA_NUMERICA')       # ← reemplazar
```

```python
# ── Opción A: Eliminar filas con outliers ────────────────────────────────
# Cuándo: son errores de carga (precio = 0, edad = 999)
df_raw = df_raw[
    (df_raw['COLUMNA'] >= lim_inf) &                                    # ← reemplazar
    (df_raw['COLUMNA'] <= lim_sup)
]
```

```python
# ── Opción B: Capping (reemplazar por el límite) ─────────────────────────
# Cuándo: los valores extremos son reales pero distorsionan el análisis
df_raw['COLUMNA'] = df_raw['COLUMNA'].clip(                             # ← reemplazar
    lower=lim_inf,
    upper=lim_sup
)
```

```python
# ── Opción C: Dejar los outliers ─────────────────────────────────────────
# Cuándo: son casos reales con sentido de negocio
# (una venta muy grande es real, no un error)
# → Documentar la decisión en celda de texto
```

> **Regla para decidir:**
> - El valor es físicamente imposible (edad = -5, precio = 0) → **eliminar**
> - El valor es extremo pero posible (venta de $10M en retail) → **dejar o hacer capping**
> - No sabés → **dejar y documentar**, el EDA lo va a revelar

---

## 7. Reset de índice

**Hacer siempre al final, después de todas las eliminaciones.**

```python
# Ver si el índice tiene huecos
print(f'Índice antes del reset: {df_raw.index.tolist()[:10]}...')

# Resetear
df_raw = df_raw.reset_index(drop=True)

# Verificar
print(f'Índice después del reset: {df_raw.index.tolist()[:10]}...')
print(f'Filas finales: {df_raw.shape[0]}')
```

> `drop=True` descarta el índice viejo. Sin `drop=True`, el índice viejo se convierte en una columna nueva — casi nunca es lo que querés.

---

## 8. Verificación final

**Ejecutar completo antes de exportar. Confirmar que el dataset está limpio.**

```python
print('=' * 50)
print('REPORTE FINAL DE CALIDAD')
print('=' * 50)
print(f'Filas finales:      {df_raw.shape[0]}')
print(f'Columnas:           {df_raw.shape[1]}')
print(f'Duplicados:         {df_raw.duplicated().sum()}')
print(f'Nulos totales:      {df_raw.isnull().sum().sum()}')
print('=' * 50)
```

```python
# Tipos de datos finales
print(df_raw.dtypes)
```

```python
# Vista previa del dataset limpio
df_raw.head(10)
```

---

## 9. Exportar dataset limpio

**Último paso. El archivo exportado es la entrada del notebook de EDA.**

```python
# Renombrar a df_clean antes de exportar
df_clean = df_raw.copy()

# Exportar
nombre_salida = 'NOMBRE_DATASET_limpio.csv'                             # ← reemplazar
df_clean.to_csv(nombre_salida, index=False, encoding='utf-8')
print(f'Dataset exportado: {nombre_salida}')
print(f'Filas: {df_clean.shape[0]} | Columnas: {df_clean.shape[1]}')
```

```python
# Verificar que el archivo se guardó correctamente
verificacion = pd.read_csv(nombre_salida)
print(f'Verificación: {verificacion.shape[0]} filas x {verificacion.shape[1]} columnas')
```

```python
# Solo en Colab — descargar el archivo a tu PC
# from google.colab import files
# files.download(nombre_salida)
```

---

## 10. Cuándo usar cada estrategia

### Nulos

| % de nulos | Tipo de columna | Estrategia |
|-----------|-----------------|-----------|
| < 5% | Numérica o categórica | Eliminar las filas |
| 5% - 30% | Numérica con outliers | Imputar con mediana |
| 5% - 30% | Numérica sin outliers | Imputar con media |
| 5% - 30% | Categórica | Imputar con moda |
| > 50% | Cualquiera | Evaluar eliminar la columna |

### Outliers

| Situación | Estrategia |
|-----------|-----------|
| Valor imposible (precio = 0, edad negativa) | Eliminar fila |
| Valor extremo pero posible | Capping o dejar |
| No sabés si es error o caso real | Dejar y documentar |

### Tipos de datos

| La columna tiene... | Convertir a... |
|--------------------|----------------|
| Fechas como texto | `pd.to_datetime()` |
| Números como texto | `pd.to_numeric()` |
| Números con `$` o `.` | Limpiar string → `pd.to_numeric()` |
| Texto con pocos valores únicos | `astype('category')` |
| Texto con orden lógico | `pd.Categorical(..., ordered=True)` |

---

*Para el análisis exploratorio del dataset limpio → `plantilla_eda.md`*
