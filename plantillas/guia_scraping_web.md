# Guía de Scraping Web para Ciencia de Datos
**Repositorio:** ds-toolkit  
**Autor:** Andrés  
**Versión:** 1.0  
**Última actualización:** 2025

> Guía práctica para extraer datos de páginas web en distintos escenarios.  
> Complementa la guía de carga de datasets del toolkit.

---

## Índice

1. [Árbol de decisión — qué herramienta usar](#1-árbol-de-decisión--qué-herramienta-usar)
2. [Escenario 1 — Tabla HTML simple](#2-escenario-1--tabla-html-simple)
3. [Escenario 2 — Múltiples tablas en la página](#3-escenario-2--múltiples-tablas-en-la-página)
4. [Escenario 3 — Tabla con estructura rota (filas duplicadas)](#4-escenario-3--tabla-con-estructura-rota-filas-duplicadas)
5. [Escenario 4 — Contenido dinámico con JavaScript](#5-escenario-4--contenido-dinámico-con-javascript)
6. [Escenario 5 — API oculta (el método más limpio)](#6-escenario-5--api-oculta-el-método-más-limpio)
7. [Escenario 6 — Sitio con bloqueo de scrapers](#7-escenario-6--sitio-con-bloqueo-de-scrapers)
8. [Escenario 7 — Paginación](#8-escenario-7--paginación)
9. [Limpieza post-scraping](#9-limpieza-post-scraping)
10. [Buenas prácticas y ética](#10-buenas-prácticas-y-ética)
11. [Errores comunes y soluciones](#11-errores-comunes-y-soluciones)
12. [Checklist de scraping](#12-checklist-de-scraping)

---

## 1. Árbol de decisión — qué herramienta usar

Antes de escribir código, diagnosticar qué tipo de página es:

```
¿La tabla está en el HTML estático de la página?
│
├── SÍ → ¿Es una sola tabla simple?
│         ├── SÍ → pd.read_html()          [Escenario 1]
│         └── NO → pd.read_html() + inspección de índices  [Escenario 2]
│
└── NO → ¿El contenido se carga con JavaScript?
          ├── SÍ → ¿Hay una API oculta detrás?
          │         ├── SÍ → requests + JSON    [Escenario 5] ← preferir siempre
          │         └── NO → Selenium / Playwright  [Escenario 4]
          └── NO → requests + BeautifulSoup   [Escenario 2 avanzado]
```

**Cómo saber si el contenido es dinámico:**
- Click derecho → Ver código fuente (Ctrl+U) → buscar los datos a mano
- Si los datos NO aparecen en el código fuente → JavaScript los carga → necesitás Selenium
- Si los datos SÍ aparecen → `read_html` o `BeautifulSoup` alcanza

---

## 2. Escenario 1 — Tabla HTML simple

**Cuándo:** la página tiene una sola tabla con los datos que necesitás y están en el HTML estático.

**Ejemplos:** Wikipedia, FBref, tablas de estadísticas simples.

```python
import pandas as pd
import requests

# --- Método directo (si no hay bloqueo de user-agent) ---
url = "https://ejemplo.com/tabla"
df = pd.read_html(url)[0]  # [0] = primera tabla de la página

print(df.shape)
print(df.head())
```

```python
# --- Método robusto (siempre recomendado) ---
import pandas as pd
import requests

def scrape_tabla_simple(url: str, indice: int = 0) -> pd.DataFrame:
    """
    Extrae una tabla HTML de una URL.
    Incluye user-agent para evitar bloqueos básicos.
    
    Parámetros:
        url    : URL de la página
        indice : índice de la tabla en la página (0 = primera)
    """
    headers = {"User-Agent": "Mozilla/5.0"}
    
    try:
        response = requests.get(url, headers=headers, timeout=10)
        response.raise_for_status()  # lanza error si status != 200
        tablas = pd.read_html(response.text, header=0)
        
        print(f"✅ Tablas encontradas: {len(tablas)}")
        print(f"   Usando tabla[{indice}]: shape {tablas[indice].shape}")
        return tablas[indice]
    
    except requests.exceptions.HTTPError as e:
        print(f"❌ Error HTTP {response.status_code}: {e}")
    except requests.exceptions.Timeout:
        print("❌ Timeout — el servidor tardó demasiado")
    except ValueError as e:
        print(f"❌ No se encontraron tablas HTML en la página: {e}")
    except Exception as e:
        print(f"❌ Error inesperado: {e}")
    
    return pd.DataFrame()

# Uso
df = scrape_tabla_simple("https://ejemplo.com/tabla", indice=0)
```

---

## 3. Escenario 2 — Múltiples tablas en la página

**Cuándo:** `pd.read_html()` devuelve una lista con muchas tablas y no sabés cuál es la correcta.

**Problema real (caso Transfermarkt):** la página tenía 36 tablas. La tabla útil era la índice 1, pero su estructura interna también tenía problemas.

### Paso 1 — Inspeccionar todas las tablas

```python
import pandas as pd
import requests

def inspeccionar_tablas(url: str) -> None:
    """
    Muestra un resumen de todas las tablas encontradas en la página.
    Usar para identificar el índice correcto antes de extraer.
    """
    headers = {"User-Agent": "Mozilla/5.0"}
    response = requests.get(url, headers=headers, timeout=10)
    tablas = pd.read_html(response.text)
    
    print(f"Total de tablas encontradas: {len(tablas)}\n")
    
    for i, df in enumerate(tablas):
        print(f"--- Tabla {i} | shape: {df.shape} ---")
        print(f"    Columnas: {df.columns.tolist()}")
        print(df.head(2).to_string())
        print()

# Uso — ejecutar primero para explorar
inspeccionar_tablas("https://ejemplo.com/pagina-con-muchas-tablas")
```

### Paso 2 — Identificar la tabla correcta automáticamente

```python
def encontrar_tabla_por_columnas(url: str, columnas_clave: list) -> pd.DataFrame:
    """
    Busca la tabla que contenga determinadas columnas.
    Útil cuando el índice puede cambiar entre temporadas o actualizaciones.
    
    Parámetros:
        columnas_clave : lista de nombres de columnas que debe tener la tabla buscada
    """
    headers = {"User-Agent": "Mozilla/5.0"}
    response = requests.get(url, headers=headers, timeout=10)
    tablas = pd.read_html(response.text)
    
    for i, df in enumerate(tablas):
        cols = [str(c).lower() for c in df.columns]
        if all(col.lower() in cols for col in columnas_clave):
            print(f"✅ Tabla encontrada en índice {i} | shape: {df.shape}")
            return df
    
    print("❌ No se encontró ninguna tabla con esas columnas")
    return pd.DataFrame()

# Ejemplo — buscar tabla que tenga '#' y 'Jugadores'
df = encontrar_tabla_por_columnas(
    "https://www.transfermarkt.com.ar/club-atletico-huracan/startseite/verein/2063",
    columnas_clave=['#', 'jugadores']
)
```

### Paso 3 — Encontrar la tabla más grande (heurística útil)

```python
def encontrar_tabla_mas_grande(url: str, min_filas: int = 10) -> pd.DataFrame:
    """
    Devuelve la tabla con más filas que supere el mínimo indicado.
    Útil para páginas donde la tabla principal es claramente la más grande.
    """
    headers = {"User-Agent": "Mozilla/5.0"}
    response = requests.get(url, headers=headers, timeout=10)
    tablas = pd.read_html(response.text)
    
    candidatas = [(i, df) for i, df in enumerate(tablas) if len(df) >= min_filas]
    
    if not candidatas:
        print(f"❌ No hay tablas con más de {min_filas} filas")
        return pd.DataFrame()
    
    indice, df_grande = max(candidatas, key=lambda x: len(x[1]))
    print(f"✅ Tabla más grande: índice {indice} | shape: {df_grande.shape}")
    return df_grande

df = encontrar_tabla_mas_grande("https://ejemplo.com", min_filas=15)
```

---

## 4. Escenario 3 — Tabla con estructura rota (filas duplicadas)

**Cuándo:** `read_html` trae la tabla pero cada entidad ocupa 2 o más filas porque el HTML mezcla contenido visual con datos.

**Problema real (caso Transfermarkt):** cada jugador ocupaba 3 filas — una con `#` y nombre, una con nombre repetido, una con posición. Solo la primera fila tenía datos completos.

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
    
    Estrategia: la fila "real" tiene valor en col_id (ej: número de jugador).
    Las filas siguientes son datos secundarios (posición, etc.).
    
    Parámetros:
        col_id              : columna que identifica el inicio de cada entidad
        col_nombre          : columna con el nombre principal
        col_posicion_offset : cuántas filas después está la posición (default: 2)
    """
    df_reset = df.reset_index(drop=True)
    registros = []
    i = 0
    
    while i < len(df_reset):
        fila = df_reset.iloc[i]
        
        # Fila "raíz": tiene valor en la columna identificadora
        if pd.notna(fila[col_id]):
            registro = fila.to_dict()
            
            # Buscar posición en filas siguientes
            if i + col_posicion_offset < len(df_reset):
                fila_posicion = df_reset.iloc[i + col_posicion_offset]
                if pd.isna(fila_posicion[col_id]):  # confirmar que es fila secundaria
                    registro['Posición'] = fila_posicion[col_nombre]
                    i += col_posicion_offset  # saltar las filas secundarias
            
            registros.append(registro)
        i += 1
    
    df_limpio = pd.DataFrame(registros).reset_index(drop=True)
    print(f"✅ Registros consolidados: {len(df_limpio)}")
    return df_limpio

# Uso con el caso Transfermarkt
headers = {"User-Agent": "Mozilla/5.0"}
response = requests.get(
    "https://www.transfermarkt.com.ar/club-atletico-huracan/startseite/verein/2063",
    headers=headers
)
tablas = pd.read_html(response.text)
df_raw = tablas[1]  # tabla principal identificada en el Escenario 2

df_plantilla = limpiar_filas_duplicadas(df_raw, col_id='#', col_posicion_offset=2)
print(df_plantilla[['#', 'Jugadores', 'Posición', 'Valor de mercado']].head(10))
```

---

## 5. Escenario 4 — Contenido dinámico con JavaScript

**Cuándo:** los datos no aparecen en el código fuente HTML (Ctrl+U) porque se cargan con JavaScript después de que la página abre.

**Herramienta:** Selenium — controla un navegador real que ejecuta el JavaScript.

### Instalación en Colab

```python
# Ejecutar una sola vez por sesión de Colab
!pip install selenium --quiet
!apt-get update --quiet
!apt-get install -y chromium-chromedriver --quiet
```

### Código base

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
import pandas as pd
import time

def configurar_driver(headless: bool = True) -> webdriver.Chrome:
    """
    Configura Chrome para scraping.
    headless=True → sin ventana visible (recomendado para Colab y producción)
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
        url          : URL de la página
        selector_css : selector CSS del elemento que contiene la tabla
                       (ej: 'table.items', '#tabla-jugadores', '.responsive-table')
                       Si es None, busca la primera tabla de la página.
        espera       : segundos de espera para que cargue el contenido
    """
    driver = configurar_driver(headless=True)
    df = pd.DataFrame()
    
    try:
        driver.get(url)
        time.sleep(espera)  # espera básica
        
        if selector_css:
            # Espera explícita — más robusta que time.sleep
            elemento = WebDriverWait(driver, 10).until(
                EC.presence_of_element_located((By.CSS_SELECTOR, selector_css))
            )
            html = elemento.get_attribute("outerHTML")
        else:
            html = driver.page_source
        
        tablas = pd.read_html(html)
        df = tablas[0]
        print(f"✅ Tabla extraída | shape: {df.shape}")
    
    except Exception as e:
        print(f"❌ Error: {e}")
    
    finally:
        driver.quit()  # siempre cerrar el driver
    
    return df

# Uso
df = scrape_con_selenium(
    url="https://ejemplo.com/pagina-dinamica",
    selector_css="table.items"  # inspeccionar la clase en el navegador
)
```

### Cómo encontrar el selector CSS correcto

```
1. Abrir la página en Chrome
2. Click derecho sobre la tabla → Inspeccionar
3. En el panel de DevTools, click derecho sobre el elemento <table>
4. Copy → Copy selector
5. Usar ese selector en selector_css
```

---

## 6. Escenario 5 — API oculta (el método más limpio)

**Cuándo:** la página carga datos desde una API interna que devuelve JSON.  
**Por qué preferirlo:** más rápido, más limpio y más estable que el scraping HTML.

**Cómo descubrir si existe:**

```
1. Abrir Chrome DevTools (F12)
2. Ir a la pestaña Network
3. Recargar la página
4. Filtrar por "Fetch/XHR"
5. Buscar llamadas que devuelvan JSON con los datos que necesitás
6. Click en esa llamada → copiar la URL del Request
```

```python
import requests
import pandas as pd

def scrape_api_oculta(url_api: str, headers_extra: dict = None) -> pd.DataFrame:
    """
    Extrae datos de una API interna que devuelve JSON.
    
    Parámetros:
        url_api      : URL de la API encontrada en DevTools
        headers_extra: headers adicionales que la API pueda requerir
                       (ej: Referer, Accept, x-api-key)
    """
    headers = {
        "User-Agent": "Mozilla/5.0",
        "Accept": "application/json",
    }
    if headers_extra:
        headers.update(headers_extra)
    
    try:
        response = requests.get(url_api, headers=headers, timeout=10)
        response.raise_for_status()
        datos = response.json()
        
        # Los datos pueden venir en distintas estructuras
        if isinstance(datos, list):
            df = pd.DataFrame(datos)
        elif isinstance(datos, dict):
            # Buscar la clave que contiene la lista de datos
            print(f"Claves disponibles: {list(datos.keys())}")
            # Ajustar según la respuesta real:
            # df = pd.DataFrame(datos['data'])
            # df = pd.DataFrame(datos['results'])
            df = pd.DataFrame(datos)
        
        print(f"✅ Datos extraídos | shape: {df.shape}")
        return df
    
    except Exception as e:
        print(f"❌ Error: {e}")
        return pd.DataFrame()

# Ejemplo con una API que requiere headers específicos
df = scrape_api_oculta(
    url_api="https://api.ejemplo.com/v1/jugadores?liga=arg",
    headers_extra={"Referer": "https://ejemplo.com", "Accept-Language": "es-AR"}
)
```

---

## 7. Escenario 6 — Sitio con bloqueo de scrapers

**Cuándo:** el servidor detecta el scraper y devuelve error 403, 429 o una página de Cloudflare.

**Síntomas comunes:**
- `403 Forbidden` → el servidor reconoció que no sos un navegador real
- `429 Too Many Requests` → demasiadas requests en poco tiempo
- Página de Cloudflare o CAPTCHA → protección activa

```python
import requests
import pandas as pd
import time
import random

# --- Estrategia 1: Headers completos que imitan un navegador ---
HEADERS_NAVEGADOR = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Language": "es-AR,es;q=0.9,en;q=0.8",
    "Accept-Encoding": "gzip, deflate, br",
    "Connection": "keep-alive",
    "Upgrade-Insecure-Requests": "1",
}

def scrape_con_reintentos(url: str, max_intentos: int = 3,
                           espera_base: float = 2.0) -> pd.DataFrame:
    """
    Scraping con manejo de errores de throttling y reintentos automáticos.
    
    Parámetros:
        max_intentos : número máximo de reintentos ante error 429
        espera_base  : segundos de espera base (se multiplica en cada reintento)
    """
    for intento in range(1, max_intentos + 1):
        try:
            # Espera aleatoria para no parecer un bot
            espera = espera_base * intento + random.uniform(0.5, 1.5)
            time.sleep(espera)
            
            response = requests.get(url, headers=HEADERS_NAVEGADOR, timeout=15)
            
            if response.status_code == 429:
                print(f"⚠️  Rate limit (intento {intento}/{max_intentos}) — esperando {espera:.1f}s")
                continue
            
            response.raise_for_status()
            tablas = pd.read_html(response.text)
            print(f"✅ Éxito en intento {intento} | Tablas encontradas: {len(tablas)}")
            return tablas
        
        except requests.exceptions.HTTPError as e:
            print(f"❌ Error HTTP: {e}")
            break
        except Exception as e:
            print(f"❌ Error en intento {intento}: {e}")
    
    print("❌ Se agotaron los reintentos")
    return []

# --- Estrategia 2: Sesión persistente (más creíble para el servidor) ---
def crear_sesion() -> requests.Session:
    """
    Crea una sesión HTTP que mantiene cookies y headers entre requests.
    Más difícil de detectar como bot que requests aislados.
    """
    sesion = requests.Session()
    sesion.headers.update(HEADERS_NAVEGADOR)
    return sesion

sesion = crear_sesion()
response = sesion.get("https://ejemplo.com/datos")
```

---

## 8. Escenario 7 — Paginación

**Cuándo:** los datos están repartidos en múltiples páginas (página 1, 2, 3...).

```python
import pandas as pd
import requests
import time

def scrape_paginado(url_base: str, param_pagina: str = 'page',
                    pagina_inicio: int = 1, max_paginas: int = 10,
                    espera: float = 2.0) -> pd.DataFrame:
    """
    Extrae tablas de múltiples páginas y las concatena.
    
    Parámetros:
        url_base     : URL base sin el parámetro de página
        param_pagina : nombre del parámetro de paginación en la URL
        pagina_inicio: número de la primera página
        max_paginas  : límite de páginas a recorrer (seguridad)
        espera       : segundos entre requests
    
    Ejemplos de URL:
        https://ejemplo.com/datos?page=1
        https://ejemplo.com/datos?p=2
        https://fbref.com/stats?comp=21&page=3
    """
    headers = {"User-Agent": "Mozilla/5.0"}
    todos_los_dfs = []
    
    for pagina in range(pagina_inicio, pagina_inicio + max_paginas):
        url = f"{url_base}?{param_pagina}={pagina}"
        
        try:
            time.sleep(espera)
            response = requests.get(url, headers=headers, timeout=10)
            response.raise_for_status()
            tablas = pd.read_html(response.text)
            
            if not tablas:
                print(f"   Página {pagina}: sin tablas — fin de la paginación")
                break
            
            df_pagina = tablas[0]
            
            # Condición de corte: si la tabla viene vacía o con solo encabezado
            if len(df_pagina) <= 1:
                print(f"   Página {pagina}: tabla vacía — fin de la paginación")
                break
            
            todos_los_dfs.append(df_pagina)
            print(f"   Página {pagina}: {len(df_pagina)} filas ✅")
        
        except Exception as e:
            print(f"   Página {pagina}: error — {e}")
            break
    
    if todos_los_dfs:
        df_total = pd.concat(todos_los_dfs, ignore_index=True)
        df_total = df_total.drop_duplicates().reset_index(drop=True)
        print(f"\n✅ Total acumulado: {len(df_total)} filas en {len(todos_los_dfs)} páginas")
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

## 9. Limpieza post-scraping

Todo scraping requiere limpieza. Estos son los problemas más frecuentes:

```python
import pandas as pd

def limpiar_post_scraping(df: pd.DataFrame, col_id: str = None) -> pd.DataFrame:
    """
    Limpieza estándar después de cualquier scraping.
    """
    df = df.copy()
    
    # 1. Eliminar filas que son repetición del encabezado (común en read_html)
    for col in df.columns:
        df = df[df[col] != str(col)]
    
    # 2. Eliminar filas completamente vacías
    df = df.dropna(how='all')
    
    # 3. Eliminar columnas completamente vacías
    df = df.dropna(axis=1, how='all')
    
    # 4. Limpiar espacios en strings
    for col in df.select_dtypes(include='object').columns:
        df[col] = df[col].str.strip()
    
    # 5. Convertir columnas numéricas (vienen como string del HTML)
    for col in df.columns:
        if col != col_id:  # no convertir la columna ID
            df[col] = pd.to_numeric(df[col], errors='ignore')
    
    # 6. Resetear índice
    df = df.reset_index(drop=True)
    
    print(f"✅ Shape post-limpieza: {df.shape}")
    return df


def limpiar_headers_multinivel(df: pd.DataFrame) -> pd.DataFrame:
    """
    Aplana columnas con MultiIndex (frecuente en FBref y tablas con subencabezados).
    """
    if isinstance(df.columns, pd.MultiIndex):
        df.columns = ['_'.join(str(c) for c in col).strip('_ ') 
                      for col in df.columns]
        print("✅ MultiIndex aplanado")
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

## 10. Buenas prácticas y ética

### Respetar el servidor

```python
import time
import random

# ✅ Siempre esperar entre requests
time.sleep(2)

# ✅ Mejor: espera aleatoria (más natural, menos detectable)
time.sleep(random.uniform(1.5, 3.5))

# ✅ Para scraping masivo: respetar el robots.txt
# Verificar manualmente: https://ejemplo.com/robots.txt
# No scrapear rutas marcadas como Disallow
```

### Guardar el raw data siempre

```python
import os
from datetime import date

def guardar_raw(df: pd.DataFrame, nombre: str, carpeta: str = 'data/raw') -> None:
    """
    Guarda el DataFrame crudo antes de cualquier transformación.
    Incluye la fecha en el nombre para versionado básico.
    """
    os.makedirs(carpeta, exist_ok=True)
    fecha = date.today().strftime('%Y%m%d')
    ruta = f"{carpeta}/{nombre}_{fecha}.csv"
    df.to_csv(ruta, index=False)
    print(f"✅ Raw data guardado: {ruta}")

# Flujo recomendado:
# 1. Scrapear
df_raw = scrape_tabla_simple(url)
# 2. Guardar raw SIN modificar
guardar_raw(df_raw, "transfermarkt_huracan")
# 3. Limpiar sobre una copia
df_limpio = limpiar_post_scraping(df_raw.copy())
```

### Reglas generales

- Verificar el `robots.txt` del sitio antes de scrapear
- No hacer más de 1 request por segundo en sitios que no son APIs
- No scrapear datos personales sensibles sin propósito claro
- Si el sitio tiene API oficial → usarla siempre antes que scraping

---

## 11. Errores comunes y soluciones

| Error | Causa probable | Solución |
|-------|---------------|----------|
| `403 Forbidden` | El servidor bloqueó la request | Agregar headers completos de navegador |
| `429 Too Many Requests` | Demasiadas requests seguidas | Agregar `time.sleep()` entre requests |
| `ValueError: No tables found` | No hay tablas HTML en la página | El contenido es dinámico → usar Selenium |
| Tabla con índice incorrecto | La página tiene múltiples tablas | Usar `inspeccionar_tablas()` para explorar |
| Columnas con MultiIndex | FBref u otras fuentes con subencabezados | Usar `limpiar_headers_multinivel()` |
| Filas duplicadas por jugador | HTML tiene estructura visual mezclada con datos | Usar `limpiar_filas_duplicadas()` |
| Columnas numéricas como string | `read_html` no convierte automáticamente | `pd.to_numeric(col, errors='coerce')` |
| Columna `Nac.` vacía (banderas) | Las imágenes no se capturan con `read_html` | Usar BeautifulSoup para extraer el atributo `alt` de la imagen |
| Timeout | El servidor tarda mucho | Aumentar `timeout=` o usar Selenium con espera explícita |
| Datos desactualizados | Se guardó el raw de una sesión anterior | Documentar siempre la fecha de extracción |

---

## 12. Checklist de scraping

Antes de dar por finalizada una extracción:

### Diagnóstico previo
- [ ] Verificar si el contenido está en el HTML estático (Ctrl+U) o es dinámico
- [ ] Revisar `robots.txt` del sitio
- [ ] Buscar si existe una API oficial o API oculta (DevTools → Network → XHR)

### Extracción
- [ ] Usar headers con User-Agent siempre
- [ ] Agregar espera entre requests (`time.sleep`)
- [ ] Usar `inspeccionar_tablas()` antes de asumir un índice fijo
- [ ] Verificar que la tabla tiene los datos esperados (shape, columnas, primeras filas)

### Post-extracción
- [ ] Guardar el raw data sin modificar con fecha en el nombre
- [ ] Limpiar sobre una copia del DataFrame original
- [ ] Verificar columnas vacías (¿datos estáticos o dinámicos?)
- [ ] Documentar en el notebook: fuente, fecha, URL, índice de tabla usado

---

*Guía viva — agregar nuevos escenarios a medida que aparecen en proyectos reales.*
