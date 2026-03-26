# OLTP → DW End‑to‑End con Incrementales + CDC (dbt + SQL Server)

Este documento es **tu guía de trabajo** para implementar un pipeline **productivo y defendible**:

- OLTP → Staging → Data Warehouse
- **Cargas incrementales reales**
- **CDC lógico** (ModifiedDate)
- **SCD Type 2** con snapshots
- **Repo Git listo para portafolio / entrevistas**

Perfil objetivo: **DBA / Senior SQL / Analytics Engineer**

---

## 1. Objetivo del diseño

Evitar:
- Full refresh innecesarios
- Reprocesar historia completa
- Stored procedures monolíticos
- ETL opaco

Lograr:
- Cargas eficientes
- Auditabilidad
- Escalabilidad
- Gobierno de datos

---

## 2. Estrategia de CDC (Change Data Capture)

Para este laboratorio se utiliza **CDC lógico basado en `ModifiedDate`**.

### ¿Por qué?
- Disponible en cualquier edición de SQL Server
- Simple de explicar y mantener
- Muy usado en entornos reales
- Perfecto para portafolio

> 📌 Alternativa Enterprise: SQL Server CDC nativo (fuera del scope inicial)

---

## 3. Arquitectura incremental

```
OLTP (AdventureWorks)
 ├── SalesOrderHeader (ModifiedDate)
 ├── SalesOrderDetail (ModifiedDate)
        ↓
STAGING / RAW (Carga 1:1)
        ↓
dbt STAGING (incremental)
        ↓
DIMENSIONES / HECHOS (incremental)
```

Principio:
> **Solo se procesa lo que cambió**

---

## 4. Incrementales en dbt – Staging

### stg_sales_order_incremental.sql

```sql
{{
  config(
    materialized = 'incremental',
    unique_key = 'SalesOrderDetailID'
  )
}}

SELECT
  sod.SalesOrderDetailID,
  sod.SalesOrderID,
  sod.ProductID,
  soh.CustomerID,
  CAST(soh.OrderDate AS DATE) AS ORDER_DATE,
  sod.LineTotal  AS SALES_AMOUNT,
  sod.OrderQty   AS ORDER_QTY,
  sod.ModifiedDate
FROM {{ source('oltp', 'SalesOrderDetail') }} sod
JOIN {{ source('oltp', 'SalesOrderHeader') }} soh
  ON sod.SalesOrderID = soh.SalesOrderID

{% if is_incremental() %}
WHERE sod.ModifiedDate > (
  SELECT MAX(ModifiedDate) FROM {{ this }}
)
{% endif %}
```

✅ CDC lógico
✅ Idempotente
✅ Escalable

---

## 5. Incrementales en FACT (DW real)

### fct_sales_incremental.sql

```sql
{{
  config(
    materialized = 'incremental',
    unique_key = 'FACT_SALES_KEY'
  )
}}

SELECT
  d.DATE_KEY,
  p.PRODUCT_KEY,
  c.CUSTOMER_KEY,
  SUM(s.SALES_AMOUNT) AS SALES_AMOUNT,
  SUM(s.ORDER_QTY)    AS ORDER_QTY,
  MAX(s.ModifiedDate) AS LAST_MODIFIED
FROM {{ ref('stg_sales_order_incremental') }} s
JOIN {{ ref('dim_date') }} d
  ON s.ORDER_DATE = d.DATE
JOIN {{ ref('dim_product') }} p
  ON s.ProductID = p.PRODUCT_BK
JOIN {{ ref('dim_customer') }} c
  ON s.CustomerID = c.CUSTOMER_BK

{% if is_incremental() %}
WHERE s.ModifiedDate > (
  SELECT MAX(LAST_MODIFIED) FROM {{ this }}
)
{% endif %}

GROUP BY
  d.DATE_KEY,
  p.PRODUCT_KEY,
  c.CUSTOMER_KEY;
```

📌 Patrón **altamente valorado** en entrevistas senior.

---

## 6. Dimensiones incrementales – Estrategia

| Dimensión | Estrategia |
|---------|------------|
| DIM_DATE | Seed / Static |
| DIM_PRODUCT | Incremental |
| DIM_CUSTOMER | Snapshot (SCD2) |

---

## 7. Snapshots – SCD Type 2 real

### snapshots/snap_dim_customer.sql

```sql
{% snapshot snap_dim_customer %}

{{
  config(
    target_schema = 'snapshots',
    unique_key = 'CustomerID',
    strategy = 'timestamp',
    updated_at = 'ModifiedDate'
  )
}}

SELECT
  c.CustomerID,
  p.FirstName,
  p.LastName,
  c.ModifiedDate
FROM {{ source('oltp', 'Customer') }} c
JOIN {{ source('oltp', 'Person') }} p
  ON c.PersonID = p.BusinessEntityID

{% endsnapshot %}
```

✅ Historia completa
✅ Cumple Kimball
✅ Listo para auditoría

---

## 8. Repo Git listo para portafolio

### Nombre recomendado

```
oltp-to-dw-dbt-sqlserver
```

---

## 9. Estructura profesional del repositorio

```
oltp-to-dw-dbt-sqlserver/
│
├── README.md
├── dbt_project.yml
├── profiles.yml.example
│
├── models/
│   ├── staging/
│   │   ├── stg_sales_order_incremental.sql
│   │   └── stg_customers.sql
│   │
│   ├── marts/
│   │   ├── dim_date.sql
│   │   ├── dim_product.sql
│   │   ├── dim_customer.sql
│   │   └── fct_sales_incremental.sql
│
├── snapshots/
│   └── snap_dim_customer.sql
│
├── tests/
│   └── relationships.yml
│
├── docs/
│   └── architecture.md
│
└── diagrams/
    └── dw_architecture.drawio
```

👉 **Esta estructura comunica seniority inmediatamente.**

---

## 10. Tests mínimos obligatorios

```yaml
models:
  - name: fct_sales_incremental
    columns:
      - name: CUSTOMER_KEY
        tests:
          - not_null
          - relationships:
              to: ref('dim_customer')
              field: CUSTOMER_KEY
```

---

## 11. README.md – Contenido recomendado

Incluye claramente:

- Contexto del negocio
- Arquitectura OLTP → DW
- Decisiones de modelado
- Incrementales + CDC
- Snapshots (SCD2)
- Tests y gobernanza

Ejemplo:

```md
## Architecture Decisions

- Incremental loading using ModifiedDate
- Kimball dimensional model
- dbt snapshots for SCD Type 2
- SQL Server execution engine

This project demonstrates a production‑ready OLTP to DW pipeline.
```

---

## 12. Checklist antes de subir a GitHub

✅ Incrementales funcionando
✅ Snapshots ejecutados
✅ dbt test sin errores
✅ dbt docs generado
✅ README claro
✅ Diagrama simple

---

## 13. Resultado final

Con este repo puedes afirmar:

> “Diseñé e implementé un pipeline OLTP → DW con dbt, usando incrementales y CDC lógico en SQL Server, con SCD Type 2 y modelo dimensional Kimball.”

Eso te posiciona directamente en **nivel senior**.

---

## 14. Próximos pasos (cuando vuelvas)

- CDC nativo SQL Server
- Incrementales avanzados (merge strategies)
- CI/CD con GitHub Actions
- Versión Azure SQL / Fabric

🚀 **Este documento es tu punto de partida.**
