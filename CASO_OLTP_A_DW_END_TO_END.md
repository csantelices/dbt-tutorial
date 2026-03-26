# Caso End-to-End: OLTP → Data Warehouse con dbt (SQL Server)

## Objetivo del caso

Diseñar e implementar **un flujo completo OLTP → Data Warehouse** usando:

- **OLTP relacional** (AdventureWorks OLTP)
- **Data Warehouse dimensional** (modelo Kimball)
- **dbt sobre SQL Server** como framework de transformación

El foco está en **arquitectura, decisiones de modelado y buenas prácticas**, no solo en mover datos.

Perfil objetivo:
- DBA / Data Engineer / Arquitecto DW
- Dominio previo de SQL

---

## 1. Arquitectura general end-to-end

```
OLTP (AdventureWorks)
  ├── Sales
  ├── Person
  ├── Production
  ↓  (Extract & Load)
STAGING (raw / landing)
  ↓
dbt (Transform)
  ├── STAGING MODELS (stg_*)
  ├── DIMENSIONS (dim_*)
  └── FACTS (fct_*)
  ↓
DATA MART / DW
```

Principio rector:
> **El OLTP modela procesos técnicos, el DW modela el negocio**.

---

## 2. Origen OLTP

### Base de datos
- **AdventureWorks2019** (OLTP)

### Tablas relevantes (ventas)

| Área | Tabla |
|----|------|
| Ventas | Sales.SalesOrderHeader |
| Ventas | Sales.SalesOrderDetail |
| Cliente | Sales.Customer |
| Cliente | Person.Person |
| Producto | Production.Product |
| Producto | Production.ProductSubcategory |
| Producto | Production.ProductCategory |
| Fecha | derivada de OrderDate |

Problemas típicos OLTP:
- Modelo altamente normalizado
- Lógica técnica
- Cambios destructivos
- Difícil análisis

---

## 3. Proceso de negocio seleccionado

### Proceso
**Ventas de productos**

### Preguntas de negocio
- ¿Ventas por año?
- ¿Ventas por producto?
- ¿Ventas por cliente?
- ¿Evolución temporal?

👉 Estas preguntas definen el **modelo dimensional**.

---

## 4. Diseño Data Warehouse

### 4.1 Granularidad

> **1 fila = 1 línea de pedido**

📌 Decisión clave: se hereda la granularidad del `SalesOrderDetail`.

---

### 4.2 Tabla FACT

**FCT_SALES**

Medidas:
- SalesAmount
- OrderQuantity

Claves:
- DateKey
- ProductKey
- CustomerKey

---

### 4.3 Dimensiones

| Dimensión | Fuente OLTP |
|---------|-----------|
| DIM_DATE | OrderDate |
| DIM_PRODUCT | Production.Product + subcategory + category |
| DIM_CUSTOMER | Sales.Customer + Person.Person |

Todas con:
- Surrogate Key
- Atributos descriptivos

---

## 5. Estrategia de carga OLTP → STAGING

### Enfoque

- Carga **tal cual** desde OLTP
- Sin lógica de negocio
- Fácil de auditar

Ejemplo:

```sql
SELECT *
INTO STG_SALES_ORDER_HEADER
FROM AdventureWorks.Sales.SalesOrderHeader;
```

📌 Esta capa **NO es dbt**, es landing / raw.

---

## 6. dbt: Transformaciones

### 6.1 STAGING MODELS (dbt)

Ejemplo: `stg_sales_order.sql`

```sql
{{ config(materialized='view') }}

SELECT
  sod.SalesOrderID,
  sod.ProductID,
  soh.CustomerID,
  CAST(soh.OrderDate AS DATE) AS ORDER_DATE,
  sod.LineTotal AS SALES_AMOUNT,
  sod.OrderQty AS ORDER_QTY
FROM {{ source('oltp', 'SalesOrderDetail') }} sod
JOIN {{ source('oltp', 'SalesOrderHeader') }} soh
  ON sod.SalesOrderID = soh.SalesOrderID;
```

Objetivo:
- Unificar entidades OLTP
- Simplificar joins aguas abajo

---

### 6.2 Dimensiones (dbt)

Ejemplo: `dim_product.sql`

```sql
{{ config(materialized='table') }}

SELECT
  p.ProductID            AS PRODUCT_BK,
  p.Name                 AS PRODUCT_NAME,
  pc.Name                AS CATEGORY_NAME,
  psc.Name               AS SUBCATEGORY_NAME
FROM {{ source('oltp', 'Product') }} p
JOIN {{ source('oltp', 'ProductSubcategory') }} psc
  ON p.ProductSubcategoryID = psc.ProductSubcategoryID
JOIN {{ source('oltp', 'ProductCategory') }} pc
  ON psc.ProductCategoryID = pc.ProductCategoryID;
```

📌 Aquí se **desnormaliza** consciente y correctamente.

---

### 6.3 Hechos (dbt)

Ejemplo: `fct_sales.sql`

```sql
{{ config(materialized='table') }}

SELECT
  d.DATE_KEY,
  pr.PRODUCT_KEY,
  c.CUSTOMER_KEY,
  SUM(s.SALES_AMOUNT) AS SALES_AMOUNT,
  SUM(s.ORDER_QTY)    AS ORDER_QTY
FROM {{ ref('stg_sales_order') }} s
JOIN {{ ref('dim_date') }} d
  ON s.ORDER_DATE = d.DATE
JOIN {{ ref('dim_product') }} pr
  ON s.ProductID = pr.PRODUCT_BK
JOIN {{ ref('dim_customer') }} c
  ON s.CustomerID = c.CUSTOMER_BK
GROUP BY
  d.DATE_KEY,
  pr.PRODUCT_KEY,
  c.CUSTOMER_KEY;
```

---

## 7. Calidad y gobernanza

### dbt tests

```yaml
models:
  - name: fct_sales
    columns:
      - name: CUSTOMER_KEY
        tests:
          - not_null
          - relationships:
              to: ref('dim_customer')
              field: CUSTOMER_KEY
```

Tests = reglas DW vivas.

---

## 8. Consumo analítico

```sql
SELECT
  d.CALENDAR_YEAR,
  p.CATEGORY_NAME,
  SUM(f.SALES_AMOUNT) AS TOTAL_SALES
FROM fct_sales f
JOIN dim_date d   ON f.DATE_KEY = d.DATE_KEY
JOIN dim_product p ON f.PRODUCT_KEY = p.PRODUCT_KEY
GROUP BY
  d.CALENDAR_YEAR,
  p.CATEGORY_NAME;
```

✅ Query estable
✅ Reutilizable
✅ BI-friendly

---

## 9. Errores comunes evitados

- ❌ Usar claves naturales
- ❌ Calcular fechas en runtime
- ❌ Mezclar OLTP con DW
- ❌ Hechos sin granularidad definida

---

## 10. Resultado final

Este caso demuestra:

- Cómo pensar **de OLTP → negocio → DW**
- Cómo dbt formaliza el DW
- Cómo un DBA puede escalar a **arquitectura analítica moderna**

---

## 11. Extensiones recomendadas

- Incremental loads
- SCD Type 2 con snapshots
- CDC desde OLTP
- CI/CD dbt

