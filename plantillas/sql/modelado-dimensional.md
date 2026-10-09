# Guía de modelado dimensional — de básico a avanzado

> Guía genérica y reutilizable. Los ejemplos usan un caso de transporte (`costo`, `cantidad`, `peso_total`, `peso_unidad`) solo para ilustrar — el método se aplica igual a ventas, compras, RRHH, inventario, o cualquier otro dominio.

---

## 1. Conceptos base

### 1.1 ¿Qué es un modelo dimensional?
Es una forma de organizar datos pensada para **analizar y reportar**, no para operar día a día (eso lo hace el modelo transaccional / OLTP, como el `compras.sql` con el que veníamos trabajando). Se compone de dos tipos de tablas:

- **Tabla de hechos (fact table):** registra eventos o transacciones — un viaje, una venta, una compra. Contiene las **métricas** (números que se miden) y claves que apuntan a las dimensiones.
- **Tablas de dimensión:** describen el **contexto** del hecho — quién, cuándo, dónde, qué producto. Contienen atributos descriptivos (texto, categorías, fechas), casi nunca números que se sumen.

**Regla práctica para distinguir una de otra:** si la pregunta es "¿cuánto?" (cuánto costó, cuántas unidades, cuánto pesó) → va en la tabla de hechos. Si la pregunta es "¿quién/cuándo/dónde/qué tipo?" → va en una dimensión.

### 1.2 El proceso de diseño (4 pasos clásicos de Kimball)
1. **Elegir el proceso de negocio** a modelar (ej: viajes de transporte, ventas, compras).
2. **Declarar el grano** — qué representa exactamente una fila de la tabla de hechos (ej: "un viaje individual de un camión", no "un día de viajes" ni "un cliente"). Este es el paso que más errores de diseño evita: si no tenés claro el grano, no sabés si una métrica es aditiva o no.
3. **Identificar las dimensiones** — todo lo que responde quién/cuándo/dónde/qué, al nivel de detalle del grano elegido.
4. **Identificar las métricas (hechos)** — los números que se van a medir, y clasificarlos según cómo se pueden agregar. Acá es donde entra la parte que no tenías clara, así que le dedicamos la sección siguiente completa.

---

## 2. Clasificar métricas: aditivas, semi-aditivas, no aditivas

Esta clasificación determina **qué función de agregación (`SUM`, `AVG`, `MAX`...) tiene sentido matemático usar**, y por qué dimensión podés sumar sin obtener un número mentiroso.

### 2.1 Métricas aditivas
**Definición:** se pueden sumar por *cualquier* dimensión — tiempo, cliente, producto, vehículo, lo que sea — y el resultado sigue teniendo sentido de negocio.

**Cómo reconocerlas:** preguntate "si sumo esta columna para dos filas distintas, ¿el resultado significa algo real?". Si la respuesta es sí sin importar qué dos filas elijas, es aditiva.

**Ejemplos genéricos:** cantidad vendida, importe de una venta, peso transportado, horas trabajadas, unidades producidas. En tu caso: `costo`, `cantidad`, `peso_total`.

```sql
-- Sumar una métrica aditiva por distintas dimensiones: todas son válidas
SELECT SUM(costo) AS costo_total FROM hechos_viaje;                          -- total general
SELECT id_cliente, SUM(costo) AS costo_por_cliente FROM hechos_viaje GROUP BY id_cliente;
SELECT id_camion, SUM(peso_total) AS peso_por_camion FROM hechos_viaje GROUP BY id_camion;
SELECT YEAR(fecha), SUM(costo) AS costo_anual FROM hechos_viaje GROUP BY YEAR(fecha);
```

### 2.2 Métricas semi-aditivas
**Definición:** se pueden sumar por *algunas* dimensiones, pero **no por tiempo**. Son típicas de "fotos" de un estado en un momento dado (saldos, inventarios, niveles).

**Cómo reconocerlas:** preguntate "¿esta métrica representa una *cantidad acumulada en un momento* o un *flujo que ocurre durante un período*?". Si es una foto de un instante (saldo de cuenta, stock en depósito, temperatura), sumar varios instantes de tiempo da un número sin sentido — sumar el saldo de enero + el de febrero no te da "el saldo de los dos meses", te da un número inventado.

**Ejemplos genéricos:** saldo de cuenta bancaria, nivel de stock/inventario, cantidad de empleados activos a fin de mes, temperatura registrada.

```sql
-- Sumar por cliente SÍ tiene sentido (saldo total entre varios clientes en el mismo instante)
SELECT SUM(saldo) AS saldo_total_hoy
FROM hechos_saldo
WHERE fecha = CURDATE();

-- Sumar por tiempo NO tiene sentido — en vez de SUM, usás el último valor o un promedio
SELECT fecha, AVG(saldo) AS saldo_promedio_mensual
FROM hechos_saldo
GROUP BY fecha;

-- Para "saldo a fin de período", se suele tomar el último registro, no sumar todos
SELECT id_cuenta, saldo
FROM hechos_saldo
WHERE fecha = (SELECT MAX(fecha) FROM hechos_saldo h2 WHERE h2.id_cuenta = hechos_saldo.id_cuenta);
```

**Caso de tu TP:** no tenías métricas semi-aditivas en el modelo de transporte, y eso está bien — no todas las tablas de hechos tienen las tres categorías. Aclarar explícitamente "no aplican métricas semi-aditivas en este modelo" en tu entrega es correcto y demuestra que entendiste la clasificación, no que te faltó completarla.

### 2.3 Métricas no aditivas
**Definición:** **nunca** se suman, sin importar la dimensión. Son razones, promedios, porcentajes o valores unitarios — sumar dos de estos valores no produce ningún número con significado de negocio.

**Cómo reconocerlas:** preguntate "¿esta columna ya es el resultado de una división o un promedio?". Precio unitario, porcentaje, ratio, promedio — todo lo que termina en "por unidad", "promedio de", "tasa de", "%" es no aditivo.

**Ejemplos genéricos:** precio unitario, porcentaje de descuento, calificación/rating, margen de ganancia (%), velocidad promedio.

```sql
-- INCORRECTO: sumar precio_unitario no significa nada
SELECT SUM(peso_unidad) AS esto_no_sirve FROM hechos_viaje;  -- ⚠️ no hacer esto

-- CORRECTO: agregar con AVG, MIN o MAX según lo que quieras responder
SELECT AVG(peso_unidad) AS peso_promedio_por_bobina FROM hechos_viaje;
SELECT MIN(peso_unidad) AS bobina_mas_liviana, MAX(peso_unidad) AS bobina_mas_pesada FROM hechos_viaje;

-- Si necesitás un total, hay que derivarlo multiplicando por la cantidad (eso sí es aditivo)
SELECT SUM(peso_unidad * cantidad) AS peso_total_calculado FROM hechos_viaje;
```

**Truco general:** una métrica no aditiva casi siempre se puede "convertir" en aditiva si la descomponés en sus partes. `precio_unitario` no se suma, pero `precio_unitario * cantidad` (el importe total) sí. Por eso en los modelos reales suele convenir guardar tanto el valor unitario (para análisis de precio) como el importe ya calculado (para sumar sin pensar).

### 2.4 Tabla resumen para decidir rápido

| Pregunta que te hacés sobre la columna | Clasificación | Función típica |
|---|---|---|
| ¿Es una cantidad que ocurre en cada evento (venta, viaje, compra)? | Aditiva | `SUM` |
| ¿Es una "foto" de un estado en un instante (saldo, stock)? | Semi-aditiva | `SUM` por todo menos tiempo; `AVG`/último valor por tiempo |
| ¿Ya es un promedio, ratio, porcentaje o valor "por unidad"? | No aditiva | `AVG`, `MIN`, `MAX` |

---

## 3. Elegir dimensiones

### 3.1 Patrón clásico: dimensiones de quién/cuándo/dónde/qué
Para la mayoría de los procesos de negocio, las dimensiones salen de responder:
- **Quién:** cliente, empleado, proveedor, chofer.
- **Cuándo:** fecha (casi siempre merece su propia tabla `dim_tiempo`, ver 3.3).
- **Dónde:** sucursal, depósito, región, ruta.
- **Qué:** producto, servicio, categoría.

### 3.2 Cómo detectar si algo es dimensión o métrica cuando dudás
Si la columna tiene **pocos valores distintos que se repiten** (categorías, estados, tipos) → dimensión. Si tiene **muchos valores distintos y numéricos que varían por fila** → métrica. Podés verificarlo con una consulta:

```sql
-- Si el resultado es un número chico y estable → probablemente dimensión
-- Si es un número grande, casi igual al total de filas → probablemente métrica/identificador
SELECT COUNT(DISTINCT nombre_columna) AS valores_unicos, COUNT(*) AS total_filas
FROM tabla_origen;
```

### 3.3 Dimensión Tiempo (la más reutilizable de todas)
Casi todo modelo dimensional tiene una tabla de fechas separada, para poder agrupar por año/mes/día/trimestre/día de semana sin recalcular todo el tiempo con funciones.

```sql
-- Estructura típica de una dim_tiempo
CREATE TABLE dim_tiempo (
    id_fecha INT PRIMARY KEY,         -- formato AAAAMMDD, ej: 20260615
    fecha DATE NOT NULL,
    anio INT NOT NULL,
    trimestre INT NOT NULL,
    mes INT NOT NULL,
    nombre_mes VARCHAR(20) NOT NULL,
    dia INT NOT NULL,
    dia_semana VARCHAR(20) NOT NULL,
    es_fin_de_semana BOOLEAN NOT NULL
);
```

---

## 4. Armar la tabla de hechos

### 4.1 Estructura típica
```sql
CREATE TABLE hechos_proceso (
    id_hecho INT PRIMARY KEY AUTO_INCREMENT,
    -- Claves foráneas a cada dimensión (el "quién/cuándo/dónde/qué")
    id_fecha INT NOT NULL,
    id_cliente INT NOT NULL,
    id_producto INT NOT NULL,
    -- Métricas aditivas (se suman libremente)
    cantidad INT NOT NULL,
    importe DECIMAL(12,2) NOT NULL,
    -- Métricas no aditivas (se promedian o se toma min/max, nunca se suman)
    precio_unitario DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (id_fecha) REFERENCES dim_tiempo(id_fecha),
    FOREIGN KEY (id_cliente) REFERENCES dim_cliente(id_cliente),
    FOREIGN KEY (id_producto) REFERENCES dim_producto(id_producto)
);
```

### 4.2 Grano: por qué es el paso más importante
Si el grano es "un viaje", cada fila es un viaje y `cantidad`/`peso_total` representan ese viaje puntual. Si por error mezclás filas con distinto grano (algunas por viaje, otras ya agregadas por día), **ninguna métrica es confiablemente aditiva** porque estarías sumando cosas de naturaleza distinta. Antes de escribir una sola consulta de métricas, respondé por escrito: *"una fila de mi tabla de hechos representa ___"*. Si no podés completar esa frase con una sola unidad de negocio clara, el modelo todavía no está listo.

---

## 5. Consultas de validación (de básico a avanzado)

### 5.1 Validar que las métricas aditivas realmente sumen bien
```sql
-- El total general debe coincidir con la suma de los totales parciales
SELECT SUM(importe) AS total_general FROM hechos_proceso;

SELECT id_cliente, SUM(importe) AS total_cliente
FROM hechos_proceso
GROUP BY id_cliente;
-- La suma de todos los total_cliente de arriba debe dar lo mismo que total_general
```

### 5.2 Detectar si una métrica fue mal clasificada como aditiva
```sql
-- Si esta consulta da un número absurdo (mucho mayor al rango real de precios), 
-- es señal de que estás sumando algo que debería promediarse
SELECT SUM(precio_unitario) AS suma_rara, AVG(precio_unitario) AS promedio_correcto
FROM hechos_proceso;
```

### 5.3 Construir una métrica derivada a partir de una no aditiva
```sql
-- Convertir peso_unidad (no aditiva) en un total aditivo
SELECT
    id_cliente,
    SUM(peso_unidad * cantidad) AS peso_total_transportado,  -- aditiva, derivada
    AVG(peso_unidad) AS peso_promedio_por_bobina              -- no aditiva, se mantiene como promedio
FROM hechos_proceso
GROUP BY id_cliente;
```

### 5.4 KPI combinando aditivas y no aditivas correctamente
```sql
-- Ejemplo: costo total (aditiva) y costo promedio por viaje (derivada de aditiva / cantidad de filas)
SELECT
    id_camion,
    COUNT(*) AS cantidad_viajes,
    SUM(costo) AS costo_total,                          -- aditiva: se suma sin problema
    ROUND(SUM(costo) / COUNT(*), 2) AS costo_promedio_por_viaje,  -- derivada: NO es lo mismo que AVG(costo) si hay NULLs
    AVG(peso_unidad) AS peso_promedio_bobina             -- no aditiva: solo se promedia
FROM hechos_proceso
GROUP BY id_camion
ORDER BY costo_total DESC;
```

### 5.5 Semi-aditiva: valor "a fin de período" (patrón avanzado)
```sql
-- Último saldo conocido de cada cuenta hasta una fecha de corte
SELECT h1.id_cuenta, h1.saldo, h1.fecha
FROM hechos_saldo h1
WHERE h1.fecha = (
    SELECT MAX(h2.fecha)
    FROM hechos_saldo h2
    WHERE h2.id_cuenta = h1.id_cuenta
      AND h2.fecha <= '2026-12-31'  -- fecha de corte deseada
);
```

---

## 6. Checklist para aplicar a cualquier TP nuevo

1. **Identificá el proceso de negocio** que te piden modelar (una frase: "estoy modelando ___").
2. **Definí el grano** de la tabla de hechos (una frase: "cada fila es ___").
3. **Listá las dimensiones** respondiendo quién/cuándo/dónde/qué, al nivel del grano.
4. **Listá todas las columnas numéricas candidatas a métrica** y para cada una preguntate:
   - ¿Tiene sentido sumarla por cualquier dimensión? → **Aditiva**.
   - ¿Es una foto de un estado que no se debe sumar en el tiempo? → **Semi-aditiva**.
   - ¿Ya es un promedio, ratio o valor unitario? → **No aditiva**.
5. Para cada métrica no aditiva, preguntate si se puede **derivar una versión aditiva** multiplicándola por otra columna (precio unitario × cantidad = importe).
6. Si alguna categoría (normalmente la semi-aditiva) no aplica a tu caso, **decilo explícitamente** en la entrega — es parte correcta del análisis, no un hueco.
