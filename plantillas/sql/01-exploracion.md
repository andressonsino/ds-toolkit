# 🗺️ Guía de Exploración de una Base de Datos Nueva

> **Filosofía:** primero mirar el mapa (interfaz visual), después caminar el territorio (SQL)  
> **Objetivo:** obtener comprensión completa — estructura, relaciones, calidad y negocio  
> **Motor:** MySQL / phpMyAdmin (adaptable a cualquier cliente SQL)

---

## Tabla de Contenidos

1. [Fase 1 — El mapa: exploración visual](#1-fase-1--el-mapa-exploración-visual)
2. [Fase 2 — Reconocimiento general por SQL](#2-fase-2--reconocimiento-general-por-sql)
3. [Fase 3 — Explorar cada tabla en profundidad](#3-fase-3--explorar-cada-tabla-en-profundidad)
4. [Fase 4 — Entender las relaciones](#4-fase-4--entender-las-relaciones)
5. [Fase 5 — Calidad de los datos](#5-fase-5--calidad-de-los-datos)
6. [Fase 6 — Contexto de negocio](#6-fase-6--contexto-de-negocio)
7. [Checklist de cierre](#7-checklist-de-cierre)
8. [Plantilla de notas](#8-plantilla-de-notas)

---

## 1. Fase 1 — El mapa: exploración visual

> Hacer esto antes de escribir una sola query. La interfaz visual da el panorama estructural en segundos.

### En phpMyAdmin

**1.1 Vista general de tablas**

Al entrar a la base, la pantalla principal muestra:
- Lista de todas las tablas
- Cantidad de filas por tabla → identifica cuáles son las tablas principales (más filas = más actividad)
- Tamaño en KB → tablas grandes son candidatas a tener los datos más ricos
- Tipo de motor (InnoDB soporta claves foráneas; MyISAM no)

**Preguntas a responder en esta pantalla:**
- ¿Cuántas tablas tiene la base?
- ¿Hay tablas claramente más grandes que otras?
- ¿Los nombres sugieren qué contienen? (cliente, venta, producto, detalle...)
- ¿Hay tablas que parecen "puentes" entre otras? (suelen tener nombres compuestos: `cliente_producto`, `orden_detalle`)

---

**1.2 Vista de relaciones (Diseñador)**

En phpMyAdmin: pestaña **Diseñador** (o Designer)

Muestra todas las tablas con sus columnas y las líneas de relación entre ellas.

**Qué observar:**
- Qué tablas están en el centro del diagrama (muchas conexiones = tabla principal)
- La dirección de las flechas (de hijo → padre, donde el hijo tiene la FK)
- Tablas aisladas sin relaciones (posibles tablas de configuración o catálogos)
- Jerarquías: una tabla que conecta con muchas otras es probablemente la tabla de hechos

---

**1.3 Estructura individual de cada tabla**

Para cada tabla → pestaña **Estructura**:
- Columnas y sus tipos de dato
- Cuáles son PK y FK
- Cuáles aceptan NULL
- Si hay índices (columnas indexadas son frecuentemente usadas en JOINs o filtros)

---

**1.4 Muestra de datos**

Para cada tabla → pestaña **Examinar** (primeras filas):
- ¿Los datos tienen sentido?
- ¿Hay columnas con valores siempre vacíos?
- ¿Los IDs son numéricos, textos, o códigos?
- ¿Los formatos de fecha son consistentes?

---
## Fase 2 — Reconocimiento general en SQL

### 2.1 Ver todas las tablas de la base
**✅ Directo**
```sql
SHOW TABLES;
```

### 2.2 Confirmar en qué base estás parado
**✅ Directo**
```sql
SELECT DATABASE();
```

### 2.3 Resumen de todas las tablas: filas y tamaño
**✅ Directo**
```sql
SELECT
    table_name AS tabla,
    table_rows AS filas_aprox,
    ROUND((data_length + index_length) / 1024, 2) AS tamanio_kb,
    engine AS motor,
    table_comment AS comentario
FROM information_schema.tables
WHERE table_schema = DATABASE()
ORDER BY table_rows DESC;
```
*Con esto ya tenés el ranking de tablas por volumen. Las más grandes suelen ser las más importantes para el análisis.*

### 2.4 Ver columnas de todas las tablas de una vez
**✅ Directo**
```sql
SELECT
    table_name AS tabla,
    column_name AS columna,
    data_type AS tipo,
    is_nullable AS acepta_nulos,
    column_key AS clave
FROM information_schema.columns
WHERE table_schema = DATABASE()
ORDER BY table_name, ordinal_position;
```
*`column_key`: PRI = primaria, MUL = foránea, UNI = única.*

### 2.5 Buscar una columna específica en todas las tablas
**✏️ Requiere reemplazo:** `[NOMBRE_COLUMNA]`
```sql
SELECT table_name, column_name, data_type
FROM information_schema.columns
WHERE table_schema = DATABASE()
  AND column_name LIKE '%NOMBRE_COLUMNA%';
```

### 2.6 Ver todas las claves foráneas de la base
**✅ Directo** — la consulta más valiosa al principio: muestra el mapa completo de relaciones en texto.
```sql
SELECT
    kcu.table_name AS tabla_hija,
    kcu.column_name AS columna_fk,
    kcu.referenced_table_name AS tabla_padre,
    kcu.referenced_column_name AS columna_pk
FROM information_schema.key_column_usage kcu
WHERE kcu.constraint_schema = DATABASE()
  AND kcu.referenced_table_name IS NOT NULL
ORDER BY kcu.table_name;
```

---

## Fase 3 — Explorar cada tabla en profundidad

> Repetir este bloque para cada tabla importante identificada en la Fase 2. Todos requieren reemplazo de `[NOMBRE_TABLA]` y de las columnas correspondientes.

### 3.1 Estructura de la tabla
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`
```sql
DESCRIBE NOMBRE_TABLA;
```

### 3.2 Cómo fue creada (constraints, índices, FKs)
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`
```sql
SHOW CREATE TABLE NOMBRE_TABLA;
```

### 3.3 Primeras 10 filas
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`
```sql
SELECT * FROM NOMBRE_TABLA LIMIT 10;
```

### 3.4 Últimas 10 filas
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`
```sql
SELECT * FROM NOMBRE_TABLA ORDER BY 1 DESC LIMIT 10;
```

### 3.5 Total de filas
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`
```sql
SELECT COUNT(*) AS total_filas FROM NOMBRE_TABLA;
```

### 3.6 Cantidad de valores únicos por columna
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_1]`, `[COL_2]`, `[COL_3]`
```sql
SELECT
    COUNT(DISTINCT COL_1) AS unicos_col1,
    COUNT(DISTINCT COL_2) AS unicos_col2,
    COUNT(DISTINCT COL_3) AS unicos_col3,
    COUNT(*) AS total_filas
FROM NOMBRE_TABLA;
```
*Indica si una columna es categórica (pocos únicos) o identificadora (muchos únicos).*

### 3.7 Estadísticas de columnas numéricas
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_NUMERICA]`
```sql
SELECT
    COUNT(COL_NUMERICA) AS no_nulos,
    COUNT(*) - COUNT(COL_NUMERICA) AS nulos,
    ROUND(AVG(COL_NUMERICA), 2) AS promedio,
    MIN(COL_NUMERICA) AS minimo,
    MAX(COL_NUMERICA) AS maximo,
    MAX(COL_NUMERICA) - MIN(COL_NUMERICA) AS rango
FROM NOMBRE_TABLA;
```

### 3.8 Frecuencia de valores en columnas categóricas
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]` (dos veces), `[COL_CATEGORICA]`
```sql
SELECT
    COL_CATEGORICA,
    COUNT(*) AS frecuencia,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM NOMBRE_TABLA), 2) AS porcentaje
FROM NOMBRE_TABLA
GROUP BY COL_CATEGORICA
ORDER BY frecuencia DESC;
```

### 3.9 Rango de fechas
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_FECHA]`
```sql
SELECT
    MIN(COL_FECHA) AS fecha_mas_antigua,
    MAX(COL_FECHA) AS fecha_mas_reciente,
    DATEDIFF(MAX(COL_FECHA), MIN(COL_FECHA)) AS dias_cubiertos
FROM NOMBRE_TABLA;
```

---

## Fase 4 — Entender las relaciones

### 4.1 Ver los "hijos" de un registro específico
**✏️ Requiere reemplazo:** `[TABLA_PADRE]`, `[TABLA_HIJA]`, `[PK_PADRE]`, `[FK_HIJO]`, `[ID_ESPECIFICO]` (opcional)
```sql
SELECT
    padre.*,
    hijo.*
FROM TABLA_PADRE padre
JOIN TABLA_HIJA hijo
    ON padre.PK_PADRE = hijo.FK_HIJO
WHERE padre.PK_PADRE = ID_ESPECIFICO;
```
*Si querés ver todos los cruces sin filtrar por un ID, borrá la línea `WHERE`.*

### 4.2 Cardinalidad: cuántos hijos tiene cada padre
**✏️ Requiere reemplazo:** `[TABLA_PADRE]`, `[TABLA_HIJA]`, `[PK_PADRE]`, `[FK_HIJO]`
```sql
SELECT
    padre.PK_PADRE,
    COUNT(hijo.FK_HIJO) AS cantidad_hijos
FROM TABLA_PADRE padre
LEFT JOIN TABLA_HIJA hijo ON padre.PK_PADRE = hijo.FK_HIJO
GROUP BY padre.PK_PADRE
ORDER BY cantidad_hijos DESC
LIMIT 20;
```
*Responde: ¿un cliente tiene muchas ventas? ¿o solo una?*

### 4.3 Registros huérfanos: hijos sin padre
**✏️ Requiere reemplazo:** `[TABLA_HIJA]`, `[TABLA_PADRE]`, `[FK_HIJO]`, `[PK_PADRE]`
```sql
SELECT COUNT(*) AS huerfanos
FROM TABLA_HIJA hijo
LEFT JOIN TABLA_PADRE padre ON hijo.FK_HIJO = padre.PK_PADRE
WHERE padre.PK_PADRE IS NULL;
```
*Si aparecen, hay un problema de integridad referencial en la base.*

### 4.4 Padres sin hijos
**✏️ Requiere reemplazo:** `[TABLA_PADRE]`, `[TABLA_HIJA]`, `[PK_PADRE]`, `[FK_HIJO]`
```sql
SELECT padre.*
FROM TABLA_PADRE padre
LEFT JOIN TABLA_HIJA hijo ON padre.PK_PADRE = hijo.FK_HIJO
WHERE hijo.FK_HIJO IS NULL;
```
*Clientes que nunca compraron, productos que nunca se vendieron, etc.*

### 4.5 Explorar tabla puente (relación muchos a muchos)
**✏️ Requiere reemplazo:** `[TABLA_PUENTE]`, `[TABLA_A]`, `[TABLA_B]`, `[FK_A]`, `[FK_B]`, `[PK_A]`, `[PK_B]`, `[COL_TABLA_A]`, `[COL_TABLA_B]`
```sql
SELECT
    a.COL_TABLA_A,
    b.COL_TABLA_B,
    puente.*
FROM TABLA_PUENTE puente
JOIN TABLA_A a ON puente.FK_A = a.PK_A
JOIN TABLA_B b ON puente.FK_B = b.PK_B
LIMIT 20;
```
*Las tablas puente conectan dos entidades. Ej: un alumno puede tener muchos cursos y un curso puede tener muchos alumnos → la tabla `alumno_curso` es el puente.*

---

## Fase 5 — Calidad de los datos

### 5.1 Nulos por columna
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_1]`, `[COL_2]`, `[COL_3]`
```sql
SELECT
    SUM(CASE WHEN COL_1 IS NULL THEN 1 ELSE 0 END) AS nulos_col1,
    SUM(CASE WHEN COL_2 IS NULL THEN 1 ELSE 0 END) AS nulos_col2,
    SUM(CASE WHEN COL_3 IS NULL THEN 1 ELSE 0 END) AS nulos_col3,
    COUNT(*) AS total_filas
FROM NOMBRE_TABLA;
```

### 5.2 Filas duplicadas exactas
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_1]`, `[COL_2]`, `[COL_3]` (agregar todas las columnas de la tabla)
```sql
SELECT *, COUNT(*) AS repeticiones
FROM NOMBRE_TABLA
GROUP BY COL_1, COL_2, COL_3
HAVING COUNT(*) > 1
ORDER BY repeticiones DESC;
```

### 5.3 Texto vacío vs NULL (son distintos en SQL)
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL]`
```sql
SELECT
    SUM(CASE WHEN COL IS NULL THEN 1 ELSE 0 END) AS nulos,
    SUM(CASE WHEN COL = '' THEN 1 ELSE 0 END) AS vacios,
    SUM(CASE WHEN TRIM(COL) = '' THEN 1 ELSE 0 END) AS solo_espacios
FROM NOMBRE_TABLA;
```

### 5.4 Valores numéricos imposibles
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_NUMERICA]` (y ajustar la condición si hace falta)
```sql
SELECT COUNT(*) AS invalidos
FROM NOMBRE_TABLA
WHERE COL_NUMERICA < 0;
```

### 5.5 IDs que deberían ser únicos pero no lo son
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_ID]`
```sql
SELECT COL_ID, COUNT(*) AS repeticiones
FROM NOMBRE_TABLA
GROUP BY COL_ID
HAVING COUNT(*) > 1
ORDER BY repeticiones DESC;
```

### 5.6 Inconsistencias en texto (mayúsculas, espacios, variantes)
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_TEXTO]`
```sql
SELECT
    LOWER(TRIM(COL_TEXTO)) AS valor_normalizado,
    COUNT(DISTINCT COL_TEXTO) AS variantes,
    GROUP_CONCAT(DISTINCT COL_TEXTO) AS formas_encontradas
FROM NOMBRE_TABLA
GROUP BY LOWER(TRIM(COL_TEXTO))
HAVING COUNT(DISTINCT COL_TEXTO) > 1;
```
*Detecta si "Buenos Aires" y "buenos aires" conviven como valores distintos.*

### 5.7 Fechas fuera de rango esperado
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_FECHA]` (y ajustar el límite inferior)
```sql
SELECT COUNT(*) AS fechas_invalidas
FROM NOMBRE_TABLA
WHERE COL_FECHA < '2000-01-01'
   OR COL_FECHA > CURDATE();
```

---

## Fase 6 — Contexto de negocio

### 6.1 Actividad general: registros por período
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_FECHA]`
```sql
SELECT
    YEAR(COL_FECHA) AS anio,
    MONTH(COL_FECHA) AS mes,
    COUNT(*) AS cantidad
FROM NOMBRE_TABLA
GROUP BY anio, mes
ORDER BY anio, mes;
```

### 6.2 Top 10 registros más relevantes (por una métrica)
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]`, `[COL_METRICA]`
```sql
SELECT *
FROM NOMBRE_TABLA
ORDER BY COL_METRICA DESC
LIMIT 10;
```

### 6.3 Distribución de una métrica clave
**✏️ Requiere reemplazo:** `[NOMBRE_TABLA]` (dos veces), `[COL_METRICA]` (dos veces), `[VALOR_1]`, `[VALOR_2]`
```sql
SELECT
    CASE
        WHEN COL_METRICA < VALOR_1 THEN 'Bajo'
        WHEN COL_METRICA BETWEEN VALOR_1 AND VALOR_2 THEN 'Medio'
        ELSE 'Alto'
    END AS segmento,
    COUNT(*) AS cantidad,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM NOMBRE_TABLA), 2) AS porcentaje
FROM NOMBRE_TABLA
GROUP BY segmento
ORDER BY MIN(COL_METRICA);
```

### 6.4 Cruzar dos tablas para ver el negocio completo
**✏️ Requiere reemplazo:** `[TABLA_PADRE]`, `[TABLA_HIJA]`, `[PK_PADRE]`, `[FK_HIJO]`, `[COL_NOMBRE]`, `[COL_METRICA]`
```sql
SELECT
    padre.COL_NOMBRE,
    COUNT(hijo.FK_HIJO) AS cantidad_registros,
    SUM(hijo.COL_METRICA) AS total,
    ROUND(AVG(hijo.COL_METRICA), 2) AS promedio
FROM TABLA_PADRE padre
LEFT JOIN TABLA_HIJA hijo ON padre.PK_PADRE = hijo.FK_HIJO
GROUP BY padre.PK_PADRE, padre.COL_NOMBRE
ORDER BY total DESC;
```
*Ejemplo: clientes con su cantidad de compras y monto total.*

---

## 7. Checklist de cierre

Marcá cada punto al terminar la exploración:

**Estructura**
- [ ] Sé cuántas tablas tiene la base y cuál es su propósito general
- [ ] Identifiqué las tablas principales (más filas, más conexiones)
- [ ] Entendí qué tablas son puentes (relaciones muchos a muchos)
- [ ] Mapeé todas las relaciones PK → FK

**Datos**
- [ ] Revisé nulos en columnas clave de las tablas principales
- [ ] Verifiqué que no haya registros huérfanos
- [ ] Identifiqué el rango de fechas del dataset
- [ ] Detecté posibles inconsistencias en columnas de texto

**Negocio**
- [ ] Entendí qué proceso de negocio registra cada tabla principal
- [ ] Identifiqué la métrica clave del negocio (monto, cantidad, duración...)
- [ ] Sé qué preguntas de negocio puede responder esta base

---

## 8. Plantilla de notas

> Completar a mano mientras se explora. Guardarlo en `_notas/exploracion_[nombre_base].md`

```
## Base de datos: [NOMBRE]
Fecha de exploración: [DD/MM/AAAA]

## Resumen
- Total de tablas: 
- Motor: 
- Rango temporal de los datos: 

## Tablas principales
| Tabla | Filas aprox. | Propósito |
|---|---|---|
| | | |

## Mapa de relaciones
[Describir en texto o dibujar las conexiones principales]
Ej: cliente (1) → (N) venta → (N) detalle_venta → (1) producto

## Columnas clave identificadas
- Tabla X: la métrica principal es [columna]
- Tabla Y: el identificador de negocio es [columna]

## Problemas de calidad encontrados
- 
- 

## Preguntas de negocio que puede responder
- 
- 

## Preguntas que quedaron sin responder
- 
- 
```

---

> **Regla de oro:** no empezar a analizar datos sin antes entender la estructura.  
> Cinco minutos explorando el mapa evitan horas de confusión en el territorio.

---

*Para queries de exploración inicial más rápidas → `plantilla_sql_exploracion.md`  
Para referencia completa de SQL → `sql_guia_completa.md`*
