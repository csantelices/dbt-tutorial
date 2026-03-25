# dbt (Data Build Tool): Metodología, Aplicación y Camino de Aprendizaje

## 1. ¿Qué es dbt?

dbt (data build tool) es una herramienta para la capa de transformación (T) del enfoque ELT. Permite transformar datos directamente en el data warehouse usando SQL, incorporando prácticas de ingeniería de software al trabajo analítico.

Principios clave:
- Transformación dentro del warehouse
- SQL como lenguaje principal
- Modelos modulares y reutilizables
- Tests, documentación y control de versiones integrados

---

## 2. Metodología dbt (Analytics Engineering)

### 2.1 ELT y transformación in-database
Los datos se cargan crudos (RAW) al warehouse. dbt ejecuta los SELECT directamente ahí, aprovechando el motor SQL y evitando ETL externos complejos.

### 2.2 Arquitectura por capas

```
RAW (sources)
 └─ STAGING (stg_*)
     └─ INTERMEDIATE (int_*)
         └─ MARTS (dim_*, fct_*)
```

- Staging: limpieza mínima, renombrado, tipado
- Intermediate: lógica de negocio reutilizable
- Marts: modelos finales para BI y analítica

### 2.3 Modelos como código
Cada modelo es un archivo SQL con un SELECT. dbt gestiona dependencias (DAG), orden de ejecución y materialización (view, table, incremental).

### 2.4 Calidad de datos integrada
Tests declarativos:
- not_null
- unique
- accepted_values
- relationships

Ejemplo:

```yaml
columns:
  - name: CUSTOMER_ID
    tests:
      - not_null
      - unique
```

### 2.5 Documentación y lineage
Con `dbt docs generate` se crea documentación navegable con linaje automático entre modelos.

---

## 3. Aplicación de dbt en un proyecto real

### Flujo típico
1. Ingesta de datos → RAW
2. Transformación con dbt (`dbt run`, `dbt test`)
3. Publicación de modelos confiables
4. Consumo desde herramientas BI

### Orquestación
- dbt Cloud (scheduler propio)
- Airflow
- Azure Data Factory
- GitHub Actions

---

## 4. Camino de aprendizaje recomendado

### Nivel 0 – Prerrequisitos
- SQL intermedio
- Git básico
- Conceptos de Data Warehouse

---

### Nivel 1 – Fundamentos

Objetivo:
- Entender dbt y crear el primer proyecto

Recursos:
- Documentación oficial (Introducción a dbt)
- Curso: dbt Fundamentals (dbt Labs)

Hands-on:
- Inicializar proyecto
- Conectar a un warehouse
- Crear modelo stg_
- Ejecutar dbt run y dbt test

---

### Nivel 2 – Modelado analítico

Objetivo:
- Construir modelos analíticos confiables

Temas:
- ref() y DAG
- Materializations
- Dimensiones y hechos
- Modelos incrementales

Hands-on:
- dim_customers
- fct_orders
- Tests de relaciones (FK)

---

### Nivel 3 – dbt avanzado

Objetivo:
- Reutilización y gobernanza

Temas:
- Jinja y macros
- Snapshots (SCD Type 2)
- Seeds
- Paquetes (dbt_utils)

Hands-on:
- Crear macro reutilizable
- Implementar snapshot

---

### Nivel 4 – Producción y DataOps

Objetivo:
- Operar dbt en entornos productivos

Temas:
- CI/CD
- Entornos dev / prod
- Performance
- Observabilidad

Hands-on:
- Pipeline CI con dbt build
- Gates por tests
- Publicación de docs

---

## 5. Valor de dbt para perfiles DBA / SQL

- SQL como activo central
- Control explícito de dependencias
- Calidad de datos automatizada
- Menos lógica procedural
- Mejor auditoría y gobernanza

---

## 6. Próximos pasos posibles

- Mini proyecto dbt paso a paso
- dbt en Azure / Fabric / SQL Server
- Comparativa dbt vs SSIS / ADF
- Estándares corporativos dbt
