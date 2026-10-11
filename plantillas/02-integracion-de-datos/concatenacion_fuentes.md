# 🔗 Concatenación de Fuentes de Datos

> **Variable de salida:** `df_total` (DataFrame unificado con todas las fuentes)  
> **Uso:** Copiá el caso que corresponda según tu situación  
> **Prerequisito:** tener las fuentes cargadas como DataFrames o disponibles como archivos

---

## Tabla de Contenidos

1. [Caso A — Concatenación directa desde archivos](#caso-a--concatenación-directa-desde-archivos)
2. [Caso B — Lista dinámica desde una carpeta](#caso-b--lista-dinámica-desde-una-carpeta)
3. [Caso C — Con selección y renombre de columnas](#caso-c--con-selección-y-renombre-de-columnas)
4. [Caso D — Con selección, renombre y limpieza](#caso-d--con-selección-renombre-y-limpieza)
5. [Caso E — Agregar columna de origen](#caso-e--agregar-columna-de-origen)
6. [Verificación post-concat](#verificación-post-concat)
7. [Tabla de decisión rápida](#tabla-de-decisión-rápida)

---

## Caso A — Concatenación directa desde archivos

**Cuándo:** las fuentes ya tienen los mismos nombres de columnas y los datos están limpios.

```python
df_total = pd.concat([
    pd.read_csv('ARCHIVO_1.csv'),   # ← reemplazar
    pd.read_csv('ARCHIVO_2.csv'),
    pd.read_csv('ARCHIVO_3.csv'),
], ignore_index=True)

print(f'Total combinado: {df_total.shape[0]} filas x {df_total.shape[1]} columnas')
```

---

## Caso B — Lista dinámica desde una carpeta

**Cuándo:** tenés N archivos en una carpeta y querés concatenarlos todos sin listarlos uno a uno.

```python
import glob

archivos = glob.glob('CARPETA/*.csv')          # ← reemplazar ruta y extensión
lista_dfs = [pd.read_csv(f) for f in archivos]

df_total = pd.concat(lista_dfs, ignore_index=True)
print(f'Archivos concatenados: {len(lista_dfs)}')
print(f'Total filas: {df_total.shape[0]}')
```

---

## Caso C — Con selección y renombre de columnas

**Cuándo:** cada fuente tiene nombres de columnas diferentes para el mismo dato.  
**Importante:** hacer el renombre ANTES del concat para que pandas alinee bien las columnas.

```python
fuente_1_xa_concat = fuente_1[['ID_COL', 'COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']].copy()  # ← reemplazar
fuente_1_xa_concat.columns = ['id', 'nombre', 'dni', 'email']

fuente_2_xa_concat = fuente_2[['COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']].copy()             # ← reemplazar
fuente_2_xa_concat.columns = ['nombre', 'dni', 'email']

fuente_3_xa_concat = fuente_3[['COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']].copy()             # ← reemplazar
fuente_3_xa_concat.columns = ['nombre', 'dni', 'email']

fuente_4_xa_concat = fuente_4[['COL_NOMBRE', 'COL_EMAIL']].copy()                        # ← reemplazar (sin DNI)
fuente_4_xa_concat.columns = ['nombre', 'email']

df_total = pd.concat([
    fuente_1_xa_concat,
    fuente_2_xa_concat,
    fuente_3_xa_concat,
    fuente_4_xa_concat,
], ignore_index=True)

print(f'Total combinado: {df_total.shape[0]} filas')
display(df_total.head())
```

---

## Caso D — Con selección, renombre y limpieza

**Cuándo:** querés normalizar los datos de cada fuente antes de unirlas.  
Es el más completo — combina el Caso C con las funciones de normalización.  
Requiere tener definidas `limpiar_nombre()`, `limpiar_dni()` y `limpiar_email()`.

```python
fuente_1_xa_concat = fuente_1[['ID_COL', 'COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']].copy()  # ← reemplazar
fuente_1_xa_concat.columns = ['id', 'nombre', 'dni', 'email']
fuente_1_xa_concat['nombre'] = fuente_1_xa_concat['nombre'].apply(limpiar_nombre)
fuente_1_xa_concat['dni']    = fuente_1_xa_concat['dni'].apply(limpiar_dni)
fuente_1_xa_concat['email']  = fuente_1_xa_concat['email'].apply(limpiar_email)

fuente_2_xa_concat = fuente_2[['COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']].copy()             # ← reemplazar
fuente_2_xa_concat.columns = ['nombre', 'dni', 'email']
fuente_2_xa_concat['nombre'] = fuente_2_xa_concat['nombre'].apply(limpiar_nombre)
fuente_2_xa_concat['dni']    = fuente_2_xa_concat['dni'].apply(limpiar_dni)
fuente_2_xa_concat['email']  = fuente_2_xa_concat['email'].apply(limpiar_email)

fuente_3_xa_concat = fuente_3[['COL_NOMBRE', 'COL_DNI', 'COL_EMAIL']].copy()             # ← reemplazar
fuente_3_xa_concat.columns = ['nombre', 'dni', 'email']
fuente_3_xa_concat['nombre'] = fuente_3_xa_concat['nombre'].apply(limpiar_nombre)
fuente_3_xa_concat['dni']    = fuente_3_xa_concat['dni'].apply(limpiar_dni)
fuente_3_xa_concat['email']  = fuente_3_xa_concat['email'].apply(limpiar_email)

fuente_4_xa_concat = fuente_4[['COL_NOMBRE', 'COL_EMAIL']].copy()                        # ← reemplazar (sin DNI)
fuente_4_xa_concat.columns = ['nombre', 'email']
fuente_4_xa_concat['nombre'] = fuente_4_xa_concat['nombre'].apply(limpiar_nombre)
fuente_4_xa_concat['email']  = fuente_4_xa_concat['email'].apply(limpiar_email)

df_total = pd.concat([
    fuente_1_xa_concat,
    fuente_2_xa_concat,
    fuente_3_xa_concat,
    fuente_4_xa_concat,
], ignore_index=True)

print(f'Total combinado: {df_total.shape[0]} filas')
```

---

## Caso E — Agregar columna de origen

**Cuándo:** necesitás trazabilidad — saber después de concatenar de qué fuente vino cada registro.  
Muy útil para auditoría y para el enfoque de integración fuente por fuente.  
Se puede combinar con cualquiera de los casos anteriores — agregar estas líneas antes del concat.

```python
fuente_1_xa_concat['origen'] = 'fuente_1'     # ← reemplazar con nombre descriptivo
fuente_2_xa_concat['origen'] = 'fuente_2'
fuente_3_xa_concat['origen'] = 'fuente_3'
fuente_4_xa_concat['origen'] = 'fuente_4'

df_total = pd.concat([
    fuente_1_xa_concat,
    fuente_2_xa_concat,
    fuente_3_xa_concat,
    fuente_4_xa_concat,
], ignore_index=True)

# Verificar distribución por origen
print(df_total['origen'].value_counts())
```

---

## Verificación post-concat

```python
# Ejecutar siempre después de cualquier concatenación
print(f'Shape final: {df_total.shape}')
print(f'\nColumnas: {df_total.columns.tolist()}')
print(f'\nNulos por columna:\n{df_total.isnull().sum()}')
print(f'\nPrimeras filas:')
display(df_total.head())
```

---

## Tabla de decisión rápida

| Situación | Caso |
|---|---|
| Mismos nombres de columna, datos limpios | A — directo desde archivo |
| N archivos en una carpeta | B — con glob |
| Columnas con nombres distintos entre fuentes | C — con renombre |
| Columnas distintas + datos sucios | D — con renombre y limpieza |
| Necesitás saber de qué fuente vino cada fila | E — agregar columna `origen` |

> `ignore_index=True` siempre — reinicia el índice de 0 a N en el resultado final.  
> Sin él los índices originales de cada fuente se conservan y pueden repetirse.

---

*Para el proceso completo de integración fuente por fuente → `consolidacion_identidades.md`*
