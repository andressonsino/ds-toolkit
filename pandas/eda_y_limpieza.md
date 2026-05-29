# 🔍 Guía Completa de EDA y Limpieza de Datos en Pandas

> **Nivel:** Básico → Avanzado  
> **Objetivo:** Análisis exploratorio y limpieza profesional de datasets  
> **Entorno:** Jupyter Lab / Google Colab

---

## Tabla de Contenidos

1. [Setup inicial y supresión de warnings](#1-setup-inicial-y-supresión-de-warnings)
2. [Carga del dataset](#2-carga-del-dataset)
3. [Diagnóstico inicial](#3-diagnóstico-inicial)
4. [Duplicados](#4-duplicados)
5. [Valores nulos (NaN)](#5-valores-nulos-nan)
6. [Tipos de datos y conversiones](#6-tipos-de-datos-y-conversiones)
7. [Reset de índice](#7-reset-de-índice)
8. [Valores atípicos (Outliers)](#8-valores-atípicos-outliers)
9. [Inconsistencias en texto](#9-inconsistencias-en-texto)
10. [Renombrar columnas](#10-renombrar-columnas)
11. [Creación de columnas derivadas](#11-creación-de-columnas-derivadas)
12. [Análisis estadístico](#12-análisis-estadístico)
13. [Visualizaciones de EDA](#13-visualizaciones-de-eda)
14. [Exportar dataset limpio](#14-exportar-dataset-limpio)
15. [Checklist profesional](#15-checklist-profesional)

---

## Orden recomendado del flujo completo

```
1. Setup (imports + warnings)
2. Carga del dataset
3. Diagnóstico inicial (shape, info, describe, nulos, duplicados)
4. Eliminar duplicados
5. Tratar nulos (imputar o eliminar)
6. Convertir tipos de datos
7. Reset de índice
8. Tratar outliers
9. Limpiar inconsistencias de texto
10. Renombrar columnas
11. Crear columnas derivadas
12. Análisis estadístico (groupby, correlación)
13. Visualizaciones
14. Exportar dataset limpio
```

---

## 1. Setup inicial y supresión de warnings

Siempre arrancá el notebook con esta celda. Suprime los warnings de librerías que no son errores reales y configuran el entorno de trabajo.

```python
# ─── Imports ───────────────────────────────────────────────────────────────
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# ─── Supresión de warnings ─────────────────────────────────────────────────
import warnings
warnings.filterwarnings('ignore')          # suprime todos los warnings
pd.options.mode.chained_assignment = None  # suprime SettingWithCopyWarning

# ─── Configuración de visualización ────────────────────────────────────────
pd.set_option('display.max_columns', None)   # muestra todas las columnas
pd.set_option('display.max_rows', 100)       # muestra hasta 100 filas
pd.set_option('display.float_format', '{:.2f}'.format)  # 2 decimales
plt.style.use('seaborn-v0_8')               # estilo de gráficos profesional
sns.set_palette('husl')                      # paleta de colores
```

> 💡 **¿Cuándo NO suprimir warnings?**  
> Durante desarrollo y debugging. Una vez que el código está probado y funciona, los suprimís para presentaciones o entregas.

> 💡 **¿Instalo librerías en Miniconda?**  
> No — si ya tenés el entorno `datasci` con todo instalado, no necesitás `pip install` al inicio del notebook. Ese comando solo es necesario en Colab (que no tiene nada instalado por defecto).

---

## 2. Carga del dataset

### 2.1 Desde archivo local (Jupyter Lab)

```python
df = pd.read_csv('ventas.csv')

# Con opciones adicionales
df = pd.read_csv(
    'ventas.csv',
    sep=',',              # separador (puede ser ';' en archivos europeos)
    encoding='utf-8',     # encoding (probar 'latin-1' si falla)
    parse_dates=['Fecha'] # parsear columnas de fecha directamente
)
```

### 2.2 Desde Colab (upload manual)

```python
from google.colab import files
df = pd.read_csv(list(files.upload().keys())[0])
```

### 2.3 Desde Google Drive (Colab)

```python
from google.colab import drive
drive.mount('/content/drive')
df = pd.read_csv('/content/drive/MyDrive/ventas.csv')
```

### 2.4 Verificar carga exitosa

```python
print(f'Dataset cargado: {df.shape[0]} filas x {df.shape[1]} columnas')
df.head()
```

---

## 3. Diagnóstico inicial

Esta es la **primera celda que ejecutás** después de cargar. Te da el panorama completo del dataset.

```python

# ─── Primeras y últimas filas ──────────────────────────────────────────────
print('\n--- Primeras 5 filas ---')
display(df.head())

print('\n--- Últimas 5 filas ---')
display(df.tail())

# ─── Dimensiones ───────────────────────────────────────────────────────────
print(f'Filas: {df.shape[0]} | Columnas: {df.shape[1]}')

# ─── Tipos de datos y nulos ────────────────────────────────────────────────
print('\n--- Info general ---')
df.info()

# ─── Valores nulos por columna ─────────────────────────────────────────────
print('\n--- Nulos por columna ---')
nulos = pd.DataFrame({
    'Nulos': df.isnull().sum(),
    'Porcentaje': (df.isnull().sum() / len(df) * 100).round(2)
})
display(nulos[nulos['Nulos'] > 0])

# ─── Duplicados ────────────────────────────────────────────────────────────
print(f'\nFilas duplicadas: {df.duplicated().sum()}')

# ─── Estadísticas descriptivas ─────────────────────────────────────────────
print('\n--- Estadísticas descriptivas ---')
display(df.describe())

```
### 3.1 Backup de columnas sensibles (opcional pero recomendado)

```python
# Copia de seguridad - backup antes de transformar fechas
df['fecha_venta_original'] = df['fecha_venta'].copy()

# Backup antes de imputar nulos numéricos
df['importe_original'] = df['importe'].copy()

```
---

## 4. Duplicados

### 4.1 Detectar duplicados

```python
# Cantidad total
print(f'Duplicados: {df.duplicated().sum()}')

# Ver las filas duplicadas
display(df[df.duplicated(keep=False)])  # muestra todas las copias

# Duplicados por columnas específicas (no fila completa)
print(df.duplicated(subset=['Vendedor', 'Fecha']).sum())
```

### 4.2 Eliminar duplicados

```python
# Eliminar y guardar en el mismo DataFrame
df = df.drop_duplicates()

# Mantener primera o última ocurrencia
df = df.drop_duplicates(keep='first')  # default
df = df.drop_duplicates(keep='last')

# Eliminar por columnas específicas
df = df.drop_duplicates(subset=['Vendedor', 'Fecha'])

# Verificar
print(f'Duplicados restantes: {df.duplicated().sum()}')
```

> ⚠️ **Siempre asigná el resultado** `df = df.drop_duplicates()`. Sin la asignación, pandas muestra el resultado pero no modifica el DataFrame original.

---

## 5. Valores nulos (NaN)

### 5.1 Diagnóstico de nulos

```python
# Mapa completo de nulos
nulos = pd.DataFrame({
    'Nulos': df.isnull().sum(),
    'Porcentaje': (df.isnull().sum() / len(df) * 100).round(2),
    'Tipo': df.dtypes
})
display(nulos[nulos['Nulos'] > 0].sort_values('Porcentaje', ascending=False))
```

### 5.2 Estrategias de imputación

La estrategia depende del porcentaje de nulos y del tipo de columna:

| % de nulos | Estrategia recomendada |
|-----------|----------------------|
| < 5% | Eliminar las filas |
| 5% - 30% | Imputar con media, mediana o moda |
| > 30% | Evaluar si la columna sirve; posible eliminación |

```python
# ─── Imputar con MEDIANA (columnas numéricas con outliers) ──────────────────
# Usar cuando hay valores extremos que distorsionan el promedio
# Ej: salarios, satisfacción, precios
mediana = df['Satisfaccion_Cliente'].median()
df['Satisfaccion_Cliente'] = df['Satisfaccion_Cliente'].fillna(mediana)

# ─── Imputar con MEDIA (columnas numéricas sin outliers) ───────────────────
# Usar cuando la distribución es simétrica y sin extremos
# Ej: temperatura, peso, altura
media = df['Temperatura'].mean()
df['Temperatura'] = df['Temperatura'].fillna(media)

# ─── Imputar con MODA (columnas categóricas) ───────────────────────────────
# Usar para texto o categorías — el valor más frecuente
moda = df['Ciudad'].mode()[0]
df['Ciudad'] = df['Ciudad'].fillna(moda)

# ─── Imputar con un valor fijo ─────────────────────────────────────────────
df['Estado'] = df['Estado'].fillna('Desconocido')
df['Cantidad'] = df['Cantidad'].fillna(0)

# ─── Eliminar filas con nulos ──────────────────────────────────────────────
# Cuando el % es bajo y no afecta el análisis
df = df.dropna(subset=['Columna_Critica'])

# Eliminar filas con cualquier nulo
df = df.dropna()

# ─── Eliminar columna entera ───────────────────────────────────────────────
# Cuando tiene demasiados nulos y no aporta valor
df = df.drop(columns=['Columna_Con_Muchos_Nulos'])

# ─── Verificar que no quedan nulos ─────────────────────────────────────────
print(df.isnull().sum())
```

> 💡 **Media vs Mediana:**  
> - **Media** = promedio matemático. Se ve afectada por valores extremos.  
> - **Mediana** = valor central. Resistente a extremos.  
> Ejemplo: salarios [1000, 1000, 1000, 100000] → Media: 25750 (engañosa) | Mediana: 1000 (real)

---

## 6. Tipos de datos y conversiones

### 6.1 Cuándo convertir tipos

Hacé las conversiones **después de limpiar nulos y duplicados**, antes de hacer análisis o visualizaciones. Un tipo de dato incorrecto genera errores en operaciones matemáticas y agrupaciones.

```python
# Ver tipos actuales
print(df.dtypes)
```

### 6.2 Conversiones comunes

```python
# ─── Fechas (muy común en datasets reales) ─────────────────────────────────
# Si no se parsearon al cargar el CSV
df['Fecha'] = pd.to_datetime(df['Fecha'])
df['Fecha_Venta'] = pd.to_datetime(df['Fecha_Venta'], format='%Y/%m/%d')  # formato específico
df['Fecha_Venta'] = pd.to_datetime(df['Fecha_Venta'], errors='coerce')    # NaN si no puede parsear

# Extraer partes de una fecha
df['Año'] = df['Fecha'].dt.year
df['Mes'] = df['Fecha'].dt.month
df['Dia_Semana'] = df['Fecha'].dt.day_name()

# ─── Numéricos ──────────────────────────────────────────────────────────────
df['Importe'] = pd.to_numeric(df['Importe'], errors='coerce')  # NaN si falla
df['Edad'] = df['Edad'].astype(int)
df['Precio'] = df['Precio'].astype(float)

# ─── Texto ──────────────────────────────────────────────────────────────────
df['Nombre'] = df['Nombre'].astype(str)

# ─── Categórico (ahorra memoria en columnas con pocos valores únicos) ───────
df['Ciudad'] = df['Ciudad'].astype('category')
df['Prioridad'] = pd.Categorical(df['Prioridad'],
                                  categories=['Baja', 'Media', 'Alta'],
                                  ordered=True)  # con orden lógico

# ─── Booleano ───────────────────────────────────────────────────────────────
df['Activo'] = df['Activo'].astype(bool)

# ─── Verificar después de convertir ────────────────────────────────────────
print(df.dtypes)
```

### 6.3 Limpiar columnas numéricas con formato texto

```python
# Columna con $ o . o , que impide convertir a número
# Ej: '$1.234,56' → 1234.56
df['Precio'] = df['Precio'].str.replace('$', '', regex=False)
df['Precio'] = df['Precio'].str.replace('.', '', regex=False)
df['Precio'] = df['Precio'].str.replace(',', '.', regex=False)
df['Precio'] = pd.to_numeric(df['Precio'], errors='coerce')
```

---

## 7. Reset de índice

### 7.1 Cuándo usar reset_index

El índice se "rompe" (queda con huecos o desordenado) después de:
- Eliminar filas con `dropna()` o `drop_duplicates()`
- Filtrar con condiciones booleanas
- Ordenar con `sort_values()`

```python
# Antes: índice con huecos
# 0, 1, 3, 5, 7...  ← el 2, 4, 6 fueron eliminados

df = df.reset_index(drop=True)
# Después: índice limpio
# 0, 1, 2, 3, 4...

# drop=True → descarta el índice viejo (no lo convierte en columna)
# drop=False → convierte el índice viejo en una columna nueva (default)
```

### 7.2 Cuándo NO usar reset_index

- Cuando el índice es significativo (fechas, IDs únicos)
- Inmediatamente después de cargar el dataset (índice ya está limpio)

### 7.3 Flujo correcto

```python
# 1. Eliminar duplicados
df = df.drop_duplicates()

# 2. Eliminar nulos
df = df.dropna(subset=['Columna_Critica'])

# 3. Resetear índice → DESPUÉS de todas las eliminaciones
df = df.reset_index(drop=True)

print(f'Dataset final: {df.shape[0]} filas')
```

---

## 8. Valores atípicos (Outliers)

### 8.1 Detectar outliers con IQR

```python
def detectar_outliers_iqr(df, columna):
    Q1 = df[columna].quantile(0.25)
    Q3 = df[columna].quantile(0.75)
    IQR = Q3 - Q1
    limite_inferior = Q1 - 1.5 * IQR
    limite_superior = Q3 + 1.5 * IQR

    outliers = df[(df[columna] < limite_inferior) | (df[columna] > limite_superior)]
    print(f'Columna: {columna}')
    print(f'Rango normal: [{limite_inferior:.2f}, {limite_superior:.2f}]')
    print(f'Outliers encontrados: {len(outliers)} ({len(outliers)/len(df)*100:.1f}%)\n')
    return outliers

# Usar para columnas numéricas
outliers_importe = detectar_outliers_iqr(df, 'Importe')
```

### 8.2 Visualizar outliers

```python
# Boxplot
plt.figure(figsize=(10, 4))
df[['Importe', 'Comision_x_venta']].boxplot()
plt.title('Distribución con outliers')
plt.show()
```

### 8.3 Tratar outliers

```python
# Opción 1: Eliminar filas con outliers
Q1 = df['Importe'].quantile(0.25)
Q3 = df['Importe'].quantile(0.75)
IQR = Q3 - Q1
df = df[(df['Importe'] >= Q1 - 1.5*IQR) & (df['Importe'] <= Q3 + 1.5*IQR)]

# Opción 2: Capping — reemplazar por el límite
limite_sup = Q3 + 1.5 * IQR
df['Importe'] = df['Importe'].clip(upper=limite_sup)

# Opción 3: Dejarlos (si tienen sentido de negocio)
# Un importe muy alto puede ser una venta real, no un error
```

---

## 9. Inconsistencias en texto

```python
# ─── Espacios extra ─────────────────────────────────────────────────────────
df['Ciudad'] = df['Ciudad'].str.strip()         # quita espacios al inicio y fin
df['Ciudad'] = df['Ciudad'].str.strip().str.replace(r'\s+', ' ', regex=True)  # espacios dobles

# ─── Mayúsculas/minúsculas ──────────────────────────────────────────────────
df['Ciudad'] = df['Ciudad'].str.title()   # Primera Letra Mayúscula
df['Ciudad'] = df['Ciudad'].str.upper()   # TODO MAYÚSCULAS
df['Ciudad'] = df['Ciudad'].str.lower()   # todo minúsculas

# ─── Valores mal escritos ───────────────────────────────────────────────────
df['Rubro'] = df['Rubro'].str.replace('electrodomesticos', 'electrodomésticos')

# Reemplazar múltiples valores
reemplazos = {
    'Bs As': 'Buenos Aires',
    'Bsas': 'Buenos Aires',
    'CABA': 'Buenos Aires'
}
df['Ciudad'] = df['Ciudad'].replace(reemplazos)

# ─── Ver valores únicos para detectar inconsistencias ───────────────────────
print(df['Ciudad'].value_counts())
print(df['Rubro'].unique())
```

---

## 10. Renombrar columnas

```python
# ─── Renombrar columnas específicas ────────────────────────────────────────
df = df.rename(columns={
    'Satisfaccion_cliente': 'satisfaccion_encuesta',
    'Satisfaccion_Cliente': 'satisfaccion_sistema',
    'Fecha_Venta': 'fecha_venta'
})

# ─── Limpiar todos los nombres de columnas ─────────────────────────────────
# Minúsculas, sin espacios, sin caracteres especiales
df.columns = (df.columns
              .str.lower()
              .str.strip()
              .str.replace(' ', '_')
              .str.replace('á', 'a').str.replace('é', 'e')
              .str.replace('í', 'i').str.replace('ó', 'o')
              .str.replace('ú', 'u').str.replace('ñ', 'n'))

print(df.columns.tolist())
```

---

## 11. Creación de columnas derivadas

```python
# ─── Columna calculada ──────────────────────────────────────────────────────
df['margen'] = df['importe'] - df['costo']
df['margen_pct'] = (df['margen'] / df['importe'] * 100).round(2)

# ─── Columna condicional (np.where) ─────────────────────────────────────────
df['categoria_venta'] = np.where(df['importe'] > 300000, 'Alto', 'Normal')

# ─── Columna con múltiples condiciones ─────────────────────────────────────
condiciones = [
    df['importe'] >= 400000,
    df['importe'] >= 200000,
    df['importe'] < 200000
]
valores = ['Premium', 'Medio', 'Basico']
df['segmento'] = np.select(condiciones, valores)

# ─── Columna desde fecha ────────────────────────────────────────────────────
df['año'] = df['fecha'].dt.year
df['trimestre'] = df['fecha'].dt.quarter
df['mes_nombre'] = df['fecha'].dt.month_name()
```

---

## 12. Análisis estadístico

```python
# ─── Estadísticas por grupo ─────────────────────────────────────────────────
resumen = df.groupby('sede').agg(
    total_ventas=('importe', 'sum'),
    promedio_venta=('importe', 'mean'),
    cantidad=('importe', 'count'),
    mediana=('importe', 'median')
).round(2).sort_values('total_ventas', ascending=False)

display(resumen)

# ─── Correlación entre variables numéricas ──────────────────────────────────
correlacion = df[['importe', 'comision_x_venta', 'satisfaccion_cliente']].corr()
display(correlacion)

# ─── Tabla de frecuencias ───────────────────────────────────────────────────
print(df['rubro'].value_counts())
print(df['rubro'].value_counts(normalize=True).round(3) * 100)  # en porcentaje
```

---

## 13. Visualizaciones de EDA

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
fig.suptitle('Análisis Exploratorio de Datos', fontsize=16, fontweight='bold')

# ─── Distribución de variable numérica ─────────────────────────────────────
axes[0, 0].hist(df['importe'], bins=30, color='steelblue', edgecolor='white')
axes[0, 0].set_title('Distribución de Importes')
axes[0, 0].set_xlabel('Importe')

# ─── Ventas por categoría ───────────────────────────────────────────────────
ventas_rubro = df.groupby('rubro')['importe'].sum().sort_values(ascending=True)
axes[0, 1].barh(ventas_rubro.index, ventas_rubro.values, color='coral')
axes[0, 1].set_title('Ventas por Rubro')

# ─── Boxplot para outliers ──────────────────────────────────────────────────
axes[1, 0].boxplot(df['importe'].dropna())
axes[1, 0].set_title('Outliers en Importe')

# ─── Evolución temporal ─────────────────────────────────────────────────────
if 'fecha' in df.columns:
    ventas_mes = df.groupby(df['fecha'].dt.to_period('M'))['importe'].sum()
    axes[1, 1].plot(ventas_mes.index.astype(str), ventas_mes.values, marker='o')
    axes[1, 1].set_title('Evolución de Ventas por Mes')
    axes[1, 1].tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.show()
```

---

## 14. Exportar dataset limpio

```python
# ─── Guardar CSV limpio ─────────────────────────────────────────────────────
df.to_csv('ventas_limpio.csv', index=False, encoding='utf-8')
print('Dataset exportado como ventas_limpio.csv')

# ─── Guardar Excel ──────────────────────────────────────────────────────────
df.to_excel('ventas_limpio.xlsx', index=False, sheet_name='Datos')

# ─── Guardar con fecha en el nombre ─────────────────────────────────────────
from datetime import datetime
fecha_hoy = datetime.now().strftime('%Y%m%d')
df.to_csv(f'ventas_limpio_{fecha_hoy}.csv', index=False)

# ─── Verificar archivo exportado ────────────────────────────────────────────
df_verificacion = pd.read_csv('ventas_limpio.csv')
print(f'Archivo exportado: {df_verificacion.shape[0]} filas x {df_verificacion.shape[1]} columnas')
```

---

## 15. Checklist profesional

Antes de dar por terminado el análisis, verificá cada punto:

```python
print('='*50)
print('REPORTE FINAL DE CALIDAD DEL DATASET')
print('='*50)
print(f'Dimensiones: {df.shape[0]} filas x {df.shape[1]} columnas')
print(f'Duplicados: {df.duplicated().sum()}')
print(f'Nulos totales: {df.isnull().sum().sum()}')
print(f'Tipos de datos correctos: {dict(df.dtypes)}')
print('='*50)
```

**Lista de verificación:**

- [ ] Dataset cargado y dimensiones verificadas
- [ ] Duplicados detectados y eliminados
- [ ] Nulos analizados con estrategia justificada
- [ ] Tipos de datos corregidos (fechas, números, categorías)
- [ ] Índice reseteado después de eliminaciones
- [ ] Outliers detectados y tratados o documentados
- [ ] Inconsistencias de texto corregidas
- [ ] Columnas renombradas con criterio consistente
- [ ] Columnas derivadas creadas si aportan valor
- [ ] Visualizaciones generadas para validar
- [ ] Dataset limpio exportado

---


*Guía elaborada para uso personal y académico · Pandas Documentation: [pandas.pydata.org](https://pandas.pydata.org/docs/)*
