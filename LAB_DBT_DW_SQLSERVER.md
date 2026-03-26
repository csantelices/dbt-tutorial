# Laboratorio Práctico: Data Warehouse + dbt (SQL Server)

## Objetivo del laboratorio

Construir un **Data Warehouse dimensional (Facts & Dimensions)** usando **AdventureWorksDW** en **SQL Server**, aplicando **buenas prácticas de modelado Kimball** y **dbt como framework de transformación, testing y documentación**.

Perfil objetivo:
- DBA / SQL avanzado
- Interés en arquitectura DW moderna
- Enfoque práctico (laboratorio real)

---

## 1. Stack del laboratorio

- Motor: **SQL Server 2017+**
- Base de datos: **AdventureWorksDW** (Microsoft Sample)
- Framework ELT: **dbt Core**
- Adaptador: **dbt-sqlserver**
- Control de versiones: **Git**

---

## 2. Dataset base (AdventureWorksDW)

Microsoft provee versiones OLTP y DW del mismo negocio. Para este laboratorio se utiliza **AdventureWorksDW**.

Descarga oficial y guía de instalación:
- https://learn.microsoft.com/sql/samples/adventureworks-install-configure

La base ya incluye:
- Dimensiones: `DimCustomer`, `DimProduct`, `DimDate`, `DimGeography`
- Hechos: `FactInternetSales`, `FactResellerSales`

---

## 3. Arquitectura del laboratorio

```
AdventureWorksDW (source)
   ↓
dbt - STAGING (stg_*)
   ↓
dbt - MARTS
   ├── DIM_* (dimensiones conformadas)
   └── FCT_* (hechos analíticos)
   ↓
Consumo BI / SQL Analítico
```

Principio:
> dbt no reemplaza el DW, **refuerza sus reglas** con código, tests y lineage.

---

## 4. Estructura dbt del laboratorio

```
models/
├── staging/
│   ├── stg_dim_customer.sql
│   ├── stg_dim_product.sql
│   ├── stg_dim_date.sql
│   └── stg_fact_internet_sales.sql
│
├── marts/
│   ├── dim_customer.sql
│   ├── dim_product.sql
│   ├── dim_date.sql
│   └── fct_internet_sales.sql
│
└── schema.yml
```

Convenciones:
- STG: limpieza mínima, sin negocio
- DIM: entidades descriptivas
- FCT: eventos de negocio

---

## 5. Laboratorio paso a paso

### Fase 1 – Setup

1. Restaurar `AdventureWorksDW2019.bak`
2. Instalar dbt Core + adaptador SQL Server
3. Configurar `profiles.yml`
4. Validar conexión (`dbt debug`)

Guía oficial SQL Server + dbt:
- https://docs.getdbt.com/docs/local/connect-data-platform/mssql-setup

---

### Fase 2 – Staging

Ejemplo: `stg_dim_customer.sql`

```sql
{{ config(materialized='view') }}

SELECT
  CustomerKey          AS CUSTOMER_KEY,
  CustomerAlternateKey AS CUSTOMER_BK,
  FirstName,
  LastName,
  Gender,
  DateFirstPurchase
FROM dbo.DimCustomer;
```

Objetivo:
- Aislar dependencias
- Estandarizar nombres

---

### Fase 3 – Dimensiones

Ejemplo: `dim_customer.sql`

```sql
{{ config(materialized='table') }}

SELECT
  CUSTOMER_KEY,
  CUSTOMER_BK,
  FirstName,
  LastName,
  Gender,
  DateFirstPurchase
FROM {{ ref('stg_dim_customer') }};
```

Conceptos reforzados:
- Surrogate Key
- Dimensión conformada

---

### Fase 4 – Hechos

Ejemplo: `fct_internet_sales.sql`

```sql
{{ config(materialized='table') }}

SELECT
  OrderDateKey,
  CustomerKey,
  ProductKey,
  SUM(SalesAmount)   AS SALES_AMOUNT,
  SUM(OrderQuantity) AS ORDER_QTY
FROM {{ ref('stg_fact_internet_sales') }}
GROUP BY
  OrderDateKey,
  CustomerKey,
  ProductKey;
```

Granularidad:
> Ventas por producto / cliente / día

---

### Fase 5 – Tests (calidad DW)

```yaml
models:
  - name: fct_internet_sales
    columns:
      - name: CustomerKey
        tests:
          - not_null
          - relationships:
              to: ref('dim_customer')
              field: CUSTOMER_KEY
```

Tests = reglas DW automatizadas.

---

### Fase 6 – Documentación

```bash
dbt docs generate
dbt docs serve
```

Resultado:
- Lineage
- Contratosógicos DW
- Navegación del modelo

---

## 6. Laboratorio recomendado (GitHub)

### ✅ dbt + AdventureWorks (Kimball completo)

Repositorio:
https://github.com/hermandr/dbt-adventureworks

Incluye:
- Identificación de procesos de negocio
- Dimensional modeling paso a paso
- Documentación del star schema

Ideal como **laboratorio principal**.

---

## 7. Cursos prácticos recomendados

### dbt Fundamentals (oficial, gratis)
https://www.getdbt.com/dbt-learn

- Hands-on guiado
- Ideal como base

---

### Coursera – Analytics Engineering with dbt
https://www.coursera.org/specializations/analytics-engineering-with-dbt

- Modelado dimensional
- Proyecto aplicado

---

### Udemy – dbt Bootcamp (Zero to Hero)
https://www.udemy.com/course/complete-dbt-data-build-tool-bootcamp-zero-to-hero-learn-dbt/

- DW + SCD
- Testing avanzado

---

## 8. Orden recomendado de estudio

1. Restaurar AdventureWorksDW
2. Setup dbt + SQL Server
3. Ejecutar laboratorio dbt-adventureworks
4. Complementar con dbt Fundamentals
5. Consolidar con un curso largo (Coursera / Udemy)

---

## 9. Resultado esperado

Al finalizar:

- Dominas **Facts & Dimensions**
- Diseñas DW defendibles
- Aplicas dbt con criterio arquitectónico
- Pasas de DBA a **Analytics Engineer / DW Architect**

---

## 10. Siguientes extensiones posibles

- Incremental models
- Snapshots (SCD Type 2)
- CI/CD con Azure DevOps
- dbt + Microsoft Fabric
