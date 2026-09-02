# 🗄️ Plantilla SQL — Exploración de Dataset Nuevo

> **Uso:** Copiá cada bloque en tu cliente SQL al conectarte a una base nueva  
> **Orden:** Seguí las secciones de arriba hacia abajo  
> **Variable:** Reemplazá `NOMBRE_TABLA` con tu tabla real en cada query

---

## Tabla de Contenidos

1. [Reconocimiento de la base](#1-reconocimiento-de-la-base)
2. [Estructura de la tabla](#2-estructura-de-la-tabla)
3. [Diagnóstico de datos](#3-diagnóstico-de-datos)
4. [Variables numéricas](#4-variables-numéricas)
5. [Variables categóricas](#5-variables-categóricas)
6. [Relaciones entre tablas](#6-relaciones-entre-tablas)
7. [Detección de problemas de calidad](#7-detección-de-problemas-de-calidad)
8. [Serie temporal (si hay fecha)](#8-serie-temporal-si-hay-fecha)

---

## 1. Reconocimiento de la base

```sql
-- Ver todas las tablas de la base de datos actual
SHOW TABLES;

-- Ver en qué base estás trabajando
SELECT DATABASE();

-- Ver todas las bases disponibles
SHOW DATABASES;

-- Ver tamaño aproximado de cada tabla
SELECT table_name         AS tabla,
       table_rows         AS filas_aprox,
       ROUND((data_length + index_length) / 1024 / 1024, 2) AS tamanio_mb
FROM information_schema.tables
WHERE table_schema = DATABASE()
ORDER BY tamanio_mb DESC;
```

---

## 2. Estructura de la tabla

```sql
-- Ver columnas, tipos y si aceptan nulos
DESCRIBE NOMBRE_TABLA;                              -- ← reemplazar

-- Alternativa más detallada
SHOW COLUMNS FROM NOMBRE_TABLA;                     -- ← reemplazar

-- Ver cómo fue creada la tabla (constraints, índices, claves)
SHOW CREATE TABLE NOMBRE_TABLA;                     -- ← reemplazar

-- Ver índices y claves de la tabla
SHOW INDEX FROM NOMBRE_TABLA;                       -- ← reemplazar
```

---

## 3. Diagnóstico de datos

```sql
-- Primeras 10 filas — vista general del contenido real
SELECT *
FROM NOMBRE_TABLA                                   -- ← reemplazar
LIMIT 10;

-- Últimas 10 filas — detectar si hay datos más recientes
SELECT *
FROM NOMBRE_TABLA                                   -- ← reemplazar
ORDER BY 1 DESC
LIMIT 10;

-- Total de filas
SELECT COUNT(*) AS total_filas
FROM NOMBRE_TABLA;                                  -- ← reemplazar

-- Total de filas y columnas (resumen rápido)
SELECT COUNT(*) AS total_filas
FROM NOMBRE_TABLA;                                  -- ← reemplazar

-- Estadísticas descriptivas de columnas numéricas
-- Reemplazá COL_NUMERICA con cada columna que te interese
SELECT
    COUNT(COL_NUMERICA)                    AS total_no_nulos,
    COUNT(*) - COUNT(COL_NUMERICA)         AS total_nulos,
    ROUND(AVG(COL_NUMERICA), 2)            AS promedio,
    MIN(COL_NUMERICA)                      AS minimo,
    MAX(COL_NUMERICA)                      AS maximo,
    MAX(COL_NUMERICA) - MIN(COL_NUMERICA)  AS rango
FROM NOMBRE_TABLA;                                  -- ← reemplazar

-- Nulos por columna — hacerlo para cada columna relevante
SELECT
    SUM(CASE WHEN COL_1 IS NULL THEN 1 ELSE 0 END) AS nulos_col1,   -- ← reemplazar
    SUM(CASE WHEN COL_2 IS NULL THEN 1 ELSE 0 END) AS nulos_col2,   -- ← reemplazar
    SUM(CASE WHEN COL_3 IS NULL THEN 1 ELSE 0 END) AS nulos_col3,   -- ← reemplazar
    COUNT(*) AS total_filas
FROM NOMBRE_TABLA;                                  -- ← reemplazar

-- Filas duplicadas exactas
SELECT COUNT(*) AS duplicados
FROM (
    SELECT *, COUNT(*) AS repeticiones
    FROM NOMBRE_TABLA                               -- ← reemplazar
    GROUP BY COL_1, COL_2, COL_3                   -- ← reemplazar con todas las columnas
    HAVING COUNT(*) > 1
) AS dup;

-- Ver las filas duplicadas
SELECT *, COUNT(*) AS repeticiones
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY COL_1, COL_2, COL_3                        -- ← reemplazar
HAVING COUNT(*) > 1
ORDER BY repeticiones DESC;
```

---

## 4. Variables numéricas

```sql
-- Distribución por rangos (histograma manual)
SELECT
    CASE
        WHEN COL_NUMERICA < 100              THEN 'menos de 100'
        WHEN COL_NUMERICA BETWEEN 100 AND 500 THEN '100 - 500'
        WHEN COL_NUMERICA BETWEEN 500 AND 1000 THEN '500 - 1000'
        ELSE 'más de 1000'
    END AS rango,
    COUNT(*) AS cantidad,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM NOMBRE_TABLA), 2) AS porcentaje
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY rango
ORDER BY MIN(COL_NUMERICA);                         -- ← reemplazar

-- Percentiles aproximados (valores de corte)
SELECT
    MIN(COL_NUMERICA)                              AS p0_minimo,
    SUBSTRING_INDEX(
        SUBSTRING_INDEX(
            GROUP_CONCAT(COL_NUMERICA ORDER BY COL_NUMERICA), ',',
            CEIL(COUNT(*) * 0.25)
        ), ',', -1
    )                                              AS p25,
    SUBSTRING_INDEX(
        SUBSTRING_INDEX(
            GROUP_CONCAT(COL_NUMERICA ORDER BY COL_NUMERICA), ',',
            CEIL(COUNT(*) * 0.50)
        ), ',', -1
    )                                              AS p50_mediana,
    SUBSTRING_INDEX(
        SUBSTRING_INDEX(
            GROUP_CONCAT(COL_NUMERICA ORDER BY COL_NUMERICA), ',',
            CEIL(COUNT(*) * 0.75)
        ), ',', -1
    )                                              AS p75,
    MAX(COL_NUMERICA)                              AS p100_maximo
FROM NOMBRE_TABLA                                   -- ← reemplazar
WHERE COL_NUMERICA IS NOT NULL;                     -- ← reemplazar

-- Detección de outliers con IQR (valores fuera del rango normal)
WITH stats AS (
    SELECT
        SUBSTRING_INDEX(GROUP_CONCAT(COL_NUMERICA ORDER BY COL_NUMERICA), ',',
            CEIL(COUNT(*) * 0.25)), ',', -1) AS q1,
        SUBSTRING_INDEX(GROUP_CONCAT(COL_NUMERICA ORDER BY COL_NUMERICA), ',',
            CEIL(COUNT(*) * 0.75)), ',', -1) AS q3
    FROM NOMBRE_TABLA                               -- ← reemplazar
    WHERE COL_NUMERICA IS NOT NULL                  -- ← reemplazar
)
SELECT COUNT(*) AS posibles_outliers
FROM NOMBRE_TABLA, stats                            -- ← reemplazar
WHERE COL_NUMERICA < (q1 - 1.5 * (q3 - q1))        -- ← reemplazar
   OR COL_NUMERICA > (q3 + 1.5 * (q3 - q1));        -- ← reemplazar

-- Top 10 valores más altos
SELECT COL_NUMERICA, COUNT(*) AS frecuencia         -- ← reemplazar
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY COL_NUMERICA
ORDER BY COL_NUMERICA DESC
LIMIT 10;

-- Top 10 valores más bajos
SELECT COL_NUMERICA, COUNT(*) AS frecuencia         -- ← reemplazar
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY COL_NUMERICA
ORDER BY COL_NUMERICA ASC
LIMIT 10;
```

---

## 5. Variables categóricas

```sql
-- Valores únicos y su frecuencia (para una columna categórica)
SELECT COL_CATEGORICA,                              -- ← reemplazar
       COUNT(*) AS frecuencia,
       ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM NOMBRE_TABLA), 2) AS porcentaje
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY COL_CATEGORICA
ORDER BY frecuencia DESC;

-- Cantidad de valores únicos por columna
SELECT COUNT(DISTINCT COL_CATEGORICA) AS valores_unicos   -- ← reemplazar
FROM NOMBRE_TABLA;                                  -- ← reemplazar

-- Categorías con muy poca frecuencia (posibles errores de carga)
SELECT COL_CATEGORICA, COUNT(*) AS frecuencia       -- ← reemplazar
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY COL_CATEGORICA
HAVING COUNT(*) < 5
ORDER BY frecuencia ASC;

-- Detectar espacios extra o variantes del mismo valor
SELECT COL_CATEGORICA,                              -- ← reemplazar
       TRIM(COL_CATEGORICA) AS sin_espacios,        -- ← reemplazar
       LENGTH(COL_CATEGORICA) AS largo,             -- ← reemplazar
       COUNT(*) AS frecuencia
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY COL_CATEGORICA
ORDER BY largo DESC;

-- Comparar mayúsculas vs minúsculas (detectar 'Buenos Aires' vs 'buenos aires')
SELECT LOWER(COL_CATEGORICA) AS valor_normalizado,  -- ← reemplazar
       COUNT(*) AS variantes,
       GROUP_CONCAT(DISTINCT COL_CATEGORICA) AS formas_encontradas
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY LOWER(COL_CATEGORICA)                      -- ← reemplazar
HAVING COUNT(DISTINCT COL_CATEGORICA) > 1;
```

---

## 6. Relaciones entre tablas

```sql
-- Ver si una clave foránea tiene valores huérfanos
-- (registros en tabla_hija que no tienen par en tabla_padre)
SELECT COUNT(*) AS huerfanos
FROM TABLA_HIJA h                                   -- ← reemplazar
LEFT JOIN TABLA_PADRE p ON h.ID_FK = p.ID_PK        -- ← reemplazar
WHERE p.ID_PK IS NULL;                              -- ← reemplazar

-- Ver los registros huérfanos
SELECT h.*
FROM TABLA_HIJA h                                   -- ← reemplazar
LEFT JOIN TABLA_PADRE p ON h.ID_FK = p.ID_PK        -- ← reemplazar
WHERE p.ID_PK IS NULL;                              -- ← reemplazar

-- Ver registros de tabla_padre sin hijos (sin relación)
SELECT p.*
FROM TABLA_PADRE p                                  -- ← reemplazar
LEFT JOIN TABLA_HIJA h ON p.ID_PK = h.ID_FK         -- ← reemplazar
WHERE h.ID_FK IS NULL;                              -- ← reemplazar

-- Cardinalidad de la relación (cuántos hijos por padre en promedio)
SELECT
    COUNT(DISTINCT p.ID_PK)         AS total_padres,
    COUNT(h.ID_FK)                  AS total_hijos,
    ROUND(COUNT(h.ID_FK) /
          COUNT(DISTINCT p.ID_PK), 2) AS promedio_hijos_por_padre
FROM TABLA_PADRE p                                  -- ← reemplazar
LEFT JOIN TABLA_HIJA h ON p.ID_PK = h.ID_FK;        -- ← reemplazar
```

---

## 7. Detección de problemas de calidad

```sql
-- Filas con cualquier columna nula
SELECT *
FROM NOMBRE_TABLA                                   -- ← reemplazar
WHERE COL_1 IS NULL                                 -- ← reemplazar
   OR COL_2 IS NULL                                 -- ← reemplazar
   OR COL_3 IS NULL;                                -- ← reemplazar

-- Valores numéricos imposibles (negativos donde no debería haberlos)
SELECT COUNT(*) AS invalidos
FROM NOMBRE_TABLA                                   -- ← reemplazar
WHERE COL_NUMERICA < 0;                             -- ← reemplazar

-- Texto vacío vs NULL (son distintos en SQL)
SELECT
    SUM(CASE WHEN COL IS NULL THEN 1 ELSE 0 END)     AS nulos,
    SUM(CASE WHEN COL = '' THEN 1 ELSE 0 END)         AS vacios,
    SUM(CASE WHEN TRIM(COL) = '' THEN 1 ELSE 0 END)   AS solo_espacios
FROM NOMBRE_TABLA;                                  -- ← reemplazar

-- Fechas fuera de rango esperado
SELECT COUNT(*) AS fechas_invalidas
FROM NOMBRE_TABLA                                   -- ← reemplazar
WHERE COL_FECHA < '2000-01-01'                      -- ← ajustar rango esperado
   OR COL_FECHA > CURDATE();                        -- ← reemplazar

-- IDs duplicados (cuando deberían ser únicos)
SELECT COL_ID, COUNT(*) AS repeticiones             -- ← reemplazar
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY COL_ID
HAVING COUNT(*) > 1
ORDER BY repeticiones DESC;
```

---

## 8. Serie temporal (si hay fecha)

```sql
-- Rango de fechas del dataset
SELECT MIN(COL_FECHA) AS fecha_inicio,              -- ← reemplazar
       MAX(COL_FECHA) AS fecha_fin,
       DATEDIFF(MAX(COL_FECHA), MIN(COL_FECHA)) AS dias_cubiertos
FROM NOMBRE_TABLA;                                  -- ← reemplazar

-- Registros por año
SELECT YEAR(COL_FECHA) AS anio,                     -- ← reemplazar
       COUNT(*) AS cantidad
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY YEAR(COL_FECHA)
ORDER BY anio;

-- Registros por mes (para un año específico)
SELECT MONTH(COL_FECHA) AS mes,                     -- ← reemplazar
       COUNT(*) AS cantidad
FROM NOMBRE_TABLA                                   -- ← reemplazar
WHERE YEAR(COL_FECHA) = 2026                        -- ← ajustar año
GROUP BY MONTH(COL_FECHA)
ORDER BY mes;

-- Evolución de una métrica por mes
SELECT DATE_FORMAT(COL_FECHA, '%Y-%m') AS periodo,  -- ← reemplazar
       COUNT(*)                         AS registros,
       SUM(COL_NUMERICA)               AS suma,
       ROUND(AVG(COL_NUMERICA), 2)     AS promedio
FROM NOMBRE_TABLA                                   -- ← reemplazar
GROUP BY DATE_FORMAT(COL_FECHA, '%Y-%m')
ORDER BY periodo;

-- Detectar gaps (períodos sin datos)
SELECT periodo_esperado
FROM (
    SELECT DATE_FORMAT(
        DATE_ADD('2024-01-01', INTERVAL seq MONTH), '%Y-%m'
    ) AS periodo_esperado
    FROM (
        SELECT 0 AS seq UNION SELECT 1 UNION SELECT 2 UNION SELECT 3
        UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7
        UNION SELECT 8 UNION SELECT 9 UNION SELECT 10 UNION SELECT 11
    ) AS secuencia
) AS periodos_esperados
WHERE periodo_esperado NOT IN (
    SELECT DISTINCT DATE_FORMAT(COL_FECHA, '%Y-%m')  -- ← reemplazar
    FROM NOMBRE_TABLA                                -- ← reemplazar
);
```

---

> **Orden recomendado al explorar una base nueva:**
> 1. Ver qué tablas existen → `SHOW TABLES`
> 2. Ver estructura de cada tabla → `DESCRIBE`
> 3. Ver primeras y últimas filas → `SELECT * LIMIT 10`
> 4. Contar filas y nulos → diagnóstico general
> 5. Explorar numéricas → distribución, outliers
> 6. Explorar categóricas → frecuencias, inconsistencias
> 7. Verificar relaciones → huérfanos, cardinalidad
> 8. Explorar fechas → rango, evolución, gaps

---

*Para referencia completa de SQL → `sql_guia_completa.md`*
