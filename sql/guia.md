# 🗄️ Guía Completa de SQL para Ciencia de Datos

> **Motor:** MySQL / MariaDB (compatible con PostgreSQL salvo excepciones indicadas)  
> **Nivel:** Básico → Avanzado  
> **Uso:** Referencia y plantilla de consultas reutilizables

---

## Tabla de Contenidos

1. [Conceptos clave](#1-conceptos-clave)
2. [Consultas simples](#2-consultas-simples)
3. [Filtros con WHERE](#3-filtros-con-where)
4. [Ordenar y limitar resultados](#4-ordenar-y-limitar-resultados)
5. [Funciones de agregación](#5-funciones-de-agregación)
6. [GROUP BY y HAVING](#6-group-by-y-having)
7. [JOIN — combinar tablas](#7-join--combinar-tablas)
8. [Subconsultas (subqueries)](#8-subconsultas-subqueries)
9. [Manipulación de datos](#9-manipulación-de-datos)
10. [Crear y modificar tablas](#10-crear-y-modificar-tablas)
11. [Funciones de texto y fecha](#11-funciones-de-texto-y-fecha)
12. [Patrones avanzados para data science](#12-patrones-avanzados-para-data-science)
13. [Errores comunes](#13-errores-comunes)
14. [Cheat sheet](#14-cheat-sheet)

---

## 1. Conceptos clave

**¿Qué es SQL?**
Lenguaje de consulta estructurada para interactuar con bases de datos relacionales. Los datos se organizan en tablas con filas (registros) y columnas (campos).

**Tipos de sentencias:**

| Tipo | Qué hace | Ejemplos |
|------|----------|---------|
| DQL | Consultar datos | `SELECT` |
| DML | Manipular datos | `INSERT`, `UPDATE`, `DELETE` |
| DDL | Definir estructura | `CREATE`, `ALTER`, `DROP` |
| DCL | Control de acceso | `GRANT`, `REVOKE` |

**Orden de ejecución de una consulta SQL:**
```
1. FROM / JOIN    → de dónde vienen los datos
2. WHERE          → filtra filas
3. GROUP BY       → agrupa filas
4. HAVING         → filtra grupos
5. SELECT         → elige columnas
6. ORDER BY       → ordena resultado
7. LIMIT          → limita cantidad de filas
```

> ⚠️ El orden en que SQL **ejecuta** las cláusulas es diferente al orden en que se **escriben**.

---

## 2. Consultas simples

```sql
-- Traer todas las columnas de una tabla
SELECT * FROM articulo;

-- Traer columnas específicas
SELECT cod_art, descripcion, precio
FROM articulo;

-- Renombrar columnas en el resultado (alias)
SELECT cod_art AS codigo,
       descripcion AS nombre,
       precio AS precio_unitario
FROM articulo;

-- Eliminar duplicados
SELECT DISTINCT ciudad
FROM proveedor;

-- Contar total de filas
SELECT COUNT(*) AS total_articulos
FROM articulo;
```

---

## 3. Filtros con WHERE

```sql
-- Igual
SELECT * FROM proveedor WHERE ciudad = 'Buenos Aires';

-- Distinto
SELECT * FROM articulo WHERE precio <> 0;

-- Mayor / menor
SELECT descripcion, precio FROM articulo WHERE precio > 100;
SELECT descripcion, precio FROM articulo WHERE precio <= 50;

-- Rango inclusivo
SELECT descripcion, precio
FROM articulo
WHERE precio BETWEEN 50 AND 200;

-- Lista de valores
SELECT * FROM proveedor
WHERE ciudad IN ('Buenos Aires', 'Rosario', 'Córdoba');

-- Texto que contiene (LIKE)
SELECT * FROM articulo WHERE descripcion LIKE '%mesa%';   -- contiene "mesa"
SELECT * FROM articulo WHERE descripcion LIKE 'sill%';    -- empieza con "sill"
SELECT * FROM articulo WHERE descripcion LIKE '%a';       -- termina en "a"

-- Nulos
SELECT * FROM proveedor WHERE domicilio IS NULL;
SELECT * FROM proveedor WHERE domicilio IS NOT NULL;

-- Múltiples condiciones
SELECT descripcion, precio
FROM articulo
WHERE precio > 100 AND precio < 500;

SELECT * FROM proveedor
WHERE ciudad = 'Buenos Aires' OR ciudad = 'Rosario';

-- Negar una condición
SELECT * FROM articulo WHERE NOT precio > 500;
```

> **LIKE — wildcards:**
> - `%` → cualquier cantidad de caracteres
> - `_` → exactamente un caracter

---

## 4. Ordenar y limitar resultados

```sql
-- Ordenar ascendente (por defecto)
SELECT nombre, ciudad FROM proveedor ORDER BY ciudad ASC;

-- Ordenar descendente
SELECT descripcion, precio FROM articulo ORDER BY precio DESC;

-- Ordenar por múltiples columnas
SELECT descripcion, precio
FROM articulo
ORDER BY precio DESC, descripcion ASC;

-- Limitar cantidad de filas
SELECT descripcion, precio
FROM articulo
ORDER BY precio DESC
LIMIT 5;                    -- los 5 más caros

-- Paginación (offset)
SELECT * FROM articulo
ORDER BY cod_art
LIMIT 10 OFFSET 20;         -- filas 21 a 30
```

---

## 5. Funciones de agregación

```sql
-- Contar filas
SELECT COUNT(*) AS total FROM articulo;

-- Contar valores no nulos de una columna
SELECT COUNT(precio) AS con_precio FROM articulo;

-- Suma
SELECT SUM(precio) AS precio_total FROM articulo;

-- Promedio
SELECT AVG(precio) AS precio_promedio FROM articulo;

-- Máximo y mínimo
SELECT MAX(precio) AS mas_caro,
       MIN(precio) AS mas_barato
FROM articulo;

-- Todas juntas
SELECT COUNT(*)    AS total,
       AVG(precio) AS promedio,
       MAX(precio) AS maximo,
       MIN(precio) AS minimo,
       SUM(precio) AS suma_total
FROM articulo;
```

> Las funciones de agregación **ignoran los NULL** — excepto `COUNT(*)` que cuenta todas las filas.

---

## 6. GROUP BY y HAVING

```sql
-- Contar artículos por almacén
SELECT nro_alm, COUNT(*) AS cantidad_articulos
FROM tiene
GROUP BY nro_alm;

-- Precio promedio por categoría (si existiera la columna)
SELECT categoria, AVG(precio) AS precio_promedio
FROM articulo
GROUP BY categoria
ORDER BY precio_promedio DESC;

-- HAVING — filtrar después de agrupar
-- (WHERE no funciona con funciones de agregación)
SELECT nro_alm, COUNT(*) AS cantidad
FROM tiene
GROUP BY nro_alm
HAVING COUNT(*) > 3;        -- solo almacenes con más de 3 artículos

-- Combinando WHERE y HAVING
SELECT nro_alm, COUNT(*) AS cantidad
FROM tiene
INNER JOIN articulo ON tiene.cod_art = articulo.cod_art
WHERE articulo.precio > 50  -- filtra filas ANTES de agrupar
GROUP BY nro_alm
HAVING COUNT(*) >= 2;       -- filtra grupos DESPUÉS de agrupar
```

> **WHERE vs HAVING:**
> - `WHERE` → filtra filas individuales, antes de agrupar
> - `HAVING` → filtra grupos, después de `GROUP BY`

---

## 7. JOIN — combinar tablas

```sql
-- INNER JOIN — solo filas con coincidencia en ambas tablas
SELECT a.nro_alm, a.responsable, ar.descripcion, ar.precio
FROM almacen a
INNER JOIN tiene t    ON a.nro_alm = t.nro_alm
INNER JOIN articulo ar ON t.cod_art = ar.cod_art;

-- LEFT JOIN — todas las filas de la tabla izquierda
-- aunque no tengan par en la derecha (NULL donde no hay coincidencia)
SELECT a.nro_alm, a.responsable, ar.descripcion
FROM almacen a
LEFT JOIN tiene t     ON a.nro_alm = t.nro_alm
LEFT JOIN articulo ar ON t.cod_art = ar.cod_art;

-- RIGHT JOIN — todas las filas de la tabla derecha
SELECT a.nro_alm, ar.descripcion
FROM almacen a
RIGHT JOIN tiene t ON a.nro_alm = t.nro_alm;

-- Encontrar filas SIN par (usando LEFT JOIN + IS NULL)
SELECT a.nro_alm, a.responsable
FROM almacen a
LEFT JOIN tiene t ON a.nro_alm = t.nro_alm
WHERE t.nro_alm IS NULL;    -- almacenes sin ningún artículo

-- JOIN con alias y filtro
SELECT a.responsable, ar.descripcion, ar.precio
FROM almacen a
INNER JOIN tiene t     ON a.nro_alm = t.nro_alm
INNER JOIN articulo ar ON t.cod_art = ar.cod_art
WHERE ar.precio > 100
ORDER BY ar.precio DESC;
```

**Tipos de JOIN:**

```
INNER JOIN    →  A ∩ B   (solo los que coinciden)
LEFT JOIN     →  A       (todos de A, con o sin par en B)
RIGHT JOIN    →  B       (todos de B, con o sin par en A)
FULL JOIN     →  A ∪ B   (todos de ambos — no disponible en MySQL, se simula)
```

---

## 8. Subconsultas (subqueries)

```sql
-- Subquery que devuelve UN valor (usar con =)
-- Artículo más caro
SELECT descripcion, precio
FROM articulo
WHERE precio = (SELECT MAX(precio) FROM articulo);

-- Artículos con precio mayor al promedio
SELECT descripcion, precio
FROM articulo
WHERE precio > (SELECT AVG(precio) FROM articulo)
ORDER BY precio DESC;

-- Subquery que devuelve VARIOS valores (usar con IN)
-- Almacenes que tienen algún artículo caro
SELECT nro_alm, responsable
FROM almacen
WHERE nro_alm IN (
    SELECT t.nro_alm
    FROM tiene t
    INNER JOIN articulo ar ON t.cod_art = ar.cod_art
    WHERE ar.precio > 200
);

-- Subquery en FROM (tabla derivada)
-- Promedio de cantidades por almacén
SELECT AVG(cantidad) AS promedio_por_almacen
FROM (
    SELECT nro_alm, COUNT(*) AS cantidad
    FROM tiene
    GROUP BY nro_alm
) AS conteo_por_almacen;

-- Subquery correlacionada (referencia a la query exterior)
-- Artículos cuyo precio supera el promedio de su categoría
SELECT descripcion, precio
FROM articulo a1
WHERE precio > (
    SELECT AVG(precio)
    FROM articulo a2
    WHERE a2.cod_art = a1.cod_art
);
```

> **Subquery vs JOIN:**
> - Las subqueries son más legibles para lógica compleja
> - Los JOINs son generalmente más eficientes en performance
> - Para datasets grandes, preferir JOIN

---

## 9. Manipulación de datos

```sql
-- INSERT — insertar una fila
INSERT INTO articulo (descripcion, precio)
VALUES ('Silla de madera', 4500.00);

-- INSERT — insertar varias filas
INSERT INTO articulo (descripcion, precio)
VALUES
    ('Mesa ratona', 8900.00),
    ('Estante', 3200.00),
    ('Sillón', 12500.00);

-- UPDATE — modificar datos
UPDATE articulo
SET precio = 5000.00
WHERE cod_art = 1;

-- UPDATE con cálculo
UPDATE articulo
SET precio = precio * 1.10     -- aumentar 10% a todos
WHERE precio < 1000;

-- DELETE — eliminar filas
DELETE FROM articulo
WHERE cod_art = 5;

-- DELETE con condición múltiple
DELETE FROM articulo
WHERE precio < 100 AND descripcion LIKE '%roto%';
```

> ⚠️ **Siempre usá WHERE en UPDATE y DELETE.** Sin WHERE, afecta TODOS los registros de la tabla.

---

## 10. Crear y modificar tablas

```sql
-- Crear tabla
CREATE TABLE producto (
    id_producto   INT AUTO_INCREMENT PRIMARY KEY,
    nombre        VARCHAR(100) NOT NULL,
    precio        DECIMAL(10, 2) DEFAULT 0.00,
    categoria     VARCHAR(50),
    fecha_alta    DATE
);

-- Agregar columna
ALTER TABLE articulo ADD COLUMN stock INT DEFAULT 0;

-- Modificar tipo de columna
ALTER TABLE articulo MODIFY precio DECIMAL(12, 2);

-- Renombrar columna
ALTER TABLE articulo RENAME COLUMN descripcion TO nombre;

-- Eliminar columna
ALTER TABLE articulo DROP COLUMN stock;

-- Eliminar tabla (irreversible)
DROP TABLE IF EXISTS tabla_temporal;

-- Vaciar tabla (borra datos, mantiene estructura)
TRUNCATE TABLE tabla_temporal;

-- Ver estructura de una tabla
DESCRIBE articulo;
SHOW COLUMNS FROM articulo;
```

---

## 11. Funciones de texto y fecha

```sql
-- Texto
SELECT UPPER(descripcion) FROM articulo;         -- MAYÚSCULAS
SELECT LOWER(nombre) FROM proveedor;             -- minúsculas
SELECT LENGTH(descripcion) FROM articulo;        -- largo del texto
SELECT TRIM(nombre) FROM proveedor;              -- quita espacios
SELECT CONCAT(nombre, ' - ', ciudad) AS info
FROM proveedor;                                  -- concatenar
SELECT SUBSTRING(descripcion, 1, 20) FROM articulo;  -- primeros 20 chars
SELECT REPLACE(descripcion, 'viejo', 'nuevo') FROM articulo;

-- Números
SELECT ROUND(precio, 2) FROM articulo;           -- redondear
SELECT CEIL(precio) FROM articulo;               -- redondear arriba
SELECT FLOOR(precio) FROM articulo;              -- redondear abajo
SELECT ABS(precio) FROM articulo;                -- valor absoluto

-- Fechas (si hay columna de fecha)
SELECT NOW();                                    -- fecha y hora actual
SELECT CURDATE();                                -- fecha actual
SELECT YEAR(fecha_alta) FROM producto;           -- extraer año
SELECT MONTH(fecha_alta) FROM producto;          -- extraer mes
SELECT DAY(fecha_alta) FROM producto;            -- extraer día
SELECT DATEDIFF(NOW(), fecha_alta) AS dias_desde_alta
FROM producto;                                   -- diferencia en días

-- Nulos
SELECT IFNULL(domicilio, 'Sin domicilio') FROM proveedor;
SELECT COALESCE(domicilio, ciudad, 'Sin dato') FROM proveedor;
```

---

## 12. Patrones avanzados para data science

### Tabla pivote manual

```sql
-- Contar artículos por rango de precio
SELECT
    SUM(CASE WHEN precio < 100 THEN 1 ELSE 0 END)              AS menos_100,
    SUM(CASE WHEN precio BETWEEN 100 AND 500 THEN 1 ELSE 0 END) AS entre_100_500,
    SUM(CASE WHEN precio > 500 THEN 1 ELSE 0 END)              AS mas_500
FROM articulo;
```

### CASE WHEN — columna condicional

```sql
SELECT descripcion,
       precio,
       CASE
           WHEN precio < 100  THEN 'Económico'
           WHEN precio < 500  THEN 'Medio'
           WHEN precio < 1000 THEN 'Premium'
           ELSE 'Lujo'
       END AS categoria_precio
FROM articulo
ORDER BY precio;
```

### Window functions (MySQL 8+)

```sql
-- Ranking de artículos por precio
SELECT descripcion,
       precio,
       RANK() OVER (ORDER BY precio DESC) AS ranking
FROM articulo;

-- Precio acumulado
SELECT descripcion,
       precio,
       SUM(precio) OVER (ORDER BY cod_art) AS precio_acumulado
FROM articulo;

-- Comparar con el anterior
SELECT descripcion,
       precio,
       LAG(precio) OVER (ORDER BY cod_art)  AS precio_anterior,
       precio - LAG(precio) OVER (ORDER BY cod_art) AS diferencia
FROM articulo;
```

### CTEs (Common Table Expressions)

```sql
-- Más legible que subqueries anidadas
WITH articulos_caros AS (
    SELECT cod_art, descripcion, precio
    FROM articulo
    WHERE precio > 500
),
almacenes_con_caros AS (
    SELECT t.nro_alm, COUNT(*) AS cantidad
    FROM tiene t
    INNER JOIN articulos_caros ac ON t.cod_art = ac.cod_art
    GROUP BY t.nro_alm
)
SELECT a.responsable, ac.cantidad
FROM almacen a
INNER JOIN almacenes_con_caros ac ON a.nro_alm = ac.nro_alm
ORDER BY ac.cantidad DESC;
```

### Exportar resultado a CSV (desde consola MySQL)

```sql
SELECT *
FROM articulo
INTO OUTFILE '/tmp/articulos.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

---

## 13. Errores comunes

### ❌ WHERE con función de agregación

```sql
-- Incorrecto
SELECT nro_alm, COUNT(*) FROM tiene WHERE COUNT(*) > 3 GROUP BY nro_alm;

-- Correcto
SELECT nro_alm, COUNT(*) FROM tiene GROUP BY nro_alm HAVING COUNT(*) > 3;
```

### ❌ UPDATE o DELETE sin WHERE

```sql
-- Incorrecto — borra TODOS los registros
DELETE FROM articulo;

-- Correcto
DELETE FROM articulo WHERE cod_art = 5;
```

### ❌ Subquery con IN devuelve NULL

```sql
-- Si la subquery puede devolver NULL, IN no funciona bien
-- Usar IS NOT NULL en la subquery
SELECT * FROM articulo
WHERE cod_art IN (
    SELECT cod_art FROM tiene WHERE nro_alm IS NOT NULL
);
```

### ❌ Ambigüedad de columnas en JOIN

```sql
-- Incorrecto — MySQL no sabe de qué tabla es cod_art
SELECT cod_art FROM tiene INNER JOIN articulo ON tiene.cod_art = articulo.cod_art;

-- Correcto — especificar la tabla
SELECT t.cod_art FROM tiene t INNER JOIN articulo ar ON t.cod_art = ar.cod_art;
```

---

## 14. Cheat sheet

```sql
-- Consulta básica
SELECT col1, col2 FROM tabla WHERE condicion ORDER BY col1 DESC LIMIT 10;

-- Agregación
SELECT col, COUNT(*), AVG(num), SUM(num), MAX(num), MIN(num)
FROM tabla GROUP BY col HAVING COUNT(*) > 1;

-- JOIN
SELECT a.col, b.col
FROM tabla_a a
INNER JOIN tabla_b b ON a.id = b.id_fk;

-- Subquery con valor único
SELECT * FROM tabla WHERE col = (SELECT MAX(col) FROM tabla);

-- Subquery con múltiples valores
SELECT * FROM tabla WHERE col IN (SELECT col FROM otra_tabla WHERE condicion);

-- CASE WHEN
SELECT col, CASE WHEN col > 100 THEN 'Alto' ELSE 'Bajo' END AS categoria FROM tabla;

-- INSERT
INSERT INTO tabla (col1, col2) VALUES ('valor1', 100);

-- UPDATE
UPDATE tabla SET col1 = 'nuevo' WHERE id = 1;

-- DELETE
DELETE FROM tabla WHERE id = 1;

-- Crear tabla
CREATE TABLE nombre (id INT AUTO_INCREMENT PRIMARY KEY, col VARCHAR(50) NOT NULL);
```

---

*Documentación oficial: [MySQL Docs](https://dev.mysql.com/doc/) · [PostgreSQL Docs](https://www.postgresql.org/docs/)*
