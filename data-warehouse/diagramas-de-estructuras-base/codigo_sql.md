# Plantillas de Modelado Dimensional (Draw.io)

Este documento contiene scripts SQL estandarizados para generar automáticamente diagramas Entidad-Relación (ER) de Data Warehouses en **Draw.io**.

**Instrucciones de uso en Draw.io:**
1. Crear un diagrama en blanco.
2. Ir a `Organizar` > `Insertar` > `Avanzado` > `SQL`.
3. Pegar el bloque de código deseado y presionar `Insertar`.

---

## 🌟 1. Esquema en Estrella (Star Schema)
**Recomendado para el 90% de los proyectos analíticos.** 
Prioriza la velocidad de lectura desnormalizando los datos (aceptando redundancia en las dimensiones). La tabla de Hechos está en el centro y se conecta a 1 solo paso de distancia de cada Dimensión.

```sql
CREATE TABLE Dim_Tiempo (
  id_tiempo INT PRIMARY KEY,
  fecha DATE,
  anio INT,
  mes INT,
  nombre_mes VARCHAR(20),
  trimestre INT
);

CREATE TABLE Dim_Inmueble (
  id_inmueble INT PRIMARY KEY,
  nro_partida VARCHAR(20),
  direccion VARCHAR(100),
  localidad VARCHAR(50),
  provincia VARCHAR(50),
  tipo_inmueble VARCHAR(30)
);

CREATE TABLE Dim_Moneda (
  id_moneda INT PRIMARY KEY,
  codigo_iso CHAR(3),
  nombre_moneda VARCHAR(30)
);

CREATE TABLE Dim_Proveedor (
  id_proveedor INT PRIMARY KEY,
  cuit VARCHAR(15),
  razon_social VARCHAR(50),
  rubro VARCHAR(30)
);

CREATE TABLE Dim_Contrato (
  id_contrato INT PRIMARY KEY,
  inquilino_nombre VARCHAR(50),
  propietario_nombre VARCHAR(50),
  estado_contrato VARCHAR(20)
);

CREATE TABLE Fact_Movimientos (
  id_movimiento INT PRIMARY KEY,
  id_tiempo INT,
  id_inmueble INT,
  id_moneda INT,
  id_proveedor INT,
  id_contrato INT,
  tipo_movimiento VARCHAR(30),
  monto_ingreso DECIMAL(12,2),
  monto_egreso DECIMAL(12,2),
  comision_inmobiliaria DECIMAL(12,2),
  FOREIGN KEY (id_tiempo) REFERENCES Dim_Tiempo(id_tiempo),
  FOREIGN KEY (id_inmueble) REFERENCES Dim_Inmueble(id_inmueble),
  FOREIGN KEY (id_moneda) REFERENCES Dim_Moneda(id_moneda),
  FOREIGN KEY (id_proveedor) REFERENCES Dim_Proveedor(id_proveedor),
  FOREIGN KEY (id_contrato) REFERENCES Dim_Contrato(id_contrato)
);
```

---

## ❄️ 2. Esquema en Copo de Nieve (Snowflake Schema)
**Variante Normalizada.** 
Busca ahorrar espacio de almacenamiento y evitar redundancias dividiendo (normalizando) las dimensiones en sub-dimensiones. 
*Nota: Es más lento para consultas BI debido a la mayor cantidad de JOINs.*

```sql
/* -- SUB-DIMENSIONES NORMALIZADAS -- */
CREATE TABLE Dim_Localidad (
  id_localidad INT PRIMARY KEY,
  nombre_localidad VARCHAR(50),
  provincia VARCHAR(50)
);

CREATE TABLE Dim_Rubro (
  id_rubro INT PRIMARY KEY,
  nombre_rubro VARCHAR(30)
);

/* -- DIMENSIONES PRINCIPALES -- */
CREATE TABLE Dim_Tiempo (
  id_tiempo INT PRIMARY KEY,
  fecha DATE,
  anio INT,
  mes INT,
  nombre_mes VARCHAR(20),
  trimestre INT
);

CREATE TABLE Dim_Moneda (
  id_moneda INT PRIMARY KEY,
  codigo_iso CHAR(3),
  nombre_moneda VARCHAR(30)
);

CREATE TABLE Dim_Contrato (
  id_contrato INT PRIMARY KEY,
  inquilino_nombre VARCHAR(50),
  propietario_nombre VARCHAR(50),
  estado_contrato VARCHAR(20)
);

/* -- DIMENSIONES CON CLAVES FORANEAS A LAS SUB-DIMENSIONES -- */
CREATE TABLE Dim_Inmueble (
  id_inmueble INT PRIMARY KEY,
  nro_partida VARCHAR(20),
  direccion VARCHAR(100),
  id_localidad INT,
  tipo_inmueble VARCHAR(30),
  FOREIGN KEY (id_localidad) REFERENCES Dim_Localidad(id_localidad)
);

CREATE TABLE Dim_Proveedor (
  id_proveedor INT PRIMARY KEY,
  cuit VARCHAR(15),
  razon_social VARCHAR(50),
  id_rubro INT,
  FOREIGN KEY (id_rubro) REFERENCES Dim_Rubro(id_rubro)
);

/* -- TABLA DE HECHOS CENTRAL -- */
CREATE TABLE Fact_Movimientos (
  id_movimiento INT PRIMARY KEY,
  id_tiempo INT,
  id_inmueble INT,
  id_moneda INT,
  id_proveedor INT,
  id_contrato INT,
  tipo_movimiento VARCHAR(30),
  monto_ingreso DECIMAL(12,2),
  monto_egreso DECIMAL(12,2),
  comision_inmobiliaria DECIMAL(12,2),
  FOREIGN KEY (id_tiempo) REFERENCES Dim_Tiempo(id_tiempo),
  FOREIGN KEY (id_inmueble) REFERENCES Dim_Inmueble(id_inmueble),
  FOREIGN KEY (id_moneda) REFERENCES Dim_Moneda(id_moneda),
  FOREIGN KEY (id_proveedor) REFERENCES Dim_Proveedor(id_proveedor),
  FOREIGN KEY (id_contrato) REFERENCES Dim_Contrato(id_contrato)
);
```