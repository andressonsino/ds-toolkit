# Guía de Scraping Web para Ciencia de Datos
**Repositorio:** ds-toolkit  
**Autor:** Andrés  
**Versión:** 2.0  
**Última actualización:** 2025

> Guía práctica para extraer datos de páginas web en distintos escenarios.  
> Complementa la guía de carga de datasets del toolkit.

---

## Índice

1. [Configuración de importaciones](#1-configuración-de-importaciones)
2. [Árbol de decisión — qué herramienta usar](#2-árbol-de-decisión--qué-herramienta-usar)
3. [Flujo de trabajo completo](#3-flujo-de-trabajo-completo)
4. [Escenario 1 — Tabla HTML simple](#4-escenario-1--tabla-html-simple)
5. [Escenario 2 — Múltiples tablas en la página](#5-escenario-2--múltiples-tablas-en-la-página)
6. [Escenario 3 — Tabla con estructura rota (filas duplicadas)](#6-escenario-3--tabla-con-estructura-rota-filas-duplicadas)
7. [Escenario 4 — Contenido dinámico con JavaScript](#7-escenario-4--contenido-dinámico-con-javascript)
8. [Escenario 5 — API oculta (el método más limpio)](#8-escenario-5--api-oculta-el-método-más-limpio)
9. [Escenario 6 — Sitio con bloqueo de scrapers](#9-escenario-6--sitio-con-bloqueo-de-scrapers)
10. [Escenario 7 — Paginación](#10-escenario-7--paginación)
11. [Limpieza post-scraping](#11-limpieza-post-scraping)
12. [Buenas prácticas y ética](#12-buenas-prácticas-y-ética)
13. [Errores comunes y soluciones](#13-errores-comunes-y-soluciones)
14. [Checklist de scraping](#14-checklist-de-scraping)

---

## 1. Configuración de importaciones

Ejecutar esta celda **una sola vez** al inicio de cada notebook de scraping.
Cubre todos los escenarios de la guía.

```python
# Librerías estándar
import os
import time
import random
from datetime import date

# Datos
import pandas as pd
import numpy as np

# HTTP y scraping estático
import requests
from bs4 import BeautifulSoup

# Scraping dinámico (JavaScript)
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options

# Configuración de pandas
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', 100)
pd.set_option('display.float_format', '{:.2f}'.format)

# Headers base para todas las requests
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Language": "es-AR,es;q=0.9,en;q=0.8",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
}

print("Configuración lista")
```

### Instalación en Colab (ejecutar una sola vez por sesión)

```python
# Solo si usás Selenium en Colab
!pip install selenium beautifulsoup4 --quiet
!apt-get update --quiet
!apt-get install -y chromium-chromedriver --quiet
```

---

## 2. Árbol de decisión — qué herramienta usar

Antes de escribir código, diagnosticar qué tipo de página es:

```
¿La tabla está en el HTML estático de la página?
│
├── SÍ → ¿Es una sola tabla simple?
│         ├── SÍ → pd.read_html()                        [Escenario 1]
│         └── NO → pd.read_html() + inspección índices   [Escenario 2]
│                  ¿Filas duplicadas por entidad?         [Escenario 3]
│
└── NO → ¿El contenido se carga con JavaScript?
          ├── SÍ → ¿Hay una API oculta detrás?
          │         ├── SÍ → requests + JSON             [Escenario 5] ← preferir siempre
          │         └── NO → Selenium                    [Escenario 4]
          └── NO → requests + BeautifulSoup              [Escenario 2 avanzado]
```

**Cómo saber si el contenido es dinámico:**
- Click derecho → Ver código fuente `Ctrl+U` → buscar los datos manualmente
- Si los datos **NO** aparecen → JavaScript los carga → necesitás Selenium
- Si los datos **SÍ** aparecen → `read_html` o `BeautifulSoup` alcanza

---

## 3. Flujo de trabajo completo

> Seguir este orden en **todos** los proyectos de scraping, sin excepciones.

```
PASO 1 — Diagnóstico
    ¿Los datos están en el HTML estático?
    ¿Cuántas tablas tiene la página?
    ¿Hay API oculta?

PASO 2 — Extracción raw
    Scrapear sin modificar nada
    Guardar raw en data/raw/ con fecha

PASO 3 — Inspección
    Ver shape, columnas, primeras filas
    Identificar problemas (filas duplicadas, NaN, tipos)

PASO 4 — Limpieza
    Trabajar SIEMPRE sobre df_raw.copy()
    Nunca modificar el raw

PASO 5 — Guardar limpio
    Guardar en data/processed/

PASO 6 — Documentar
    Fuente, fecha, URL, índice de tabla, decisiones de limpieza
```

### Caso real — Transfermarkt plantilla de Huracán

Este flujo resuelve el caso concreto documentado en este repositorio:
página con 36 tablas, tabla útil en índice 1, con 3 filas por jugador.

**Paso 1 — Extracción raw**

```python
URL = "https://www.transfermarkt.com.ar/club-atletico-huracan/startseite/verein/2063"

response = requests.get(URL, headers=HEADERS, timeout=10)
response.raise_for_status()
tablas = pd.read_html(response.text)

print(f"Tablas encontradas: {len(tablas)}")

for i, df in enumerate(tablas):
    print(f"Tabla {i}: shape {df.shape} | cols: {df.columns.tolist()}")
```

**Paso 2 — Guardar raw sin tocar**

```python
# Tabla correcta identificada en el paso anterior: índice 1
df_raw = tablas[1]

os.makedirs('data/raw', exist_ok=True)
fecha = date.today().strftime('%Y%m%d')
df_raw.to_csv(f'data/raw/huracan_plantilla_raw_{fecha}.csv', index=False)

print(f"Raw guardado | shape: {df_raw.shape}")
display(df_raw.head(10))
```

**Paso 3 — Inspección del raw**

```python
# Nunca limpiar sin inspeccionar primero
df = df_raw.copy()

print("=== Shape ===")
print(df.shape)

print("\n=== Tipos de datos ===")
print(df.dtypes)

print("\n=== Valores nulos por columna ===")
print(df.isnull().sum())

print("\n=== Primeras filas ===")
display(df.head(9))  # 9 filas = 3 jugadores completos (3 filas cada uno)
```

**Paso 4 — Limpieza (problema: 3 filas por jugador)**

```python
# Estructura detectada en la inspección:
# Fila 0: # = 32, Jugadores = "Sebastián Meza Portero", Edad = "14/03/2000", Valor = "450 mil"
# Fila 1: # = NaN, Jugadores = "Sebastián Meza",        Edad = NaN,          Valor = NaN
# Fila 2: # = NaN, Jugadores = "Portero",               Edad = NaN,          Valor = NaN
# Fila 3: # = 1,   Jugadores = "Hernán Galíndez...",    ...

df_reset = df.reset_index(drop=True)
registros = []
i = 0

while i < len(df_reset):
    fila = df_reset.iloc[i]

    if pd.notna(fila['#']):
        registro = {
            'numero':  fila['#'],
            'jugador': fila['Jugadores'],
            'edad':    fila['F. Nacim./Edad'],
            'valor':   fila['Valor de mercado'],
        }
        # La posición está 2 filas después
        if i + 2 < len(df_reset):
            fila_pos = df_reset.iloc[i + 2]
            if pd.isna(fila_pos['#']):
                registro['posicion'] = fila_pos['Jugadores']
                i += 2

        registros.append(registro)
    i += 1

df_limpio = pd.DataFrame(registros).reset_index(drop=True)

print(f"Jugadores consolidados: {len(df_limpio)}")
display(df_limpio.head(10))
```

**Paso 5 — Guardar limpio**

```python
os.makedirs('data/processed', exist_ok=True)
df_limpio.to_csv(f'data/processed/huracan_plantilla_{fecha}.csv', index=False)
print(f"Procesado guardado | shape: {df_limpio.shape}")
```

**Paso 6 — Documentar en el notebook**

```python
print(f"""
=== DOCUMENTACIÓN DE EXTRACCIÓN ===
Fuente       : Transfermarkt
URL          : {URL}
Fecha        : {fecha}
Tabla índice : 1 (de 36 totales)
Shape raw    : {df_raw.shape}
Shape limpio : {df_limpio.shape}
Problema     : 3 filas por jugador (nombre completo, nombre solo, posición)
Solución     : iteración con detección de fila raíz por columna '#'
Columna Nac. : vacía — Transfermarkt carga banderas como imágenes (no capturadas por read_html)
""")
```

---

## 4. Escenario 1 — Tabla HTML simple

**Cuándo:** la página tiene una tabla con los datos en el HTML estático.
**Ejemplos:** Wikipedia, FBref, tablas de estadísticas simples.

```python
# Método directo
url = "https://ejemplo.com/tabla"
df = pd.read_html(url)[0]

print(df.shape)
print(df.head())
```

```python
# Método robusto (siempre preferir este)
def scrape_tabla_simple(url: str, indice: int = 0) -> pd.DataFrame:
    """
    Extrae una tabla HTML de una URL.

    Parámetros:
        url    : URL de la página
        indice : índice de la tabla en la página (0 = primera)
    """
    try:
        response = requests.get(url, headers=HEADERS, timeout=10)
        response.raise_for_status()
        tablas = pd.read_html(response.text, header=0)

        print(f"Tablas encontradas: {len(tablas)}")
        print(f"Usando tabla[{indice}]: shape {tablas[indice].shape}")
        return tablas[indice]

    except requests.exceptions.HTTPError as e:
        print(f"Error HTTP {response.status_code}: {e}")
    except requests.exceptions.Timeout:
        print("Timeout — el servidor tardó demasiado")
    except ValueError as e:
        print(f"No se encontraron tablas HTML: {e}")
    except Exception as e:
        print(f"Error inesperado: {e}")

    return pd.DataFrame()

# Uso
df = scrape_tabla_simple("https://ejemplo.com/tabla", indice=0)
```

---

## 5. Escenario 2 — Múltiples tablas en la página

**Cuándo:** `pd.read_html()` devuelve muchas tablas y no sabés cuál es la correcta.

### Paso 1 — Inspeccionar todas las tablas

```python
def inspeccionar_tablas(url: str) -> None:
    """
    Muestra un resumen de todas las tablas encontradas.
    Ejecutar siempre antes de asumir un índice.
    """
    response = requests.get(url, headers=HEADERS, timeout=10)
    tablas = pd.read_html(response.text)

    print(f"Total de tablas encontradas: {len(tablas)}\n")

    for i, df in enumerate(tablas):
        print(f"--- Tabla {i} | shape: {df.shape} ---")
        print(f"    Columnas: {df.columns.tolist()}")
        print(df.head(2).to_string())
        print()

# Uso — siempre ejecutar primero
inspeccionar_tablas("https://ejemplo.com/pagina-con-muchas-tablas")
```

### Paso 2 — Buscar tabla por columnas clave

```python
def encontrar_tabla_por_columnas(url: str, columnas_clave: list) -> pd.DataFrame:
    """
    Busca la tabla que contenga determinadas columnas.
    Más robusto que usar índice fijo (el índice puede cambiar).
    """
    response = requests.get(url, headers=HEADERS, timeout=10)
    tablas = pd.read_html(response.text)

    for i, df in enumerate(tablas):
        cols = [str(c).lower() for c in df.columns]
        if all(col.lower() in cols for col in columnas_clave):
            print(f"Tabla encontrada en índice {i} | shape: {df.shape}")
            return df

    print("No se encontró ninguna tabla con esas columnas")
    return pd.DataFrame()

# Ejemplo
df = encontrar_tabla_por_columnas(
    "https://www.transfermarkt.com.ar/club-atletico-huracan/startseite/verein/2063",
    columnas_clave=['#', 'jugadores']
)
```

### Paso 3 — Buscar la tabla más grande

```python
def encontrar_tabla_mas_grande(url: str, min_filas: int = 10) -> pd.DataFrame:
    """
    Devuelve la tabla con más filas que supere el mínimo indicado.
    Útil cuando la tabla principal es claramente la más grande.
    """
    response = requests.get(url, headers=HEADERS, timeout=10)
    tablas = pd.read_html(response.text)

    candidatas = [(i, df) for i, df in enumerate(tablas) if len(df) >= min_filas]

    if not candidatas:
        print(f"No hay tablas con más de {min_filas} filas")
        return pd.DataFrame()

    indice, df_grande = max(candidatas, key=lambda x: len(x[1]))
    print(f"Tabla más grande: índice {indice} | shape: {df_grande.shape}")
    return df_grande

df = encontrar_tabla_mas_grande("https://ejemplo.com", min_filas=15)
```

---

## 6. Escenario 3 — Tabla con estructura rota (filas duplicadas)

**Cuándo:** `read_html` trae la tabla pero cada entidad ocupa 2 o más filas porque el HTML mezcla contenido visual con datos.

**Problema real (caso Transfermarkt):** cada jugador ocupaba 3 filas.

```
Fila 0: # = 32, Jugador = "Sebastián Meza Portero", Edad = "14/03/2000", Valor = "450 mil"
Fila 1: # = NaN, Jugador = "Sebastián Meza",        Edad = NaN,          Valor = NaN
Fila 2: # = NaN, Jugador = "Portero",               Edad = NaN,          Valor = NaN
Fila 3: # = 1,   Jugador = "Hernán Galíndez...",    ...
```

```python
def limpiar_filas_duplicadas(df: pd.DataFrame,
                              col_id: str = '#',
                              col_nombre: str = 'Jugadores',
                              col_posicion_offset: int = 2) -> pd.DataFrame:
    """
    Consolida tablas donde cada entidad ocupa múltiples filas.

    Parámetros:
        col_id              : columna que identifica el inicio de cada entidad
        col_nombre          : columna con el nombre principal
        col_posicion_offset : filas después donde está la posición (default: 2)
    """
    df_reset = df.reset_index(drop=True)
    registros = []
    i = 0

    while i < len(df_reset):
        fila = df_reset.iloc[i]

        if pd.notna(fila[col_id]):
            registro = fila.to_dict()

            if i + col_posicion_offset < len(df_reset):
                fila_pos = df_reset.iloc[i + col_posicion_offset]
                if pd.isna(fila_pos[col_id]):
                    registro['Posición'] = fila_pos[col_nombre]
                    i += col_posicion_offset

            registros.append(registro)
        i += 1

    df_limpio = pd.DataFrame(registros).reset_index(drop=True)
    print(f"Registros consolidados: {len(df_limpio)}")
    return df_limpio

# Uso
df_plantilla = limpiar_filas_duplicadas(df_raw, col_id='#', col_posicion_offset=2)
```

---

## 7. Escenario 4 — Contenido dinámico con JavaScript

**Cuándo:** los datos no aparecen en `Ctrl+U` porque JavaScript los carga después.

```python
def configurar_driver(headless: bool = True) -> webdriver.Chrome:
    """
    Configura Chrome para scraping.
    headless=True  → sin ventana visible (Colab y producción)
    headless=False → con ventana (útil para depurar)
    """
    opciones = Options()
    if headless:
        opciones.add_argument("--headless")
    opciones.add_argument("--no-sandbox")
    opciones.add_argument("--disable-dev-shm-usage")
    opciones.add_argument("--window-size=1920,1080")
    opciones.add_argument("user-agent=Mozilla/5.0")

    return webdriver.Chrome(options=opciones)


def scrape_con_selenium(url: str, selector_css: str = None,
                         espera: int = 5) -> pd.DataFrame:
    """
    Extrae una tabla de una página con contenido dinámico.

    Parámetros:
        selector_css : selector CSS del elemento con la tabla
                       Ej: 'table.items', '#tabla-jugadores', '.responsive-table'
                       Si es None, usa toda la página
        espera       : segundos de espera para que cargue el contenido
    """
    driver = configurar_driver(headless=True)
    df = pd.DataFrame()

    try:
        driver.get(url)
        time.sleep(espera)

        if selector_css:
            elemento = WebDriverWait(driver, 10).until(
                EC.presence_of_element_located((By.CSS_SELECTOR, selector_css))
            )
            html = elemento.get_attribute("outerHTML")
        else:
            html = driver.page_source

        tablas = pd.read_html(html)
        df = tablas[0]
        print(f"Tabla extraída | shape: {df.shape}")

    except Exception as e:
        print(f"Error: {e}")

    finally:
        driver.quit()  # siempre cerrar

    return df

# Uso
df = scrape_con_selenium(
    url="https://ejemplo.com/pagina-dinamica",
    selector_css="table.items"
)
```

**Cómo encontrar el selector CSS:**
```
1. Abrir la página en Chrome
2. Click derecho sobre la tabla → Inspeccionar
3. Click derecho sobre el <table> en DevTools
4. Copy → Copy selector
5. Usar ese valor en selector_css
```

---

## 8. Escenario 5 — API oculta (el método más limpio)

**Cuándo:** la página carga datos desde una API interna que devuelve JSON.
**Por qué preferirlo:** más rápido, más limpio y más estable que el scraping HTML.

**Cómo descubrir si existe:**
```
1. Chrome DevTools (F12)
2. Pestaña Network → recargar la página
3. Filtrar por "Fetch/XHR"
4. Buscar llamadas que devuelvan JSON con los datos que necesitás
5. Copiar la URL del Request
```

```python
def scrape_api_oculta(url_api: str, headers_extra: dict = None) -> pd.DataFrame:
    """
    Extrae datos de una API interna que devuelve JSON.

    Parámetros:
        headers_extra : headers adicionales que la API requiera
                        Ej: {'Referer': 'https://sitio.com', 'x-api-key': 'valor'}
    """
    headers = {**HEADERS, "Accept": "application/json"}
    if headers_extra:
        headers.update(headers_extra)

    try:
        response = requests.get(url_api, headers=headers, timeout=10)
        response.raise_for_status()
        datos = response.json()

        if isinstance(datos, list):
            df = pd.DataFrame(datos)
        elif isinstance(datos, dict):
            print(f"Claves disponibles: {list(datos.keys())}")
            # Ajustar según la respuesta real:
            # df = pd.DataFrame(datos['data'])
            # df = pd.DataFrame(datos['results'])
            df = pd.DataFrame(datos)

        print(f"Datos extraídos | shape: {df.shape}")
        return df

    except Exception as e:
        print(f"Error: {e}")
        return pd.DataFrame()

# Uso
df = scrape_api_oculta(
    url_api="https://api.ejemplo.com/v1/jugadores?liga=arg",
    headers_extra={"Referer": "https://ejemplo.com"}
)
```

---

## 9. Escenario 6 — Sitio con bloqueo de scrapers

**Síntomas:**
- `403 Forbidden` → el servidor detectó que no sos un navegador
- `429 Too Many Requests` → demasiadas requests seguidas
- Página de Cloudflare o CAPTCHA → protección activa

```python
def scrape_con_reintentos(url: str, max_intentos: int = 3,
                           espera_base: float = 2.0) -> list:
    """
    Scraping con reintentos automáticos ante errores de throttling.

    Parámetros:
        max_intentos : reintentos máximos ante error 429
        espera_base  : segundos base (se multiplica en cada reintento)
    """
    for intento in range(1, max_intentos + 1):
        try:
            espera = espera_base * intento + random.uniform(0.5, 1.5)
            time.sleep(espera)

            response = requests.get(url, headers=HEADERS, timeout=15)

            if response.status_code == 429:
                print(f"Rate limit (intento {intento}/{max_intentos}) — esperando {espera:.1f}s")
                continue

            response.raise_for_status()
            tablas = pd.read_html(response.text)
            print(f"Éxito en intento {intento} | Tablas: {len(tablas)}")
            return tablas

        except requests.exceptions.HTTPError as e:
            print(f"Error HTTP: {e}")
            break
        except Exception as e:
            print(f"Error en intento {intento}: {e}")

    print("Se agotaron los reintentos")
    return []


def crear_sesion() -> requests.Session:
    """
    Sesión HTTP persistente — mantiene cookies entre requests.
    Más difícil de detectar como bot que requests aislados.
    """
    sesion = requests.Session()
    sesion.headers.update(HEADERS)
    return sesion

# Uso con sesión
sesion = crear_sesion()
response = sesion.get("https://ejemplo.com/datos")
tablas = pd.read_html(response.text)
```

---

## 10. Escenario 7 — Paginación

**Cuándo:** los datos están repartidos en múltiples páginas.

```python
def scrape_paginado(url_base: str, param_pagina: str = 'page',
                    pagina_inicio: int = 1, max_paginas: int = 10,
                    espera: float = 2.0) -> pd.DataFrame:
    """
    Extrae tablas de múltiples páginas y las concatena.

    Parámetros:
        url_base     : URL base sin el parámetro de página
        param_pagina : nombre del parámetro en la URL (ej: 'page', 'p')
        pagina_inicio: número de la primera página
        max_paginas  : límite de páginas (evita iterar infinito)
        espera       : segundos entre requests

    Ejemplos de URL:
        https://ejemplo.com/datos?page=1
        https://ejemplo.com/datos?p=2
    """
    todos_los_dfs = []

    for pagina in range(pagina_inicio, pagina_inicio + max_paginas):
        url = f"{url_base}?{param_pagina}={pagina}"

        try:
            time.sleep(espera)
            response = requests.get(url, headers=HEADERS, timeout=10)
            response.raise_for_status()
            tablas = pd.read_html(response.text)

            if not tablas:
                print(f"   Página {pagina}: sin tablas — fin")
                break

            df_pagina = tablas[0]

            if len(df_pagina) <= 1:
                print(f"   Página {pagina}: tabla vacía — fin")
                break

            todos_los_dfs.append(df_pagina)
            print(f"   Página {pagina}: {len(df_pagina)} filas")

        except Exception as e:
            print(f"   Página {pagina}: error — {e}")
            break

    if todos_los_dfs:
        df_total = pd.concat(todos_los_dfs, ignore_index=True)
        df_total = df_total.drop_duplicates().reset_index(drop=True)
        print(f"\nTotal: {len(df_total)} filas en {len(todos_los_dfs)} páginas")
        return df_total

    return pd.DataFrame()

# Uso
df = scrape_paginado(
    url_base="https://ejemplo.com/jugadores",
    param_pagina="page",
    max_paginas=5
)
```

---

## 11. Limpieza post-scraping

```python
def guardar_raw(df: pd.DataFrame, nombre: str, carpeta: str = 'data/raw') -> str:
    """
    Guarda el DataFrame crudo antes de cualquier transformación.
    Incluye la fecha en el nombre para versionado básico.
    Retorna la ruta del archivo guardado.
    """
    os.makedirs(carpeta, exist_ok=True)
    fecha = date.today().strftime('%Y%m%d')
    ruta = f"{carpeta}/{nombre}_{fecha}.csv"
    df.to_csv(ruta, index=False)
    print(f"Raw guardado: {ruta}")
    return ruta


def guardar_procesado(df: pd.DataFrame, nombre: str, carpeta: str = 'data/processed') -> str:
    """
    Guarda el DataFrame limpio y procesado.
    Retorna la ruta del archivo guardado.
    """
    os.makedirs(carpeta, exist_ok=True)
    fecha = date.today().strftime('%Y%m%d')
    ruta = f"{carpeta}/{nombre}_{fecha}.csv"
    df.to_csv(ruta, index=False)
    print(f"Procesado guardado: {ruta}")
    return ruta


def limpiar_post_scraping(df: pd.DataFrame, col_id: str = None) -> pd.DataFrame:
    """
    Limpieza estándar después de cualquier scraping.
    Llamar siempre sobre df_raw.copy(), nunca sobre el raw directamente.
    """
    df = df.copy()

    # 1. Eliminar filas que son repetición del encabezado
    for col in df.columns:
        df = df[df[col] != str(col)]

    # 2. Eliminar filas completamente vacías
    df = df.dropna(how='all')

    # 3. Eliminar columnas completamente vacías
    df = df.dropna(axis=1, how='all')

    # 4. Limpiar espacios en strings
    for col in df.select_dtypes(include='object').columns:
        df[col] = df[col].str.strip()

    # 5. Convertir columnas numéricas
    for col in df.columns:
        if col != col_id:
            df[col] = pd.to_numeric(df[col], errors='ignore')

    # 6. Resetear índice
    df = df.reset_index(drop=True)

    print(f"Shape post-limpieza: {df.shape}")
    return df


def limpiar_headers_multinivel(df: pd.DataFrame) -> pd.DataFrame:
    """
    Aplana columnas con MultiIndex.
    Frecuente en FBref y tablas con subencabezados.
    """
    if isinstance(df.columns, pd.MultiIndex):
        df.columns = ['_'.join(str(c) for c in col).strip('_ ')
                      for col in df.columns]
        print("MultiIndex aplanado")
    return df


def normalizar_nombres_columnas(df: pd.DataFrame) -> pd.DataFrame:
    """
    Estandariza nombres de columnas: minúsculas, sin espacios ni caracteres raros.
    """
    df.columns = (
        df.columns
        .str.strip()
        .str.lower()
        .str.replace(' ', '_')
        .str.replace('.', '', regex=False)
        .str.replace('/', '_', regex=False)
        .str.replace('(', '', regex=False)
        .str.replace(')', '', regex=False)
    )
    return df
```

---

## 12. Buenas prácticas y ética

### Respetar el servidor

```python
# Siempre esperar entre requests
time.sleep(2)

# Mejor: espera aleatoria — más natural, menos detectable como bot
time.sleep(random.uniform(1.5, 3.5))

# Verificar robots.txt antes de scrapear cualquier sitio:
# https://ejemplo.com/robots.txt
# No scrapear rutas marcadas como Disallow
```

### Reglas generales

- Verificar el `robots.txt` del sitio antes de scrapear
- No hacer más de 1 request por segundo en sitios que no son APIs
- No scrapear datos personales sensibles sin propósito claro
- Si el sitio tiene API oficial → usarla siempre antes que scraping
- Documentar siempre: fuente, fecha, URL, decisiones tomadas

---

## 13. Errores comunes y soluciones

| Error | Causa probable | Solución |
|-------|---------------|----------|
| `403 Forbidden` | Servidor bloqueó la request | Usar `HEADERS` completos con User-Agent |
| `429 Too Many Requests` | Demasiadas requests seguidas | Usar `scrape_con_reintentos()` con sleep |
| `ValueError: No tables found` | No hay tablas HTML en la página | Contenido dinámico → usar Selenium |
| Tabla con índice incorrecto | Página con múltiples tablas | Usar `inspeccionar_tablas()` primero |
| Columnas con MultiIndex | FBref y tablas con subencabezados | Usar `limpiar_headers_multinivel()` |
| Filas duplicadas por jugador | HTML mezcla visual con datos | Usar `limpiar_filas_duplicadas()` |
| Columnas numéricas como string | `read_html` no convierte solo | `pd.to_numeric(col, errors='coerce')` |
| Columna `Nac.` vacía | Transfermarkt carga banderas como imágenes | BeautifulSoup para extraer atributo `alt` |
| Timeout | Servidor tarda demasiado | Aumentar `timeout=` o usar Selenium |
| Índice cambia entre temporadas | El sitio actualizó su estructura | Usar `encontrar_tabla_por_columnas()` |

---

## 14. Checklist de scraping

### Diagnóstico previo
- [ ] Verificar si el contenido está en el HTML estático (`Ctrl+U`)
- [ ] Revisar `robots.txt` del sitio
- [ ] Buscar API oculta en DevTools → Network → Fetch/XHR

### Extracción
- [ ] Usar `HEADERS` con User-Agent en todas las requests
- [ ] Agregar espera entre requests (`time.sleep`)
- [ ] Ejecutar `inspeccionar_tablas()` antes de asumir un índice fijo
- [ ] Verificar shape, columnas y primeras filas del raw

### Post-extracción
- [ ] Guardar raw con `guardar_raw()` antes de cualquier limpieza
- [ ] Limpiar siempre sobre `df_raw.copy()`
- [ ] Verificar columnas vacías (¿datos estáticos o dinámicos?)
- [ ] Guardar procesado con `guardar_procesado()`
- [ ] Documentar: fuente, fecha, URL, índice de tabla, problemas encontrados

---

*Guía viva — agregar nuevos escenarios a medida que aparecen en proyectos reales.*
