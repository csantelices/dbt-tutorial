# 📘 Guía práctica de Markdown

Documento consolidado con los temas tratados en el chat para crear documentación técnica clara, ordenada y profesional usando Markdown.

---

## 🎯 Objetivo

Aprender a estructurar documentos Markdown utilizando:

- Secciones claras
- Iconos (emojis)
- Bloques colapsables
- Listas y checklists
- Buenas prácticas para GitHub, VS Code y Azure DevOps

---

## 📐 Estructura base recomendada

```markdown
# 📘 Título del Documento

Descripción breve.

---

## 🎯 Objetivo

## 🧩 Alcance

## 🛠️ Requisitos

## ⚙️ Procedimiento

## ✅ Validaciones

## 📎 Referencias
```

---

## 🎨 Uso de iconos (emojis)

Los emojis funcionan como iconos en Markdown y son ampliamente soportados.

Ejemplos comunes:

- 📘 Documento
- 🎯 Objetivo
- ⚠️ Advertencia
- ✅ Aprobado / correcto
- 🛑 Rollback
- 📊 Monitoreo
- 🗄️ Base de datos

Ejemplo:

```markdown
## ⚠️ Consideraciones importantes
✅ Ejecutar en horario de baja carga
```

---

## 📦 Bloques colapsables (`<details>`)

### ✅ Sintaxis básica

```markdown
<details>
<summary>🔽 Ver detalles</summary>

Contenido oculto:

- Texto
- Código
- Imágenes

</details>
```

### ✅ Regla clave

> **1 `<details>` = 1 bloque colapsable**
> **1 `<summary>` = título de ese bloque**

No se pueden crear múltiples secciones solo con varios `<summary>`.

---

## 🧩 Múltiples bloques colapsables al mismo nivel

```markdown
<details>
<summary>📘 Primera sección</summary>

Contenido A

</details>

<details>
<summary>📕 Segunda sección</summary>

Contenido B

</details>
```

---

## 🪜 Bloques colapsables anidados (uso moderado)

```markdown
<details>
<summary>⚙️ Procedimiento</summary>

Contenido principal

<details>
<summary>🛑 Rollback</summary>

Acción de reversa

</details>

</details>
```

---

## 📋 Listas con viñetas

Opciones válidas:

```markdown
- Item (recomendado)
+ Item
* Item
```

⚠️ Siempre debe haber un espacio después del símbolo.

---

## ✅ Checklists

### ✅ Sintaxis básica

```markdown
- [ ] Tarea pendiente
- [x] Tarea completada
```

### ✅ Ejemplo práctico

```markdown
## ✅ Checklist pre-ejecución

- [ ] Ventana aprobada
- [ ] Backup realizado
- [ ] Scripts revisados
```

### ✅ Dentro de `<details>`

```markdown
<details>
<summary>🔍 Checklist de validación</summary>

- [ ] Servicio activo
- [ ] Jobs habilitados

</details>
```

---

## 🧪 Checklists anidadas

```markdown
- [ ] Preparación
  - [ ] Revisar espacio
  - [ ] Validar permisos
- [ ] Ejecución
  - [x] Script principal
  - [ ] Validación final
```

---

## ☁️ Comportamiento en Azure DevOps

### ✅ Soportado en:

- Repositorios (.md)
- Wiki
- Pull Requests

✔️ Renderiza checklists y `<details>`
❌ No son interactivos (no clicables)

### ❌ No soportado en:

- Boards (Work Items)

---

## 📎 Buenas prácticas finales

- Usar `-` para listas
- Mantener consistencia visual
- Usar `<details>` para secciones largas
- No abusar de anidamientos
- Documentar pensando en el lector técnico

---

## ✅ Plantilla DBA sugerida

```markdown
# 🗄️ Nombre del procedimiento

## 🎯 Objetivo

## 🔐 Permisos requeridos

<details>
<summary>⚙️ Ejecución</summary>

```sql
-- Script principal
```

</details>

<details>
<summary>✅ Validación</summary>

```sql
-- Consultas de validación
```

</details>

<details>
<summary>🛑 Rollback</summary>

```sql
-- Plan de reversa
```

</details>
```

---

📘 **Fin del documento**
