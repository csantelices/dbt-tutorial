# Comparación Parquet vs Delta Lake

## 1. Objetivo
Este documento consolida una **comparación arquitectónica y técnica** entre **Parquet** y **Delta Lake**, explicando **qué es cada uno**, **por qué nació**, **qué problema resuelve**, y **cuándo usar cada uno** dentro del viaje de datos en arquitecturas modernas **on‑premise y multi‑cloud (Azure, AWS, GCP, OCI)**.

---

## 2. Definición rápida

### Parquet
Formato de archivo **binario y columnar**, optimizado para **lecturas analíticas** de alto volumen.

### Delta Lake
**Capa transaccional** que se construye **sobre archivos Parquet**, agregando **ACID, versionado, gobierno y confiabilidad** al Data Lake.

> Punto clave: **Delta Lake NO reemplaza Parquet; lo extiende.**

---

## 3. Origen e historia (problema → solución)

### Parquet: el problema del rendimiento analítico

**Contexto histórico**
- Año aproximado: **2013**
- Creadores: Twitter + Cloudera
- Ecosistema: Hadoop

**Problema**
- HDFS almacenaba datos orientados a filas
- Consultas analíticas requerían leer grandes volúmenes completos
- Alto I/O, bajo rendimiento

**Solución**
- Almacenamiento **por columnas**
- Compresión y encoding por tipo de dato
- Lectura selectiva de columnas necesarias

**Resultado**
- Alto rendimiento analítico
- Se convierte en estándar open‑source cloud

---

### Delta Lake: el problema de la confiabilidad del Data Lake

**Contexto histórico**
- Año aproximado: **2019**
- Creado por: Databricks

**Problema**
- Data Lakes carecían de:
  - Transacciones ACID
  - Control de concurrencia
  - Versionado
  - Manejo de esquemas
- Ingestas fallidas dejaban datos corruptos

**Solución**
- Mantener Parquet como formato físico
- Agregar:
  - Transaction Log (_delta_log)
  - ACID
  - Time travel
  - Schema enforcement

**Resultado**
- Nace el concepto de **Lakehouse**

---

## 4. Comparación conceptual

| Dimensión | Parquet | Delta Lake |
|---|---|---|
| Qué es | Formato de archivo | Capa transaccional |
| Nivel | Físico | Lógico/metadata |
| Relación | Standalone | Usa Parquet |
| Transacciones | No | Sí (ACID) |
| Versionado | No | Sí |
| Manejo de fallos | Manual | Automático |

---

## 5. Comparación técnica detallada

| Característica | Parquet | Delta Lake |
|---|---|---|
| Orientación | Columnar | Columnar (Parquet) |
| Compresión | Muy alta | Muy alta |
| Esquema | Parcial | Enforcement + Evolution |
| ACID | No | Sí |
| Updates / Deletes | No nativos | Sí |
| Concurrent writes | Riesgoso | Seguro |
| Time travel | No | Sí |
| Data quality | Externa | Integrada |
| Performance lectura | Excelente | Excelente |
| Performance escritura | Muy buena | Buena (ligero overhead) |

---

## 6. Uso en el viaje de datos

| Etapa | Parquet | Delta Lake |
|---|---|---|
| Zona RAW | ✅ | ✅ (recomendado) |
| Zona CURATED | ✅ | ✅✅ |
| Zona GOLD | ✅ | ✅✅ |
| Streaming ingestion | ⚠️ Limitado | ✅ |
| BI tradicional | ✅ | ✅ |
| Machine Learning | ✅ | ✅ |

---

## 7. Multi‑cloud y soporte

| Plataforma | Parquet | Delta Lake |
|---|---|---|
| Azure | Nativo | Full (Fabric, Databricks) |
| AWS | Nativo | Soportado (Databricks, EMR) |
| GCP | Nativo | Soportado (Dataproc, DBX) |
| OCI | Nativo | Limitado / vía Spark |

---

## 8. Ventajas y desventajas

### Parquet

**Ventajas**
- Open‑source estándar
- Interoperable
- Excelente performance analítica
- Compatible con cualquier motor

**Desventajas**
- Sin transacciones
- Sin versionado
- Gestión manual de cambios

---

### Delta Lake

**Ventajas**
- ACID en Data Lake
- Actualizaciones y deletes
- Time travel
- Confiabilidad operacional
- Base del Lakehouse

**Desventajas**
- Dependencia de motores compatibles
- Overhead de metadata
- Menor portabilidad que Parquet puro

---

## 9. Reglas arquitectónicas claras

- ✅ Si solo necesitas **lecturas analíticas eficientes** → **Parquet**
- ✅ Si necesitas **confiabilidad, updates y control** → **Delta Lake**
- ✅ Si operas **Data Lake productivo** → **Delta Lake**
- ✅ Si priorizas **máxima portabilidad multi‑cloud** → **Parquet**

---

## 10. Mensaje clave de arquitecto

> **Parquet resuelve el problema del rendimiento.**  
> **Delta Lake resuelve el problema de la confianza.**

En arquitecturas modernas, **Delta Lake es Parquet con garantías**.

---

**Documento de referencia para decisiones de arquitectura de datos.**
