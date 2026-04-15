# Preguntas de Aclaración — Plan One-Page Site Builder

> Este archivo está listo para que el usuario responda cada pregunta antes de crear el documento de trabajo definitivo.

---

## 1. Formato de los C-MD (Cuestionarios en Markdown)

Los archivos C-MD en `data-usr/` — ¿deben ser **formularios rellenables** en el propio Markdown o simplemente **documentos guía**?

**Opción A — Formulario rellenable:**
```markdown
### E2. Colores de tu marca
- Paleta preferida: [ ] warm  [ ] cool  [ ] vibrant  [ ] custom
- Color primario: _______________
- Color secundario: _______________
```

**Opción B — Documento guía:**
```markdown
### E2. Colores de tu marca
Elige una paleta y anota los colores hexadecimales.
Respuesta → editar `estructura-mi-negocio.json`
```

**Opción C — Híbrido:** Preguntas con espacio para responder en el mismo `.md`, más referencias a dónde va cada dato.


### Tu respuesta:
> _Opción A_

---

## 2. Conversión C-MD → JSON

¿Debe existir un **script automático** que lea las respuestas de los C-MD y genere los 3 JSON en `data-json/`, o el usuario (o un asistente IA posterior) hará esa conversión manualmente?

### Tu respuesta:
> _(un asistente IA posterior hará esa conversión, que será ejecutado por el usr manualmente desde el terminal)_

---

## 3. Formato de archivos de contenido (Fase 2 — esqueleto/andamio)

Para los archivos que el usuario rellene en `data-usr/`, ¿qué formato prefieres?

| Opción | Descripción |
|--------|-------------|
| **A** | Archivos `.md`/`.mdx` con frontmatter y placeholders (`## Título aquí`, `[Escribe tu texto...]`) |
| **B** | Archivos `.astro` con contenido placeholder dentro del template |
| **C** | Mixto: `.md` para contenido puro (about, servicios, etc.) y `.astro` para layouts |

### Tu respuesta:
> _(C)_

---

## 4. Sistema de Templates

¿Cada proyecto tiene **su propia copia** de un template base, o `templates/` contiene **tipos de template** y cada proyecto se asigna a uno?

**Opción A — Copia por proyecto:**
```
templates/mi-cafeteria/    ← Copia de un base, modificable
templates/mi-tienda/       ← Otra copia, modificable
```

**Opción B — Tipos reutilizables:**
```
templates/business/        ← Tipo negocio
templates/portfolio/       ← Tipo portfolio
templates/landing/         ← Tipo landing page
→ mi-cafeteria se asigna a "business"
```

### Tu respuesta:
> _(A)_

---

## 5. Proveedores de IA en el Cuestionario

¿Ofrecemos **todos los 9 proveedores** que ya tiene LocalSite-ai o un **subconjunto reducido** para el usuario novato?

**Todos (9):** DeepSeek, OpenRouter, Anthropic, Google, Mistral, Cerebras, OpenAI-compatible, Ollama, LM Studio

**Reducido sugerido:** Ollama (local gratis), DeepSeek (cloud económico), OpenAI-compatible (flexible)

### Tu respuesta:
> _(Todos 9)_

---

## 6. Contenido de `material-req.md`

¿Debe incluir **solo la lista de materiales requeridos** (checklist de archivos, colores, etc.) o también las **preguntas E1-E6 y J1-J6 con espacio para responder** dentro del mismo archivo?

**Opción A — Solo checklist:** Lista de lo que el usuario debe proporcionar (archivos, nombres, colores).

**Opción B — Checklist + Preguntas:** Lista de materiales + las preguntas E y J con espacio para responder en el mismo archivo.

### Tu respuesta:
> _(B)_

---

## 7. Orden del Flujo de Trabajo

¿Es correcto este orden?

```
1. Usuario recibe C-MD en data-usr/
2. Usuario rellena C-MD + proporciona assets en data-usr/
3. Script/conversión genera JSON en data-json/
4. Script crea esqueleto de contenido (Fase 2) en data-usr/
5. Usuario rellena archivos de contenido
6. Script ejecuta prompts IA → genera sitio Astro
7. Preview local
```

### Tu respuesta:
> _(Correcto)_

---

## 8. Formato de FAQs

Las FAQs van en archivo aparte siguiendo **FAQPage Schema de Schema.org**. ¿Qué formato prefieres?

**Opción A — JSON-LD directo:** El usuario recibe un archivo `.jsonld` con estructura FAQPage que rellena directamente.

**Opción B — Markdown P/R:** El usuario escribe en un `.md` con formato `P: ... / R: ...` y un script/prompt lo convierte a JSON-LD después.

### Tu respuesta:
> _(B)_

---

## 9. Pregunta Adicional: Límite entre Contenido y Estructura

Dices que el proyecto se limita a la **construcción, no a la elaboración de contenido**. Entonces:

- ¿El usuario debe tener **ya preparado todo el texto** (hero, about, servicios, etc.) antes de empezar?
- ¿O el proyecto genera **placeholders** que el usuario rellena después, tanto en los C-MD como en los archivos de contenido?

En otras palabras: ¿el usuario responde preguntas sobre **QUÉ secciones quiere** (estructura) o también sobre **QUÉ texto va en cada una** (contenido)?

### Tu respuesta:
> _(si El usuario debe tener **ya preparado todo el texto** 
el proyecto genera **placeholders** que el usuario rellena después con el texto preparado.
El sistema propone secciones y luego el usr  rellena o modifica/agrega)_

---

## 10. Pregunta Adicional: ¿Quién ejecuta los scripts?

El usuario novato, ¿ejecutará los scripts bash directamente en terminal, o espera que haya una **interfaz más amigable** (ej. un asistente paso a paso, un script único con menú interactivo, etc.)?

### Tu respuesta:
> _(de momento ejecutará los scripts bash directamente en terminal)_

---

> **Instrucción:** Copia este archivo, responde cada pregunta y devuélvelo para proceder con la creación del documento de trabajo.
