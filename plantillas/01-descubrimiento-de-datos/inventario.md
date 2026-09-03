# 01 · Descubrimiento de Datos

Primer paso obligatorio antes de escribir cualquier línea de código.  
El objetivo es entender qué datos existen, dónde están, quién los mantiene y cómo se relacionan entre sí.

> **Regla de oro:** no integrés lo que no entendés. Un inventario completo evita el doble de trabajo después.

---

## ¿Qué es el descubrimiento de datos?

Es un proceso manual de relevamiento: abrís cada fuente de datos, la revisás, y documentás lo que encontrás. No requiere código. En la industria se llama **data discovery** y es el primer entregable que un analista produce antes de cualquier análisis.

Un buen descubrimiento de datos responde tres preguntas:
1. **¿Qué tengo?** — inventario de fuentes
2. **¿Cómo se relaciona?** — claves comunes entre fuentes
3. **¿Qué problemas hay?** — anomalías detectadas a simple vista

---

## Paso 1 — Inventario de Fuentes

Completar una fila por cada fuente de datos relevada.  
Hacerlo en orden: empezar por la fuente de verdad (la más confiable) y seguir por las secundarias.

### Plantilla de inventario

| # | Nombre del archivo/sheet | Tipo | Contenido | Responsable | Frecuencia actualización | Clave identificadora | Filas aprox. | Columnas aprox. | Se superpone con | Observaciones |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | | | | | | | | | | |
| 2 | | | | | | | | | | |
| 3 | | | | | | | | | | |

**Glosario de columnas:**

- **Tipo**: Google Sheets / Excel / CSV / SQL / API / PDF
- **Contenido**: descripción breve de qué información contiene (alumnos, pagos, asistencia, etc.)
- **Responsable**: quién actualiza este archivo (nombre, área, o "automático")
- **Frecuencia**: con qué regularidad se actualiza (diaria, mensual, manual sin frecuencia fija)
- **Clave identificadora**: columna que identifica unívocamente cada fila (DNI, legajo, ID, nombre)
- **Se superpone con**: qué otras fuentes contienen información similar o relacionada
- **Observaciones**: cualquier anomalía visible (celdas mergeadas, totales mezclados, fechas como texto, etc.)

---

## Paso 2 — Mapa de Relaciones

Una vez completado el inventario, identificar cómo se conectan las fuentes entre sí.

### Plantilla de relaciones

| Fuente A | Fuente B | Clave común | Tipo de relación | Conflictos potenciales |
|---|---|---|---|---|
| | | | | |

**Tipos de relación:**
- **1 a 1**: cada registro de A corresponde a exactamente uno de B
- **1 a N**: un registro de A corresponde a varios de B (ej: un alumno tiene varios pagos)
- **N a N**: varios registros de A corresponden a varios de B (requiere tabla intermedia)

---

## Paso 3 — Registro de Anomalías

Documentar los problemas detectados a simple vista durante el relevamiento.  
No resolverlos acá — solo registrarlos para tenerlos en cuenta en la limpieza.

### Plantilla de anomalías

| # | Fuente | Columna afectada | Tipo de problema | Ejemplo | Impacto estimado |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

**Tipos de problema comunes:**
- Celdas mergeadas que dificultan la lectura
- Filas de totales mezcladas con datos
- Fechas escritas como texto (01/03/24 vs 1 de marzo de 2024 vs 2024-03-01)
- El mismo registro escrito de formas distintas (Juan López / LOPEZ JUAN / j.lopez)
- Columnas con nombres distintos en distintos archivos para la misma información
- Archivos con estructura distinta por año o período
- Datos faltantes sin criterio claro

---

## Paso 4 — Definición de la Fuente de Verdad

Para cada tipo de información, definir cuál es la fuente más confiable cuando hay conflicto.

### Plantilla de fuentes de verdad

| Tipo de dato | Fuente de verdad | Justificación | Fuentes secundarias |
|---|---|---|---|
| | | | |

> **Criterio general:** la fuente de verdad es la más automatizada, la menos manipulada manualmente, y la que tiene trazabilidad de cambios.

---

## Paso 5 — Checklist de Cierre

Antes de pasar al siguiente paso (integración técnica), verificar:

- [ ] Todas las fuentes relevantes están inventariadas
- [ ] Cada fuente tiene identificada su clave única
- [ ] Las relaciones entre fuentes están mapeadas
- [ ] Las anomalías están registradas
- [ ] La fuente de verdad está definida para cada tipo de dato
- [ ] El inventario fue revisado con alguien del área (si aplica)

---

## Notas del proyecto actual

> Completar acá las particularidades del proyecto específico que distinguen este relevamiento de otros.

**Cliente / organización:**  
**Fecha de relevamiento:**  
**Analista:**  
**Contexto:**  

---

## Referencia rápida — señales de alerta durante el relevamiento

Al revisar cada archivo, prestar atención a:

- Más de una fila de encabezado
- Columnas sin nombre o con nombre genérico (Columna1, Unnamed)
- Mezcla de idiomas en los nombres de columnas
- Archivos con pestañas múltiples donde los datos están repartidos
- Fórmulas que dependen de otros archivos que no tenés
- Filtros activos que ocultan filas (verificar siempre que no haya datos ocultos)
- Archivos "versión final", "versión final 2", "versión final USAR ESTE"

---

*Metodología de integración de datos — ds-toolkit by andressonsino*  
*Parte de: `01-descubrimiento-de-datos/`*
