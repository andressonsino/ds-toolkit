# 📐 Plantilla Mermaid — Modelos Dimensionales

> **Uso:** Copiá el bloque de código en [draw.io](https://app.diagrams.net/) → Extras → Edit Diagram → seleccioná Mermaid  
> O pegalo directamente en [mermaid.live](https://mermaid.live) para previsualizar  
> **Para editar:** reemplazá los nombres de tablas, columnas y relaciones. No toques la sintaxis de corchetes ni los estilos.

---

## Modelo Estrella

**Cuándo usarlo:** siempre que sea posible. Una tabla de hechos central conectada directamente a cada dimensión. Sin conexiones entre dimensiones.

```mermaid
erDiagram

    %% ── TABLA DE HECHOS ──────────────────────────────────────────────────────
    FACT_VENTAS {
        int     id_venta        PK
        int     id_fecha        FK
        int     id_cliente      FK
        int     id_producto     FK
        int     id_sucursal     FK
        int     cantidad
        decimal precio_unitario
        decimal descuento
        decimal monto_total
    }

    %% ── DIMENSIONES ──────────────────────────────────────────────────────────
    DIM_FECHA {
        int     id_fecha        PK
        date    fecha
        int     dia
        int     mes
        int     trimestre
        int     anio
        string  dia_semana
    }

    DIM_CLIENTE {
        int     id_cliente      PK
        string  nombre_completo
        string  email
        string  ciudad
        string  provincia
        string  segmento
    }

    DIM_PRODUCTO {
        int     id_producto     PK
        string  nombre_producto
        string  categoria
        string  subcategoria
        string  marca
        decimal precio_lista
    }

    DIM_SUCURSAL {
        int     id_sucursal     PK
        string  nombre
        string  ciudad
        string  zona
        int     metros_cuadrados
    }

    %% ── RELACIONES (tabla_padre ||--o{ tabla_hija : "descripcion") ───────────
    %% Sintaxis: ||--o{  =  uno a muchos obligatorio
    %%           ||--||  =  uno a uno
    %%           }o--o{  =  muchos a muchos

    DIM_FECHA     ||--o{ FACT_VENTAS : "tiene"
    DIM_CLIENTE   ||--o{ FACT_VENTAS : "realiza"
    DIM_PRODUCTO  ||--o{ FACT_VENTAS : "incluye"
    DIM_SUCURSAL  ||--o{ FACT_VENTAS : "genera"
```

---

## Modelo Copo de Nieve

**Cuándo usarlo:** solo cuando una dimensión es tan grande que degrada el rendimiento. Las dimensiones se normalizan en subdimensiones conectadas entre sí.

```mermaid
erDiagram

    %% ── TABLA DE HECHOS (igual que en estrella) ─────────────────────────────
    FACT_VENTAS {
        int     id_venta        PK
        int     id_fecha        FK
        int     id_cliente      FK
        int     id_producto     FK
        int     id_sucursal     FK
        int     cantidad
        decimal precio_unitario
        decimal descuento
        decimal monto_total
    }

    %% ── DIMENSIONES DIRECTAS ─────────────────────────────────────────────────
    DIM_FECHA {
        int     id_fecha        PK
        int     id_semana       FK
        date    fecha
        int     dia
        int     mes
        int     trimestre
        int     anio
    }

    DIM_CLIENTE {
        int     id_cliente      PK
        int     id_ciudad       FK
        string  nombre_completo
        string  email
        string  segmento
    }

    DIM_PRODUCTO {
        int     id_producto     PK
        int     id_subcategoria FK
        int     id_marca        FK
        string  nombre_producto
        decimal precio_lista
    }

    DIM_SUCURSAL {
        int     id_sucursal     PK
        int     id_zona         FK
        string  nombre
        int     metros_cuadrados
    }

    %% ── SUBDIMENSIONES (dimensiones anidadas — conectadas entre sí) ──────────
    DIM_SEMANA {
        int     id_semana       PK
        int     numero_semana
        int     anio
    }

    DIM_CIUDAD {
        int     id_ciudad       PK
        int     id_provincia    FK
        string  nombre_ciudad
    }

    DIM_PROVINCIA {
        int     id_provincia    PK
        string  nombre_provincia
        string  region
    }

    DIM_SUBCATEGORIA {
        int     id_subcategoria PK
        int     id_categoria    FK
        string  nombre_subcategoria
    }

    DIM_CATEGORIA {
        int     id_categoria    PK
        string  nombre_categoria
        string  departamento
    }

    DIM_MARCA {
        int     id_marca        PK
        string  nombre_marca
        string  pais_origen
    }

    DIM_ZONA {
        int     id_zona         PK
        string  nombre_zona
        string  region
    }

    %% ── RELACIONES — TABLA DE HECHOS → DIMENSIONES DIRECTAS ─────────────────
    DIM_FECHA     ||--o{ FACT_VENTAS : "tiene"
    DIM_CLIENTE   ||--o{ FACT_VENTAS : "realiza"
    DIM_PRODUCTO  ||--o{ FACT_VENTAS : "incluye"
    DIM_SUCURSAL  ||--o{ FACT_VENTAS : "genera"

    %% ── RELACIONES — DIMENSIONES → SUBDIMENSIONES (el "copo") ───────────────
    DIM_SEMANA       ||--o{ DIM_FECHA       : "agrupa"
    DIM_CIUDAD       ||--o{ DIM_CLIENTE     : "ubica"
    DIM_PROVINCIA    ||--o{ DIM_CIUDAD      : "contiene"
    DIM_SUBCATEGORIA ||--o{ DIM_PRODUCTO    : "clasifica"
    DIM_CATEGORIA    ||--o{ DIM_SUBCATEGORIA : "agrupa"
    DIM_MARCA        ||--o{ DIM_PRODUCTO    : "identifica"
    DIM_ZONA         ||--o{ DIM_SUCURSAL    : "pertenece"
```

---

## Referencia rápida — Sintaxis Mermaid ER

```
%%  Comentario (no se renderiza)

Tipos de dato válidos:
    int     string     decimal     date     bool

Tipos de clave:
    PK      FK         UK  (unique key)

Relaciones:
    ||--||   uno a uno obligatorio
    ||--o|   uno a uno opcional
    ||--o{   uno a muchos (el más común en modelos dimensionales)
    }o--o{   muchos a muchos

Estructura de relación:
    TABLA_A  TIPO_RELACION  TABLA_B : "etiqueta"
    Ejemplo:
    DIM_FECHA  ||--o{  FACT_VENTAS : "tiene"
```

---

## Cómo adaptar estas plantillas a tu proyecto

1. **Renombrá las tablas** — cambiá `FACT_VENTAS`, `DIM_CLIENTE`, etc. por los nombres reales
2. **Editá las columnas** — agregá, quitá o renombrá columnas dentro de cada bloque `{}`
3. **Ajustá las relaciones** — solo modificá los nombres de tabla y la etiqueta entre comillas
4. **En copo de nieve** — agregá o quitá subdimensiones según cuántos niveles tenga tu jerarquía
5. **No toques** — la sintaxis `||--o{`, los `%%`, ni la estructura de bloques

---

*Previsualización online: [mermaid.live](https://mermaid.live)*  
*Draw.io: Extras → Edit Diagram → seleccionar tipo Mermaid*
