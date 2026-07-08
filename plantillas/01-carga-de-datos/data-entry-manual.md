# Data Entry Manual
**Repositorio:** ds-toolkit  
**Autor:** Andrés  
**Versión:** 1.0  
**Última actualización:** 2025

> Guía para recolección y documentación profesional de datos copiados manualmente desde fuentes externas.  
> Es una práctica válida y común — no es menos profesional que el scraping cuando el contexto lo justifica.

---

## Índice

1. [Cuándo usar data entry manual](#1-cuándo-usar-data-entry-manual)
2. [Flujo de trabajo](#2-flujo-de-trabajo)
3. [Documentación de la fuente](#3-documentación-de-la-fuente)
4. [Carga del CSV al notebook](#4-carga-del-csv-al-notebook)
5. [Validación post-carga](#5-validación-post-carga)
6. [Estándares de formato por tipo de dato](#6-estándares-de-formato-por-tipo-de-dato)
7. [Buenas prácticas](#7-buenas-prácticas)
8. [Checklist](#8-checklist)

---

## 1. Cuándo usar data entry manual

| Situación | Data entry manual | Scraping |
|-----------|:-----------------:|:--------:|
| Dataset pequeño (< 200 filas) | ✅ | Excesivo |
| Tabla con imágenes o JS complejo | ✅ | Difícil |
| Dato puntual que no se repite | ✅ | No vale |
| Prototipo o análisis rápido | ✅ | Excesivo |
| Dataset grande (> 500 filas) | Lento | ✅ |
| Datos que se actualizan periódicamente | Inviable | ✅ |
| Múltiples páginas o temporadas | Inviable | ✅ |

> **Regla práctica:** si copiar a mano lleva menos de 30 minutos, hacelo a mano.

---

## 2. Flujo de trabajo

```
1. Identificar la fuente y verificar que es confiable
2. Crear un Google Sheets nuevo
3. Copiar los datos desde la fuente
4. Aplicar estándares de formato (ver sección 6)
5. Documentar la fuente en una pestaña aparte
6. Exportar como CSV (Archivo → Descargar → CSV)
7. Guardar en data/raw/ del proyecto
8. Cargar y validar en el notebook
```

---

## 3. Documentación de la fuente

### En el Google Sheets

Crear una pestaña llamada `fuente` con esta estructura:

| Campo | Valor |
|-------|-------|
| Fuente | Transfermarkt |
| URL | https://www.transfermarkt.com.ar/... |
| Fecha de consulta | DD/MM/AAAA |
| Columnas copiadas | Nac., Valor de mercado |
| Notas | Nacionalidades en código ISO 3166-1 alpha-2 |

### En el notebook

```python
# ── Documentación de fuente ───────────────────────────────────────────────
# Método de recolección : Data entry manual (Google Sheets → CSV)
# Fuente                : NOMBRE_FUENTE                    # ← reemplazar
# URL                   : https://...                      # ← reemplazar
# Fecha de consulta     : DD/MM/AAAA                       # ← reemplazar
# Columnas recolectadas : COLUMNA_1, COLUMNA_2             # ← reemplazar
# Notas                 : aclaración de criterios usados   # ← reemplazar
```

### En el commit de GitHub

El mensaje del commit debe reflejar el origen del dato:

```
git commit -m "agrego nacionalidades del plantel — fuente: Transfermarkt (manual)"
```

---

## 4. Carga del CSV al notebook

```python
# ── Carga del archivo de data entry ──────────────────────────────────────
ruta = 'data/raw/NOMBRE_ARCHIVO.csv'                       # ← reemplazar

df_entry = pd.read_csv(ruta)

print(f'Filas: {df_entry.shape[0]} | Columnas: {df_entry.shape[1]}')
display(df_entry.head())
```

```python
# ── Merge con el dataset principal ───────────────────────────────────────
# Cuándo: el archivo de data entry complementa columnas del dataset principal

df_clean = df_clean.merge(
    df_entry,
    on='COLUMNA_CLAVE',      # ← columna común entre ambos (ej: 'jugador', 'id')
    how='left'               # left: mantiene todos los registros del dataset principal
)

print(f'Shape post-merge: {df_clean.shape}')
```

```python
# ── Alternativa: completar columna existente con NaN ─────────────────────
# Cuándo: la columna ya existe en df_clean pero tiene nulos

mapa = dict(zip(df_entry['COLUMNA_CLAVE'], df_entry['COLUMNA_VALOR']))  # ← reemplazar

df_clean['COLUMNA_NULOS'] = df_clean['COLUMNA_CLAVE'].map(mapa)         # ← reemplazar
```

---

## 5. Validación post-carga

```python
# ── Verificar cobertura ───────────────────────────────────────────────────
total = len(df_clean)
completados = df_clean['COLUMNA_COMPLETADA'].notna().sum()               # ← reemplazar
faltantes = total - completados

print(f'Cobertura: {completados}/{total} ({completados/total*100:.1f}%)')

if faltantes > 0:
    print(f'\n⚠️  {faltantes} registros sin completar:')
    display(df_clean[df_clean['COLUMNA_COMPLETADA'].isna()][['COLUMNA_CLAVE', 'COLUMNA_COMPLETADA']])
else:
    print('✅ Cobertura completa')
```

```python
# ── Verificar valores válidos ─────────────────────────────────────────────
# Cuándo: la columna debe tener valores de un conjunto conocido
# Ejemplo: nacionalidades deben ser códigos ISO válidos

valores_validos = ['AR', 'BR', 'UY', 'PY', 'BO', 'CL', 'CO', 'PE', 'EC', 'VE']  # ← reemplazar

invalidos = df_clean[
    df_clean['COLUMNA'].notna() &
    ~df_clean['COLUMNA'].isin(valores_validos)
]

if len(invalidos) > 0:
    print(f'⚠️  {len(invalidos)} valores fuera del conjunto esperado:')
    display(invalidos[['COLUMNA_CLAVE', 'COLUMNA']])
else:
    print('✅ Todos los valores son válidos')
```

```python
# ── Verificar consistencia con el dataset original ────────────────────────
# Confirmar que el merge no generó filas duplicadas ni perdió registros

assert len(df_clean) == filas_originales, \
    f'Error: filas antes {filas_originales} vs después {len(df_clean)}'

print('✅ Integridad del dataset confirmada')
```

---

## 6. Estándares de formato por tipo de dato

Usar siempre formatos estándar internacionales para facilitar el procesamiento posterior.

| Tipo de dato | Formato estándar | Ejemplo |
|-------------|-----------------|---------|
| Nacionalidad | ISO 3166-1 alpha-2 | `AR`, `BR`, `UY` |
| Fecha | DD/MM/AAAA | `14/03/2000` |
| Moneda | Número sin símbolo | `450000` |
| Booleano | `True` / `False` | `True` |
| Texto vacío | Dejar celda vacía (no poner `-` ni `N/A`) | |
| Nombres propios | Respetar tildes y mayúsculas | `Hernán Galíndez` |

> **Por qué celdas vacías y no `-`:** los guiones llegan como strings al notebook y requieren limpieza adicional. Las celdas vacías se cargan directamente como `NaN`.

---

## 7. Buenas prácticas

- **Una fila por entidad** — no mezclar formatos dentro de una columna
- **Sin filas de totales** — no agregar sumas o promedios al final de la tabla
- **Sin celdas combinadas** — cada celda debe tener un solo valor
- **Sin formato visual** — colores, negritas y bordes se pierden al exportar a CSV
- **Nombres de columnas sin espacios** — usar `_` en vez de espacio (`fecha_nacimiento`)
- **Verificar antes de exportar** — revisar que no haya celdas con espacios invisibles
- **Guardar el Sheets** — mantener el Google Sheets original como respaldo del CSV

---

## 8. Checklist

### Antes de copiar
- [ ] Verificar que la fuente es confiable
- [ ] Definir qué columnas se van a copiar
- [ ] Elegir el formato estándar para cada tipo de dato (ver sección 6)

### Durante la carga en Sheets
- [ ] Una fila por entidad
- [ ] Celdas vacías para valores faltantes (no `-` ni `N/A`)
- [ ] Nombres de columnas sin espacios
- [ ] Pestaña `fuente` con URL, fecha y notas

### Al exportar
- [ ] Archivo → Descargar → Valores separados por comas (.csv)
- [ ] Guardar en `data/raw/` con nombre descriptivo
- [ ] Commit con mensaje que mencione la fuente y el método

### En el notebook
- [ ] Documentar fuente en comentario al inicio de la celda
- [ ] Verificar cobertura con el bloque de validación
- [ ] Verificar que no se generaron duplicados post-merge
- [ ] Confirmar que los valores son del conjunto esperado

---

*Para recolección automatizada desde la misma fuente → ver `guia-scraping-web.md`*
