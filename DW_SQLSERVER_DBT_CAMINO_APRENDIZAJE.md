# Data Warehouse + dbt en SQL Server

## Objetivo del documento

Este documento consolida:
- Conceptos clave de **Data Warehouse (Facts & Dimensions)**
- Uso de **AdventureWorksDW** como base analítica
- Integración práctica de estos conceptos usando **dbt sobre SQL Server**

Enfoque: perfil **DBA / SQL avanzado** que quiere profundizar en **modelado dimensional moderno y su implementación con dbt**.

---

## 1. Arquitectura general

```
OLTP / Source
   ↓ (ETL / ELT)
AdventureWorksDW (RAW / STAGING)
   ↓
dbt (Transformaciones SQL)
   ↓
Data Marts (Facts & Dimensions)
   ↓
BI / Analytics
```

Principio central:
> dbt no reemplaza el modelado DW – **lo formaliza, versiona y prueba**.

---

## 2. Recordatorio rápido: Hechos y Dimensiones

### Dimensiones (DIM)
- Contexto descriptivo
- Clave sustituta (Surrogate Key)
- Cambian lentamente (SCD)

Ejemplos AdventureWorksDW:
- `DimCustomer`
- `DimProduct`
- `DimDate`


### Hechos (FACT)
- Eventos de negocio
- Granularidad fija
- Medidas aditivas

Ejemplos:
- `FactInternetSales`
- `FactResellerSales`

---

## 3. dbt + SQL Server: consideraciones clave

### 3.1 Adaptador

Para SQL Server se utiliza:

```
dbt-sqlserver
```

Requisitos:
- SQL Server 2017+
- Autenticación SQL o Azure AD
- ODBC Driver 17/18

---

### 3.2 profiles.yml (ejemplo)

```yaml
sqlserver_dw:
  target: dev
  outputs:
    dev:
      type: sqlserver
      host: localhost
      port: 1433
      user: dbt_user
      password: strong_password
      database: AdventureWorksDW2019
      schema: dbt_dev
```

---

## 4. Estructura dbt recomendada (DW-centric)

```
models/
├── staging/
│   ├── stg_dim_customer.sql
│   ├── stg_dim_product.sql
│   └── stg_dim_date.sql
│
├── intermediate/
│   └── int_sales_enriched.sql
│
└── marts/
    ├── dim_customer.sql
    ├── dim_product.sql
    └── fct_sales.sql
```

Convención:
- STG = limpieza mínima
- INT = lógica de negocio
- MART = consumo analítico

---

## 5. Hands-on: Staging de Dimensiones

### 5.1 stg_dim_customer.sql

```sql
SELECT
  CustomerKey              AS CUSTOMER_KEY,
  CustomerAlternateKey     AS CUSTOMER_BK,
  FirstName,
  LastName,
  Gender,
  DateFirstPurchase
FROM dbo.DimCustomer;
```

Configuración:
```sql
{{ config(materialized='view') }}
```

Objetivo:
- No aplicar lógica de negocio
- Estandarizar nombres

---

## 6. Dimensión final (SCD-ready)

### dim_customer.sql

```sql
SELECT
  CUSTOMER_KEY,
  CUSTOMER_BK,
  FirstName,
  LastName,
  Gender,
  DateFirstPurchase
FROM {{ ref('stg_dim_customer') }};
```

Materialización:
```sql
{{ config(materialized='table') }}
```

---

## 7. Hechos en dbt (FACT)

### 7.1 stg_fact_sales.sql (opcional)

```sql
SELECT
  ProductKey,
  CustomerKey,
  OrderDateKey,
  SalesOrderNumber,
  SalesAmount,
  OrderQuantity
FROM dbo.FactInternetSales;
```

---

### 7.2 fct_sales.sql

```sql
SELECT
  F.OrderDateKey,
  F.CustomerKey,
  F.ProductKey,
  SUM(F.SalesAmount)  AS SALES_AMOUNT,
  SUM(F.OrderQuantity) AS ORDER_QTY
FROM {{ ref('stg_fact_sales') }} F
GROUP BY
  F.OrderDateKey,
  F.CustomerKey,
  F.ProductKey;
```

📌 Granularidad:
> Ventas por producto / cliente / fecha

---

## 8. Tests en dbt (calidad DW)

### Primary Key lógica (FACT)

```yaml
models:
  - name: fct_sales
    columns:
      - name: OrderDateKey
        tests:
          - not_null
```

### Relaciones (FK)

```yaml
      - name: CustomerKey
        tests:
          - relationships:
              to: ref('dim_customer')
              field: CUSTOMER_KEY
```

---

## 9. dbt refuerza principios DW

| Concepto DW | dbt lo asegura |
|-----------|----------------|
| Star schema | ref() y DAG |
| Claves FK | tests relationships |
| Granularidad | modelos unitarios |
| Gobierno | docs y lineage |
| Calidad | dbt test |

---

## 10. Consultas analíticas (consumo)

```sql
SELECT
  D.CalendarYear,
  P.EnglishProductCategoryName,
  SUM(F.SALES_AMOUNT) AS TOTAL_SALES
FROM {{ ref('fct_sales') }} F
JOIN {{ ref('dim_date') }} D
  ON F.OrderDateKey = D.DateKey
JOIN {{ ref('dim_product') }} P
  ON F.ProductKey = P.ProductKey
GROUP BY
  D.CalendarYear,
  P.EnglishProductCategoryName;
```

✅ DW mindset
✅ SQL limpio
✅ Reusable

---

## 11. Checklist DBA → Analytics Engineer

Antes de crear un modelo dbt:

- ¿Es FACT o DIM?
- ¿Granularidad definida?
- ¿Medidas aditivas?
- ¿SCD requerido?
- ¿Tests mínimos?

---

## 12. Próximo nivel

- Incremental models en SQL Server
- Snapshots SCD Type 2 con dbt
- CI/CD dbt (Azure DevOps)
- dbt + Fabric DW

---

**Resultado final:**
> dbt no cambia el DW, lo vuelve **auditable, testeable y profesional**.
