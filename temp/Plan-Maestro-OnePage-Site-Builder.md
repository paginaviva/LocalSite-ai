# Plan Maestro: LocalSite-ai One-Page Site Builder

## Tabla de Contenidos
1. [Objetivo Central y Visión del Producto](#1-objetivo-central-y-visión-del-producto)
2. [Arquitectura Propuesta del Proyecto](#2-arquitectura-propuesta-del-proyecto)
3. [Estructura de Carpetas Recomendada](#3-estructura-de-carpetas-recomendada)
4. [Script 1: crear-proyecto.sh — Creación de Nuevo Proyecto](#4-script-1-crear-proyectosh--creación-de-nuevo-proyecto)
5. [Batería de Preguntas al Usuario](#5-batería-de-preguntas-al-usuario)
6. [Script 2: recopilar-info.sh — Entrevistador Secuencial](#6-script-2-recopilar-infosh--entrevistador-secuencial)
7. [Estructura del JSON de Respuestas](#7-estructura-del-json-de-respuestas)
8. [Secuencia de Prompts Jerárquicos para Generar el Sitio](#8-secuencia-de-prompts-jerárquicos-para-generar-el-sitio)
9. [Script 3: generar-sitio.sh — Ejecución de Prompts y Generación](#9-script-3-generar-sitiosh--ejecución-de-prompts-y-generación)
10. [Script 4: previsualizar.sh — Despliegue Temporal Local](#10-script-4-previsualizarsh--despliegue-temporal-local)
11. [Flujo Completo: De Cero a Producción](#11-flujo-completo-de-cero-a-producción)
12. [Decisiones Técnicas Clave](#12-decisiones-técnicas-clave)
13. [Dependencias y Requisitos](#13-dependencias-y-requisitos)
14. [Riesgos y Mitigaciones](#14-riesgos-y-mitigaciones)
15. [Recomendaciones de Mantenibilidad](#15-recomendaciones-de-mantenibilidad)
16. [Hoja de Ruta de Implementación por Fases](#16-hoja-de-ruta-de-implementación-por-fases)

---

## 1. Objetivo Central y Visión del Producto

### Declaración de Objetivo

Crear un sistema automatizado dentro del repositorio LocalSite-ai que permita a un **usuario sin experiencia técnica** generar un **sitio web onepage profesional** mediante:

1. Ejecutar un script de creación de proyecto
2. Responder una batería de preguntas guiadas por terminal
3. Proporcionar archivos (logotipo, imágenes) en una carpeta designada
4. Ejecutar un script de generación que usa la IA de LocalSite-ai
5. Previsualizar el resultado en local
6. Disponer de una carpeta dist/ lista para desplegar en cualquier destino

### Principios de Diseño

| Principio | Descripción |
|-----------|-------------|
| **Zero-code para el usuario final** | El usuario solo responde preguntas y sube archivos. Nunca toca código |
| **Todo dentro del Codespace** | Prompts, JSON, assets, scripts y sitio generado residen en el repositorio |
| **Separación clara de responsabilidades** | LocalSite-ai (generador IA) separado de sites/ (sitios generados) |
| **Deploy-ready por defecto** | La carpeta dist/ de cada sitio está lista para GitHub Pages, Cloudflare, Railway, FTP |
| **Reutilizable y extensible** | Cada proyecto nuevo hereda la infraestructura; los prompts evolucionan |

---

## 2. Arquitectura Propuesta del Proyecto

### Visión de Alto Nivel

```
REPOSITORIO LOCALSITE-AI
│
├── LocalSite-ai/              # App Next.js existente (generador IA, NO se modifica)
│   ├── app/api/generate-code  # Endpoint existente (reutilizar como referencia)
│   ├── lib/providers/         # Sistema de proveedores de IA
│   └── ...
│
├── scripts/                   # NUEVO: Scripts de automatización
│   ├── crear-proyecto.sh      # Crear estructura de nuevo proyecto
│   ├── recopilar-info.sh      # Entrevistador secuencial → brief.json
│   ├── generar-sitio.sh       # Ejecutar prompts IA → generar Astro+React
│   └── previsualizar.sh       # Servidor HTTP local para revisión
│
├── prompts/                   # NUEVO: Prompts jerárquicos para IA
│   ├── 00-system-core.md      # System prompt base (Astro+React)
│   ├── 01-structure.md        # Generar estructura del proyecto
│   ├── 02-hero.md             # Generar sección Hero
│   ├── 03-sections.md         # Generar secciones de contenido
│   ├── 04-interactivity.md    # Generar componentes React interactivos
│   ├── 05-styling.md          # Generar estilos globales y tema
│   ├── 06-seo-meta.md         # Generar meta tags SEO
│   └── 07-assemble.md         # Ensamblar todo en index.astro final
│
├── templates/                 # NUEVO: Plantillas base
│   └── astro-react-base/      # Esqueleto Astro+React mínimo
│
├── sites/                     # NUEVO: Directorio de proyectos generados
│   └── [nombre-proyecto]/     # Un subdirectorio por sitio
│       ├── brief.json         # Respuestas del usuario (fuente de verdad)
│       ├── assets-input/      # DONDE el usuario deposita sus archivos
│       ├── assets/            # Assets procesados
│       ├── prompts/           # Copia de prompts usados (auditoría)
│       ├── dist/              # RESULTADO FINAL: sitio build, listo para deploy
│       └── README.md          # Instrucciones del proyecto
│
└── temp/                      # Documentos de análisis (ya existe)
```

### Diagrama de Flujo de Datos

```
Usuario ejecuta:
  │
  ├─ 1. ./scripts/crear-proyecto.sh mi-negocio
  │     → Crea sites/mi-negocio/ con estructura base
  │
  ├─ 2. Coloca archivos en sites/mi-negocio/assets-input/
  │     → logo.png, hero-image.jpg, favicon.ico, etc.
  │
  ├─ 3. ./scripts/recopilar-info.sh mi-negocio
  │     → Pregunta secuencial en terminal (~40 preguntas)
  │     → Guarda sites/mi-negocio/brief.json
  │
  ├─ 4. ./scripts/generar-sitio.sh mi-negocio
  │     → Lee brief.json
  │     → Inyecta datos en prompts/01..07
  │     → Llama a IA (Ollama, DeepSeek, etc.) secuencialmente
  │     → Parsea respuestas en archivos Astro+React
  │     → Instala dependencias y hace astro build
  │     → Resultado en sites/mi-negocio/dist/dist/
  │
  └─ 5. ./scripts/previsualizar.sh mi-negocio
        → Sirve dist/ en http://localhost:8080
        → Usuario revisa y valida
```

### Estrategia de Integración LocalSite-ai → Astro + React

**Método elegido: Generación directa de Astro+React por IA mediante prompts especializados.**

Los 7 prompts jerárquicos instruyen a la IA para generar código Astro+React directamente. Se usa la infraestructura de proveedores de IA de LocalSite-ai (mismas API keys, mismos modelos) pero llamada directamente desde los scripts bash, sin necesidad de que el servidor Next.js esté corriendo.

**Fallback:** Si la generación directa falla, se genera HTML monolítico con LocalSite-ai y un post-procesador lo convierte en estructura Astro básica.

---

## 3. Estructura de Carpetas Recomendada

```
LocalSite-ai/                              # Raíz del repositorio
│
├── LocalSite-ai/                          # App Next.js existente (INTACTA)
│   ├── app/
│   ├── components/
│   ├── hooks/
│   ├── lib/providers/
│   └── ...
│
├── scripts/
│   ├── crear-proyecto.sh
│   ├── recopilar-info.sh
│   ├── generar-sitio.sh
│   └── previsualizar.sh
│
├── prompts/
│   ├── 00-system-core.md
│   ├── 01-structure.md
│   ├── 02-hero.md
│   ├── 03-sections.md
│   ├── 04-interactivity.md
│   ├── 05-styling.md
│   ├── 06-seo-meta.md
│   └── 07-assemble.md
│
├── templates/
│   └── astro-react-base/
│       ├── astro.config.mjs
│       ├── package.json
│       ├── tsconfig.json
│       ├── public/
│       │   └── favicon.svg
│       └── src/
│           ├── layouts/
│           │   └── Layout.astro
│           └── pages/
│               └── index.astro
│
├── sites/
│   └── mi-cafeteria/                      # Ejemplo de proyecto
│       ├── brief.json
│       ├── assets-input/
│       │   ├── logo.png
│       │   ├── hero-photo.jpg
│       │   └── favicon.ico
│       ├── assets/
│       ├── prompts/
│       │   └── rendered/
│       ├── dist/
│       │   ├── astro.config.mjs
│       │   ├── package.json
│       │   ├── public/
│       │   ├── src/
│       │   │   ├── layouts/
│       │   │   │   └── Layout.astro
│       │   │   ├── pages/
│       │   │   │   └── index.astro
│       │   │   ├── components/
│       │   │   │   ├── Hero.astro
│       │   │   │   ├── About.astro
│       │   │   │   ├── Services.astro
│       │   │   │   ├── Testimonials.astro
│       │   │   │   ├── FAQ.tsx
│       │   │   │   ├── ContactForm.tsx
│       │   │   │   └── Navbar.astro
│       │   │   └── styles/
│       │   │       └── global.css
│       │   ├── node_modules/
│       │   └── dist/                      # BUILD FINAL → deploy-ready
│       │       ├── index.html
│       │       ├── _astro/
│       │       └── assets/
│       └── README.md
│
└── temp/
    ├── LocalSite-ai-Analisis-Completo.md
    ├── LocalSite-ai-con-Astro-React-Analisis.md
    └── Plan-Maestro-OnePage-Site-Builder.md
```

### Reglas de la Estructura

| Regla | Descripción |
|-------|-------------|
| `sites/` es el único output | Todo sitio generado vive en `sites/[nombre]/` |
| `dist/` es deploy-ready | `sites/[nombre]/dist/dist/` contiene el build final |
| `assets-input/` es zona de drop | El usuario solo copia archivos aquí |
| `brief.json` es la fuente de verdad | Todo contenido proviene de este archivo |
| LocalSite-ai se mantiene intacto | Solo se añaden prompts, no se modifica la app |

---

## 4. Script 1: crear-proyecto.sh — Creación de Nuevo Proyecto

### Propósito

Crear la estructura de carpetas para un nuevo proyecto dentro de `sites/[nombre]/`.

### Uso

```bash
./scripts/crear-proyecto.sh <nombre-proyecto>
# Ejemplo:
./scripts/crear-proyecto.sh mi-cafeteria
```

### Comportamiento

1. Valida el nombre (minúsculas, números, guiones)
2. Comprueba que no exista ya
3. Crea la estructura: `assets-input/`, `assets/`, `prompts/`, `dist/`
4. Copia la plantilla base Astro+React a `dist/`
5. Copia los prompts actuales a `prompts/`
6. Crea `brief.json` con estructura vacía y metadata
7. Genera `README.md` con instrucciones del proyecto
8. Muestra resumen con siguientes pasos

### Validaciones

- Nombre: solo `a-z`, `0-9`, `-`; debe empezar y terminar con alfanumérico
- No permite nombres duplicados
- Verifica existencia de `templates/` y `prompts/`

---

## 5. Batería de Preguntas al Usuario

El cuestionario tiene **10 secciones** (~45 preguntas), organizadas de lo general a lo específico.

### Sección A: Identidad (4 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| A1 | ¿Nombre de tu marca/negocio? | Texto | ✅ |
| A2 | ¿Eslogan o frase descriptiva? | Texto | ❌ |
| A3 | ¿Idioma principal? | Selección | ✅ |
| A4 | ¿Necesitas más de un idioma? | Sí/No | ❌ |

### Sección B: Objetivo (3 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| B1 | ¿Objetivo principal? | Selección múltiple | ✅ |
| B2 | Describe qué quieres lograr | Texto largo | ✅ |
| B3 | ¿Acción más importante del visitante? | Texto | ✅ |

### Sección C: Estructura (8 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| C1 | ¿Qué secciones quieres? | Selección múltiple (16 opciones) | ✅ |
| C2 | ¿Sección de precios? | Sí/No | ✅ |
| C3 | ¿FAQ? | Sí/No | ✅ |
| C4 | ¿Galería? | Sí/No | ✅ |
| C5 | ¿Testimonios? | Sí/No | ✅ |
| C6 | ¿Formulario de contacto? | Sí/No | ✅ |
| C7 | ¿Equipo? | Sí/No | ✅ |
| C8 | ¿Estadísticas/contadores? | Sí/No | ✅ |

Opciones de C1: hero, about, services, features, pricing, testimonials, faq, gallery, team, stats, newsletter, contact, map, cta_banner, logos, footer.

### Sección D: Contenido — Textos (8 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| D1 | Título del Hero | Texto | ✅ |
| D2 | Subtítulo del Hero | Texto | ❌ |
| D3 | Texto del botón CTA | Texto | ✅ |
| D4 | Texto de Sobre Nosotros | Texto largo | Condicional |
| D5 | Lista de Servicios | Multi-línea | Condicional |
| D6 | FAQ (P:/R:) | Multi-línea | Condicional |
| D7 | Testimonios | Multi-línea | Condicional |
| D8 | Texto del Footer | Texto | ❌ |

### Sección E: Branding (6 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| E1 | ¿Tienes logotipo? | Sí/No + archivo | ❌ |
| E2 | Colores de marca | Selección (7 paletas) + custom | ✅ |
| E3 | Estilo visual | Selección (7 estilos) | ✅ |
| E4 | Fondo claro u oscuro | Selección | ✅ |
| E5 | Tipografía preferida | Texto | ❌ |
| E6 | Web de referencia | URL | ❌ |

### Sección F: Contacto (10 preguntas)

| # | Pregunta | Tipo |
|---|----------|------|
| F1 | Email | Email |
| F2 | Teléfono | Teléfono |
| F3 | Dirección | Texto |
| F4 | Google Maps URL | URL |
| F5 | Instagram | @ o URL |
| F6 | Facebook | URL |
| F7 | Twitter/X | @ o URL |
| F8 | LinkedIn | URL |
| F9 | TikTok | @ o URL |
| F10 | Otros enlaces | URL |

### Sección G: CTA (6 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| G1 | Acción principal del visitante | Texto | ✅ |
| G2 | Texto del botón principal | Texto | ✅ |
| G3 | ¿Botón secundario? | Sí/No | ✅ |
| G4 | Texto botón secundario | Texto | Condicional |
| G5 | Destino del CTA | Selección | ✅ |
| G6 | URL externa (si aplica) | URL | Condicional |

### Sección H: Público Objetivo (3 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| H1 | ¿A quién va dirigido? | Texto | ✅ |
| H2 | Zona geográfica | Texto | ❌ |
| H3 | Diferenciador del público | Texto | ❌ |

### Sección I: SEO (4 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| I1 | Título SEO (50-60 chars) | Texto | ✅ |
| I2 | Descripción SEO (150-160 chars) | Texto | ✅ |
| I3 | Palabras clave | Texto | ❌ |
| I4 | ¿Favicon? | Sí/No | ❌ |

### Sección J: Assets (6 preguntas)

| # | Pregunta | Tipo |
|---|----------|------|
| J1 | Logotipo en assets-input/ | Sí/No + nombre |
| J2 | Imagen del Hero | Sí/No + nombre |
| J3 | Imágenes de galería | Sí/No + lista |
| J4 | Fotos del equipo | Sí/No + lista |
| J5 | Favicon | Sí/No + nombre |
| J6 | Otros archivos | Sí/No + lista |

### Sección K: Técnico (4 preguntas)

| # | Pregunta | Tipo | Requerida |
|---|----------|------|-----------|
| K1 | Destino de deploy | Selección (7 opciones) | ❌ |
| K2 | ¿Accesibilidad WCAG? | Sí/No | ✅ |
| K3 | ¿Analytics? | Sí/No | ❌ |
| K4 | Notas adicionales | Texto libre | ❌ |

---

## 6. Script 2: recopilar-info.sh — Entrevistador Secuencial

### Propósito

Ejecutar las ~45 preguntas del cuestionario de forma secuencial en terminal, con validación, colores, ejemplos, y guardar todo en `brief.json`.

### Uso

```bash
./scripts/recopilar-info.sh <nombre-proyecto>
```

### Características

- **Colores y formato visual**: secciones con encabezados, emojis, hints
- **Validación**: campos requeridos no permiten respuesta vacía
- **Selecciones numeradas**: el usuario elige por número
- **Selección múltiple**: separada por comas
- **Sí/No**: acepta s/n/sí/yes/no
- **Condicionales**: preguntas que solo aparecen si aplica
- **Generación de JSON con Python**: evita errores de escaping
- **Resumen final**: confirma dónde se guardó

### Funciones auxiliares

| Función | Propósito |
|---------|-----------|
| `read_required()` | Pregunta obligatoria, repite hasta respuesta válida |
| `read_optional()` | Pregunta opcional, acepta vacío |
| `read_selection()` | Menú numerado, valida número |
| `read_multi_selection()` | Múltiple con comas |
| `read_yesno()` | Sí/No con defaults |
| `section()` | Encabezado visual de sección |
| `ask()` / `hint()` | Formato de pregunta y ejemplo |

---

## 7. Estructura del JSON de Respuestas

### brief.json — Esquema Completo

```json
{
  "_schema_version": "1.0.0",
  "_created_at": "2026-04-07T14:30:00Z",
  "_project_name": "mi-cafeteria",

  "identidad": {
    "nombre": "Café Aromas",
    "eslogan": "El café que despierta tus sentidos",
    "idioma_principal": "español",
    "multidioma": false
  },

  "objetivo": {
    "objetivos": "Mostrar información del negocio, Captar clientes potenciales, Recibir reservas",
    "descripcion": "Quiero que los visitantes conozcan mi cafetería, vean el menú y reserven mesa",
    "accion_principal": "Reservar una mesa"
  },

  "estructura": {
    "secciones": "hero, about, services, testimonials, faq, contact, footer",
    "tiene_precios": false,
    "tiene_faq": true,
    "tiene_galeria": false,
    "tiene_testimonios": true,
    "tiene_formulario": true,
    "tiene_equipo": false,
    "tiene_estadisticas": false
  },

  "contenido": {
    "hero": {
      "titulo": "El Café que Despierta tus Sentidos",
      "subtitulo": "Café de especialidad tostado artesanalmente desde 2015",
      "cta_texto": "Reserva tu Mesa"
    },
    "about": { "texto": "Somos una cafetería familiar..." },
    "servicios": { "lista": "Café de especialidad\nTalleres de cata" },
    "faq": { "lista": "P: ¿Aceptan reservas?\nR: Sí, por teléfono o web." },
    "testimonios": { "lista": "El mejor café — María G." },
    "footer": { "texto": "© 2026 Café Aromas." }
  },

  "branding": {
    "tiene_logo": true,
    "logo_archivo": "logo.png",
    "paleta_color": "warm",
    "color_secundario_hex": "#D4A574",
    "estilo_visual": "modern",
    "fondo": "oscuro",
    "tipografia": "Sans-serif moderna",
    "referencia_web": ""
  },

  "contacto": {
    "email": "hola@cafearomas.com",
    "telefono": "+34 912 345 678",
    "direccion": "Calle Mayor 15, Madrid",
    "maps_url": "https://maps.google.com/...",
    "redes_sociales": {
      "instagram": "@cafearomas",
      "facebook": "https://facebook.com/cafearomas",
      "twitter": "",
      "linkedin": "",
      "tiktok": "@cafearomas",
      "otros": "https://wa.me/34600123456"
    }
  },

  "cta": {
    "accion_principal": "Reservar una mesa",
    "texto_boton": "Reserva tu Mesa",
    "tiene_secundario": true,
    "texto_boton_secundario": "Ver el Menú",
    "destino_tipo": "sección contacto (#contacto)",
    "destino_url": ""
  },

  "publico_objetivo": {
    "descripcion": "Adultos 25-45 años que aprecian buen café",
    "zona_geografica": "Madrid centro",
    "diferenciador": "Valoran sostenibilidad y comercio local"
  },

  "seo": {
    "titulo": "Café Aromas — Café de Especialidad en Madrid",
    "descripcion": "Café de especialidad tostado artesanalmente. Reserva tu mesa en Madrid.",
    "keywords": "café especialidad, cafetería Madrid, café artesanal",
    "tiene_favicon": true
  },

  "assets": {
    "logo": "logo.png",
    "hero_image": "hero-cafe.jpg",
    "galeria": "",
    "equipo": "",
    "favicon": "favicon.ico",
    "otros": "menu.pdf"
  },

  "preferencias_tecnicas": {
    "deploy_target": "Cloudflare Pages",
    "accesibilidad": false,
    "analytics": false,
    "notas_extra": "FAQ con animación suave al abrir"
  }
}
```

### Reglas del Esquema

- Campos `_` son metadata interna
- Orden lógico: identidad → objetivo → estructura → contenido → branding → contacto → cta → público → seo → assets → técnicas
- Booleanos: `true`/`false`
- Listas: separadas por `", "` en string
- Textos multi-línea: `\n`
- Campos vacíos: `""`, no `null`

---

## 8. Secuencia de Prompts Jerárquicos para Generar el Sitio

### Filosofía

Los prompts son **secuenciales y dependientes**: cada uno usa las respuestas del brief.json como contexto. Esto evita que la IA pierda información o genere inconsistencias.

### Diagrama de Dependencias

```
                    ┌──────────────────────┐
                    │  brief.json          │
                    │  (Fuente de verdad)  │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  00-system-core.md   │  ← System Prompt (siempre activo)
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
    ┌─────────▼──────┐  ┌─────▼──────┐  ┌──────▼──────┐
    │ 01-structure.md│  │05-styling  │  │ 06-seo-meta │
    │ (Estructura    │  │ .md        │  │ .md         │
    │  base)         │  │ (Estilos)  │  │ (SEO)       │
    └─────────┬──────┘  └─────┬──────┘  └──────┬──────┘
              │                │                │
              │         ┌──────▼───────────────▼┐
              │         │ 02-hero.md            │
              │         │ (Hero section)        │
              │         └──────────┬────────────┘
              │                    │
              │         ┌──────────▼────────────┐
              │         │ 03-sections.md        │
              │         │ (Secciones estáticas) │
              │         └──────────┬────────────┘
              │                    │
              │         ┌──────────▼────────────┐
              │         │ 04-interactivity.md   │
              │         │ (Componentes .tsx)    │
              │         └──────────┬────────────┘
              │                    │
              └────────────────────┼────────────────┘
                                   │
                        ┌──────────▼──────────┐
                        │ 07-assemble.md      │
                        │ (Ensamblaje final)  │
                        └─────────────────────┘
```

### Prompt 0: System Core (`00-system-core.md`)

Establece rol, reglas y contexto base. Siempre activo en todas las llamadas.

- Rol: desarrollador web experto en Astro + React
- Stack: Astro 5.x, React 19, TypeScript, CSS scoped
- Reglas: solo código, un archivo por componente, HTML semántico, responsive mobile-first, variables CSS para colores
- Formato de respuesta: bloques `file: ruta/archivo.ext` + contenido

### Prompt 1: Estructura (`01-structure.md`)

Variables: `{{identidad.nombre}}`, `{{branding.estilo_visual}}`, `{{branding.paleta_color}}`, `{{estructura.secciones}}`, `{{seo.titulo}}`, `{{contenido.footer.texto}}`

Genera: `astro.config.mjs`, `package.json`, `tsconfig.json`, `src/layouts/Layout.astro`, `src/pages/index.astro`

### Prompt 2: Hero (`02-hero.md`)

Variables: `{{contenido.hero.*}}`, `{{cta.*}}`, `{{branding.*}}`, `{{assets.hero_image}}`

Genera: `src/components/Hero.astro`

### Prompt 3: Secciones (`03-sections.md`)

Variables: `{{estructura.secciones}}`, `{{contenido.*}}`, `{{branding.*}}`, `{{assets.*}}`

Genera: `src/components/About.astro`, `Services.astro`, `Testimonials.astro`, `Gallery.astro`, `Team.astro`, `Stats.tsx` (condicionales según brief)

### Prompt 4: Interactividad (`04-interactivity.md`)

Variables: `{{estructura.tiene_*}}`, `{{contenido.faq.lista}}`, `{{contenido.servicios.lista}}`, `{{branding.*}}`

Genera: `src/components/Pricing.tsx`, `FAQ.tsx`, `ContactForm.tsx`, `Stats.tsx`

### Prompt 5: Estilos (`05-styling.md`)

Variables: `{{branding.paleta_color}}`, `{{branding.color_secundario_hex}}`, `{{branding.estilo_visual}}`, `{{branding.fondo}}`, `{{branding.tipografia}}`

Genera: `src/styles/global.css` con variables CSS, reset, clases utilitarias

### Prompt 6: SEO (`06-seo-meta.md`)

Variables: `{{seo.*}}`, `{{identidad.*}}`, `{{contacto.*}}`

Genera: Actualización de `src/layouts/Layout.astro` con meta tags, Open Graph, JSON-LD

### Prompt 7: Ensamblaje (`07-assemble.md`)

Variables: TODO el brief.json

Genera: `src/pages/index.astro` final con todos los imports, `src/layouts/Layout.astro` definitivo

---

## 9. Script 3: generar-sitio.sh — Ejecución de Prompts y Generación

### Propósito

Leer `brief.json`, inyectar variables en los prompts, llamar a la IA secuencialmente, parsear las respuestas en archivos, instalar dependencias y hacer build.

### Uso

```bash
./scripts/generar-sitio.sh <nombre-proyecto>
```

### Flujo Interno

1. **Valida** existencia de `brief.json` y configuración de IA
2. **Prepara** `dist/` limpio con estructura base
3. **Copia** assets de `assets-input/` a `dist/public/` y `dist/src/assets/`
4. **Inyecta** variables del JSON en cada prompt (Python con regex `{{path}}`)
5. **Ejecuta** cada prompt secuencialmente:
   - Lee system prompt + prompt renderizado
   - Llama a la API del proveedor activo (Ollama, DeepSeek, OpenAI-compatible)
   - Parsea la respuesta extrayendo bloques `file: ruta.ext`
   - Escribe archivos en `dist/`
6. **Instala** dependencias con `npm install`
7. **Ejecuta** `npx astro build`
8. **Reporta** resultado

### Funciones Clave

| Función | Propósito |
|---------|-----------|
| `inject_prompt()` | Sustituye `{{path.to.value}}` con datos del JSON |
| `call_ai_api()` | Detecta proveedor y llama a la API correspondiente |
| `parse_ai_response()` | Extrae archivos del formato `file: ruta` de la respuesta IA |

### Proveedores Soportados

| Proveedor | Variable de entorno | Modelo por defecto |
|-----------|-------------------|-------------------|
| Ollama | `OLLAMA_API_BASE` | `qwen2.5-coder:32b` |
| DeepSeek | `DEEPSEEK_API_KEY`, `DEEPSEEK_API_BASE` | `deepseek-chat` |
| OpenAI-compatible | `OPENAI_COMPATIBLE_API_KEY`, `OPENAI_COMPATIBLE_API_BASE` | Configurable |

---

## 10. Script 4: previsualizar.sh — Despliegue Temporal Local

### Propósito

Levantar un servidor HTTP temporal para revisar el sitio generado antes de desplegar.

### Uso

```bash
./scripts/previsualizar.sh <nombre-proyecto> [puerto]
# Ejemplo:
./scripts/previsualizar.sh mi-cafeteria
./scripts/previsualizar.sh mi-cafeteria 9000
```

### Lógica de Prioridad

1. Si existe `dist/dist/index.html` (astro build completado) → sirve estático con Python HTTP server
2. Si existe `dist/index.html` (HTML directo) → sirve directamente
3. Si existe `dist/package.json` pero no hay build → intenta `astro build`, si falla inicia `astro dev`
4. Si no hay contenido → error con instrucciones

### Servidores Utilizados

- **Primario**: `python3 -m http.server` (siempre disponible, bind 127.0.0.1)
- **Fallback**: `npx serve` (si python3 no está)

### Puerto por Defecto

`8080` (configurable como segundo argumento)

---

## 11. Flujo Completo: De Cero a Producción

### Ejemplo Completo para Usuario Novato

```bash
# PASO 1: Crear proyecto
./scripts/crear-proyecto.sh mi-cafeteria
# → Crea sites/mi-cafeteria/ con estructura vacía

# PASO 1.5: Colocar assets (desde la UI del Codespace)
# El usuario arrastra a sites/mi-cafeteria/assets-input/:
#   logo.png, hero-photo.jpg, favicon.ico

# PASO 2: Responder cuestionario
./scripts/recopilar-info.sh mi-cafeteria
# → ~45 preguntas en 10 secciones
# → Guarda sites/mi-cafeteria/brief.json

# PASO 3: Generar sitio con IA
./scripts/generar-sitio.sh mi-cafeteria
# → 7 llamadas a IA secuenciales
# → Parsea respuestas en archivos Astro+React
# → npm install + astro build
# → Resultado en sites/mi-cafeteria/dist/dist/

# PASO 4: Previsualizar
./scripts/previsualizar.sh mi-cafeteria
# → http://localhost:8080
# → Usuario revisa, toma notas

# PASO 5 (opcional): Iterar
# Editar brief.json manualmente y regenerar:
./scripts/generar-sitio.sh mi-cafeteria

# PASO 6: Desplegar
# El contenido de sites/mi-cafeteria/dist/dist/ está listo para:
#   - GitHub Pages: copiar a rama gh-pages
#   - Cloudflare Pages: drag & drop en dashboard
#   - Netlify: drag & drop
#   - FTP: subir contenido de dist/
#   - Railway: conectar repositorio
```

---

## 12. Decisiones Técnicas Clave

### Decisión 1: Astro + React (no Vue)

**Razones:** LocalSite-ai ya usa React; JSX es más cercano al HTML (más fácil para la IA); ecosistema más grande; el usuario que usa LocalSite-ai ya ve React en acción.

### Decisión 2: Prompts jerárquicos (no mega-prompt)

**Razones:** Cada prompt enfocado; menor uso de tokens por llamada; dependencias claras; más fácil de debuggear y mejorar individualmente.

### Decisión 3: brief.json como fuente de verdad

**Razones:** Editable manualmente; versionable en Git; reutilizable para regenerar; compartible como plantilla.

### Decisión 4: Scripts Bash (no app interactiva)

**Razones:** Disponible en todo Codespace sin instalación; transparente; automatizable; sin dependencias extra.

### Decisión 5: IA directa (no proxy por LocalSite-ai)

**Razones:** No necesita servidor Next.js corriendo; más simple; los scripts detectan el proveedor activo y llaman directamente. Se reutilizan las mismas API keys y configuración.

### Decisión 6: dist/ dentro del proyecto

**Razones:** Múltiples proyectos simultáneos; cada uno con build independiente; fácil identificación.

---

## 13. Dependencias y Requisitos

### Sistema

| Dependencia | Versión | Uso | Disponibilidad |
|-------------|---------|-----|----------------|
| bash | 5.x | Scripts | Pre-instalado |
| python3 | 3.10+ | JSON, template injection | Pre-instalado |
| node.js | 18.17+ | Astro, React, build | Pre-instalado |
| npm | 9+ | Dependencias | Con Node.js |
| curl | Cualquiera | APIs de IA | Pre-instalado |

### Proveedor de IA (al menos uno)

| Proveedor | Configuración | Costo |
|-----------|--------------|-------|
| Ollama (recomendado) | `OLLAMA_API_BASE=http://localhost:11434` | Gratis |
| DeepSeek | `DEEPSEEK_API_KEY` + `DEEPSEEK_API_BASE` | Bajo |
| OpenAI-compatible | `OPENAI_COMPATIBLE_API_KEY` + `OPENAI_COMPATIBLE_API_BASE` | Variable |

### Modelo Recomendado

| Modelo | Proveedor | Contexto | Calidad código |
|--------|-----------|----------|----------------|
| qwen2.5-coder:32b | Ollama | 32K | Excelente |
| deepseek-coder:33b | Ollama | 16K | Excelente |
| deepseek-chat | DeepSeek Cloud | 32K | Excelente |
| claude-sonnet | Anthropic | 200K | Excelente |

### Dependencias del Proyecto Astro Generado

```json
{
  "astro": "^5.0.0",
  "@astrojs/react": "^4.0.0",
  "react": "^19.0.0",
  "react-dom": "^19.0.0"
}
```

---

## 14. Riesgos y Mitigaciones

| # | Riesgo | Impacto | Probabilidad | Mitigación |
|---|--------|---------|--------------|------------|
| R1 | IA genera código con errores | Alto | Media | Build falla → script detecta; usuario edita manualmente |
| R2 | Tokens insuficientes | Alto | Media | Prompts divididos en 7 pasos (4-8K tokens cada uno) |
| R3 | IA no respeta formato archivos | Medio | Media | Parser Python con fallback a raw response |
| R4 | Ollama lento o sin memoria | Medio | Alta | Modelos 7B-13B como fallback; timeout configurable |
| R5 | Dependencias incompatibles | Bajo | Baja | Versiones fijas en template base |
| R6 | CSS inconsistente entre componentes | Medio | Alta | Variables CSS globales; system prompt exige consistencia |
| R7 | Responsive no funciona | Alto | Media | System prompt exige mobile-first con media queries 768px |
| R8 | Imágenes no encontradas | Bajo | Baja | Fallback a placeholders |
| R9 | Usuario no entiende el flujo | Alto | Media | README por proyecto; mensajes claros en scripts |
| R10 | Cambio de schema brief.json | Medio | Baja | `_schema_version` en JSON; migración en scripts |

---

## 15. Recomendaciones de Mantenibilidad

### Para el Equipo de Desarrollo

| Recomendación | Descripción |
|---------------|-------------|
| Versionar prompts | Los archivos en `prompts/` son código. Revisar en PRs |
| Testear con briefs de ejemplo | Mantener `examples/` con briefs de prueba |
| Schema con validación | Validar JSON al cargar; error claro si falta campo |
| Logs de generación | Timestamp, modelo usado, tokens consumidos por proyecto |
| Gitignore correcto | No commitear `node_modules/`, `.astro/`, `dist/dist/` |

### Para el Usuario Final

| Recomendación | Descripción |
|---------------|-------------|
| README por proyecto | Cada `sites/[nombre]/README.md` explica su propio flujo |
| brief.json editable | Corregir sin repetir cuestionario |
| Assets con nombres claros | Documentar qué archivos poner en `assets-input/` |
| Preview antes de deploy | Siempre previsualizar antes de desplegar |

### .gitignore Recomendado

```gitignore
# No commitear dependencias instaladas
sites/*/dist/node_modules/
sites/*/dist/.astro/
sites/*/dist/dist/

# Secretos
.env
.env.local
```

---

## 16. Hoja de Ruta de Implementación por Fases

### Fase 1: Infraestructura Base (Semana 1-2)

| Tarea | Archivo |
|-------|---------|
| Crear estructura de carpetas | `scripts/`, `prompts/`, `templates/`, `sites/` |
| Implementar crear-proyecto.sh | `scripts/crear-proyecto.sh` |
| Implementar recopilar-info.sh | `scripts/recopilar-info.sh` |
| Crear template base Astro+React | `templates/astro-react-base/` |
| Escribir system prompt | `prompts/00-system-core.md` |

### Fase 2: Prompts y Generación (Semana 3-4)

| Tarea | Archivo |
|-------|---------|
| Escribir prompts 01-07 | `prompts/01-structure.md` ... `07-assemble.md` |
| Implementar generar-sitio.sh | `scripts/generar-sitio.sh` |
| Implementar parser de respuestas IA | Función `parse_ai_response()` |

### Fase 3: Preview y Testing (Semana 5)

| Tarea | Archivo |
|-------|---------|
| Implementar previsualizar.sh | `scripts/previsualizar.sh` |
| Crear 3 briefs de prueba | `examples/` |
| Testing end-to-end con Ollama | Manual |
| Iterar prompts basados en resultados | `prompts/*.md` |

### Fase 4: Refinamiento (Semana 6+)

| Tarea | Descripción |
|-------|-------------|
| Mejorar prompts | Ajustar donde la IA falle |
| Validación de brief.json | JSON Schema |
| Regenerar secciones individuales | `generar-sitio.sh --section hero` |
| deploy-prepare.sh | Preparar paquete por plataforma |
| Documentación completa | README.md guía paso a paso |
| Internacionalización | Cuestionario en múltiples idiomas |

---

## Apéndice A: Resumen de Scripts

| Script | Función | Entrada | Salida |
|--------|---------|---------|--------|
| `crear-proyecto.sh` | Crea estructura de carpetas | Nombre del proyecto | `sites/[nombre]/` vacío |
| `recopilar-info.sh` | Cuestionario secuencial | Nombre del proyecto | `sites/[nombre]/brief.json` |
| `generar-sitio.sh` | Genera sitio con IA | `brief.json` + prompts | `sites/[nombre]/dist/` build |
| `previsualizar.sh` | Servidor local temporal | Nombre del proyecto | http://localhost:8080 |

---

## Apéndice B: Paletas de Color Predefinidas

| Paleta | Primario | Secundario | Fondo | Texto |
|--------|----------|------------|-------|-------|
| warm | `#2C1810` | `#D4A574` | `#1a1209` | `#F5E6D3` |
| cool | `#1E3A5F` | `#60A5FA` | `#0F172A` | `#E2E8F0` |
| vibrant | `#7C3AED` | `#EC4899` | `#0F0A1A` | `#F1E8FF` |
| monochrome | `#374151` | `#9CA3AF` | `#111827` | `#F9FAFB` |
| nature | `#065F46` | `#10B981` | `#0A1A14` | `#D1FAE5` |
| pastel | `#A78BFA` | `#F9A8D4` | `#1E1B2E` | `#EDE9FE` |

---

## Apéndice C: Checklist de Deploy por Plataforma

### GitHub Pages
- [ ] Build en `dist/dist/`
- [ ] Copiar contenido a rama `gh-pages`
- [ ] Configurar en Settings → Pages → Source: gh-pages

### Cloudflare Pages
- [ ] Build en `dist/dist/`
- [ ] Dashboard → Pages → Create → Upload folder → seleccionar `dist/dist/`

### Netlify
- [ ] Build en `dist/dist/`
- [ ] Dashboard → Sites → Add new → Manual deploy → subir `dist/dist/`

### Vercel
- [ ] Conectar repositorio
- [ ] Build command: `cd sites/mi-cafeteria/dist && npm run build`
- [ ] Output directory: `sites/mi-cafeteria/dist/dist`

### FTP
- [ ] Build en `dist/dist/`
- [ ] Conectar con FileZilla/WinSCP
- [ ] Subir contenido de `dist/dist/` a `public_html/` o `www/`

### Railway
- [ ] Conectar repositorio
- [ ] Root directory: `sites/mi-cafeteria/dist`
- [ ] Build command: `npm run build`
- [ ] Start command: `npx serve dist -p $PORT`

---

*Documento generado el 7 de abril de 2026. Basado en LocalSite-ai v0.5.2, Astro v5.x, React v19.*
