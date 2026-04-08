# DOCUMENTO DE TRABAJO — Cuestionarios y Estructura de Datos para One-Page Site Builder

> **ESTADO**: Borrador inicial — documento de trabajo para revisión
> **PROYECTO**: Astro-LocalSite-ai One-Page Builder
> **BASE**: LocalSite-ai v0.5.2 + Astro 5.x + React 19
> **FECHA**: 7 de abril de 2026

---

## Tabla de Contenidos

1. [Decisiones Confirmadas](#1-decisiones-confirmadas)
2. [Estructura de Carpetas](#2-estructura-de-carpetas)
3. [Bloques de Cuestionarios C-MD](#3-bloques-de-cuestionarios-c-md)
4. [C-MD 1: perfil-mi-negocio](#4-c-md-1-perfil-mi-negocio)
5. [C-MD 2: proyecto-mi-negocio](#5-c-md-2-proyecto-mi-negocio)
6. [C-MD 3: estructura-mi-negocio](#6-c-md-3-estructura-mi-negocio)
7. [material-req.md](#7-material-reqmd)
8. [FAQs — archivo independiente](#8-faqs--archivo-independiente)
9. [Archivos de Contenido (Fase 2)](#9-archivos-de-contenido-fase-2)
10. [JSON Resultantes](#10-json-resultantes)
11. [API Keys y .env](#11-api-keys-y-env)
12. [Flujo de Trabajo Completo](#12-flujo-de-trabajo-completo)
13. [Scripts Necesarios](#13-scripts-necesarios)
14. [Templates por Proyecto](#14-templates-por-proyecto)
15. [Notas Técnicas y Pendientes](#15-notas-técnicas-y-pendientes)

---

## 1. Decisiones Confirmadas

| # | Decisión | Elección |
|---|----------|----------|
| 1 | Formato C-MD | **A — Formularios rellenables** en el propio `.md` (checkboxes, líneas, espacios) |
| 2 | C-MD → JSON | **Asistente IA posterior** ejecutado manualmente por el usuario desde terminal |
| 3 | Archivos de contenido | **C — Mixto**: `.md` para contenido puro, `.astro` para layouts |
| 4 | Templates | **A — Copia por proyecto**: cada proyecto tiene `templates/mi-negocio/` |
| 5 | Proveedores IA | **Todos 9**: DeepSeek, OpenRouter, Anthropic, Google, Mistral, Cerebras, OpenAI-compatible, Ollama, LM Studio |
| 6 | material-req.md | **B — Checklist + Preguntas E y J** con espacio para responder |
| 7 | Orden del flujo | **Correcto** (7 pasos sin cambios) |
| 8 | FAQs | **B — Markdown P/R** que luego se convierte a JSON-LD FAQPage Schema |
| 9 | Límite contenido | Usuario tiene texto preparado; sistema genera **placeholders**; sistema propone secciones; usuario rellena/modifica/agrega |
| 10 | Ejecución scripts | **Terminal directa** (bash scripts) |

---

## 2. Estructura de Carpetas

```
LocalSite-ai/                                  # Raíz del repositorio
│
├── LocalSite-ai/                              # App Next.js existente (INTACTA)
│   └── ...
│
├── scripts/
│   └── onepage/                               # Scripts específicos para onepage
│       ├── crear-proyecto.sh
│       ├── generar-cuestionarios.sh           # Genera C-MD en data-usr/
│       ├── generar-esqueleto-contenido.sh     # Fase 2: crea archivos de contenido
│       ├── convertir-json.sh                  # Ejecuta asistente IA → C-MD a JSON
│       ├── generar-sitio.sh                   # Ejecuta prompts IA → genera Astro
│       └── previsualizar.sh
│
├── prompts/
│   └── onepage/                               # Prompts específicos para onepage
│       ├── 00-system-core.md
│       ├── 01-structure.md
│       ├── 02-hero.md
│       ├── 03-sections.md
│       ├── 04-interactivity.md
│       ├── 05-styling.md
│       ├── 06-seo-meta.md
│       └── 07-assemble.md
│
├── templates/                                 # Templates base (tipos reutilizables)
│   ├── base-astro-react/                      # Template base mínimo
│   │   ├── astro.config.mjs
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── public/
│   │   └── src/
│   │       ├── layouts/
│   │       │   └── Layout.astro
│   │       ├── pages/
│   │       │   └── index.astro
│   │       └── styles/
│   │           └── global.css
│   └── ...                                    # Futuros: business/, portfolio/, etc.
│
├── sites/
│   └── mi-cafeteria/                          # Proyecto generado
│       │
│       ├── templates/                         # COPIA del template asignado a este proyecto
│       │   ├── astro.config.mjs
│       │   ├── package.json
│       │   └── src/
│       │       ├── layouts/
│       │       ├── pages/
│       │       └── styles/
│       │
│       ├── data-usr/                          # DONDE el usuario rellena C-MD y contenido
│       │   │
│       │   ├── C-MD/                          # Cuestionarios rellenables
│       │   │   ├── perfil.md                  # C-MD 1: Perfil del usuario
│       │   │   ├── proyecto.md                # C-MD 2: Info del proyecto
│       │   │   ├── estructura.md              # C-MD 3: Estructura técnica
│       │   │   ├── material-req.md            # Materiales requeridos (branding + assets)
│       │   │   └── faq.md                     # FAQs en formato P/R
│       │   │
│       │   └── contenido/                     # Fase 2: Archivos de contenido (esqueleto)
│       │       ├── hero.md                    # Título, subtítulo, CTA del hero
│       │       ├── about.md                   # Texto "Sobre nosotros"
│       │       ├── servicios.md               # Lista de servicios
│       │       ├── testimonials.md            # Testimonios
│       │       ├── contact-layout.astro       # Layout del formulario de contacto
│       │       └── ...                        # Otros archivos de contenido
│       │
│       ├── data-json/                         # JSON generados por asistente IA
│       │   ├── perfil.json
│       │   ├── proyecto.json
│       │   └── estructura.json
│       │
│       ├── assets-input/                      # Assets del usuario (imágenes, logo, etc.)
│       │   ├── logo.png
│       │   ├── hero-image.jpg
│       │   └── ...
│       │
│       ├── .env                               # API keys del proveedor IA (gitignore)
│       ├── .env.example                       # Plantilla de variables necesarias
│       └── dist/                              # RESULTADO FINAL: sitio build, deploy-ready
│           ├── astro.config.mjs
│           ├── package.json
│           ├── src/
│           │   ├── layouts/
│           │   ├── pages/
│           │   ├── components/
│           │   └── styles/
│           ├── public/
│           ├── node_modules/
│           └── dist/                          # Build final → listo para deploy
│               ├── index.html
│               ├── _astro/
│               └── assets/
│
└── temp/                                      # Documentos de trabajo
    ├── Preguntas-Aclaracion-OnePage-Builders.md
    └── BOCADILLO-Cuestionarios-OnePage.md     # ESTE ARCHIVO
```

### Reglas Clave

| Regla | Descripción |
|-------|-------------|
| `scripts/onepage/` | Subdir específico; futuro: `scripts/blog/`, `scripts/ecommerce/`, etc. |
| `prompts/onepage/` | Idem; prompts específicos para onepage |
| `data-usr/` | Único lugar donde el usuario trabaja (rellena C-MD, contenido, assets) |
| `data-json/` | Solo lectura para el usuario; generado por asistente IA |
| `templates/` (proyecto) | Cada proyecto tiene SU COPIA del template, modificable independientemente |
| `.env` (proyecto) | API keys aisladas por proyecto; NO se commitea |
| `dist/dist/` | Build final listo para cualquier plataforma de deploy |

---

## 3. Bloques de Cuestionarios C-MD

### Filosofía de los 3 Bloques

| Bloque | Archivo C-MD | JSON Resultante | Propósito |
|--------|-------------|-----------------|-----------|
| **Perfil** | `perfil.md` | `perfil.json` | Info del usuario reutilizable en futuros proyectos (identidad, contacto, redes) |
| **Proyecto** | `proyecto.md` | `proyecto.json` | Info específica de este proyecto (nombre, objetivo, contenido, SEO) |
| **Estructura** | `estructura.md` | `estructura.json` | Datos técnicos para el desarrollador/IA (secciones, interactividad, deploy, accesibilidad) |

### Archivos Adicionales

| Archivo | Propósito | JSON asociado |
|---------|-----------|---------------|
| `material-req.md` | Checklist de materiales + preguntas branding (E) y assets (J) | Se integra en `estructura.json` |
| `faq.md` | FAQs en formato P/R para conversión posterior a JSON-LD | FAQPage Schema separado |

---

## 4. C-MD 1: perfil-mi-negocio

> **Archivo**: `sites/mi-cafeteria/data-usr/C-MD/perfil.md`
> **Propósito**: Cuenta de usuario — info reutilizable en futuros proyectos

```markdown
# C-MD 1: Perfil — Tu Información Personal/Empresarial

> Este cuestionario recopila información sobre ti o tu empresa que podrás
> reutilizar en futuros proyectos. Completa lo que aplique.

---

## Identidad

### P1. Nombre completo o nombre de la empresa
<!-- Escribe tu nombre o el de tu empresa -->

_______________________________________________________

### P2. Tipo de perfil
<!-- Marca una opción -->

- [ ] Profesional autónomo / Freelancer
- [ ] Pequeña empresa
- [ ] Mediana empresa
- [ ] Startup
- [ ] Organización sin ánimo de lucro
- [ ] Otro: ___________________________________________

### P3. Sector o industria principal
<!-- Ejemplo: hostelería, tecnología, educación, salud, etc. -->

_______________________________________________________

---

## Información de Contacto

### P4. Email principal de contacto
<!-- El email que usarás profesionalmente -->

_______________________________________________________

### P5. Email alternativo (opcional)
<!-- Segundo email si tienes uno personal y otro profesional -->

_______________________________________________________

### P6. Teléfono principal
<!-- Con código de país: +34, +52, +1, etc. -->

_______________________________________________________

### P7. Dirección fiscal o de negocio (opcional)
<!-- Calle, número, ciudad, código postal, país -->

_______________________________________________________

_______________________________________________________

---

## Redes Sociales y Presencia Online

### P8. Sitio web principal (si ya tienes uno)
<!-- URL de tu web actual, si existe -->

_______________________________________________________

### P9. Instagram
<!-- Tu @ o URL completa -->

_______________________________________________________

### P10. Facebook
<!-- URL de tu página de Facebook -->

_______________________________________________________

### P11. Twitter / X
<!-- Tu @ o URL completa -->

_______________________________________________________

### P12. LinkedIn
<!-- URL de tu perfil o página de empresa -->

_______________________________________________________

### P13. TikTok
<!-- Tu @ o URL completa -->

_______________________________________________________

### P14. YouTube (opcional)
<!-- URL de tu canal -->

_______________________________________________________

### P15. Otras plataformas o enlaces relevantes
<!-- WhatsApp, Telegram, Pinterest, etc. -->

_______________________________________________________

_______________________________________________________

---

## Idioma y Localización

### P16. Idioma principal
<!-- Marca una opción -->

- [ ] Español
- [ ] Inglés
- [ ] Portugués
- [ ] Francés
- [ ] Italiano
- [ ] Alemán
- [ ] Otro: ___________________________________________

### P17. Zona geográfica principal
<!-- Ejemplo: Madrid, España / CDMX, México / Latinoamérica / Global -->

_______________________________________________________

---

## Notas Adicionales

### P18. ¿Hay algo más que quieras que sepamos sobre tu perfil?
<!-- Información adicional que pueda ser relevante para futuros proyectos -->

_______________________________________________________

_______________________________________________________

---

✅ **Has completado el C-MD 1: Perfil**

Próximo paso: completa `proyecto.md`
```

### JSON Resultante: `perfil.json`

```json
{
  "_schema_version": "1.0.0",
  "_source": "C-MD perfil.md",
  "identidad": {
    "nombre": "",
    "tipo_perfil": "",
    "sector": ""
  },
  "contacto": {
    "email_principal": "",
    "email_alternativo": "",
    "telefono": "",
    "direccion": ""
  },
  "redes_sociales": {
    "web_principal": "",
    "instagram": "",
    "facebook": "",
    "twitter": "",
    "linkedin": "",
    "tiktok": "",
    "youtube": "",
    "otros": ""
  },
  "localizacion": {
    "idioma_principal": "",
    "zona_geografica": ""
  },
  "notas_adicionales": ""
}
```

---

## 5. C-MD 2: proyecto-mi-negocio

> **Archivo**: `sites/mi-cafeteria/data-usr/C-MD/proyecto.md`
> **Propósito**: Ficha del proyecto — info específica para este onepage

```markdown
# C-MD 2: Proyecto — Información de "mi-cafeteria"

> Este cuestionario recopila información específica de este proyecto web.
> Cada respuesta ayuda a definir cómo será tu sitio onepage.

---

## Ficha del Proyecto

### P1. Nombre del proyecto / sitio web
<!-- El nombre que aparecerá en el título del navegador y en Google -->

_______________________________________________________

### P2. Eslogan o frase descriptiva (opcional)
<!-- Una frase corta que resuma tu propuesta de valor -->

_______________________________________________________

### P3. URL del sitio (si ya la tienes pensada)
<!-- Ejemplo: www.micafe.com -->

_______________________________________________________

---

## Objetivo del Sitio

### P4. ¿Cuál es el objetivo principal?
<!-- Puedes marcar más de una -->

- [ ] Mostrar información del negocio
- [ ] Captar clientes potenciales (leads)
- [ ] Vender un producto o servicio
- [ ] Mostrar portfolio o trabajo
- [ ] Recibir reservas o citas
- [ ] Construir marca o presencia online
- [ ] Informar o educar
- [ ] Otro: ___________________________________________

### P5. Describe en 2-3 frases qué quieres lograr con esta web
<!-- Ejemplo: Quiero que los visitantes conozcan mi cafetería, vean el menú y reserven mesa -->

_______________________________________________________

_______________________________________________________

### P6. ¿Cuál es la acción MÁS importante que quieres que haga el visitante?
<!-- Ejemplo: Reservar una mesa, Comprar ahora, Llamarnos -->

_______________________________________________________

---

## Contenido del Sitio

> **IMPORTANTE**: Este proyecto se encarga de la CONSTRUCCIÓN del sitio,
> no de la elaboración del contenido. Debes tener preparado el texto
> que irá en cada sección. Los archivos para rellenar el contenido
> se generarán después de completar este cuestionario.

### P7. ¿Qué secciones quieres en tu página onepage?
<!-- Marca las que apliquen. Hero y Footer siempre están incluidas. -->

- [x] hero (siempre incluida — sección principal de entrada)
- [ ] about (sobre nosotros / quiénes somos)
- [ ] services (servicios / qué ofrecemos)
- [ ] features (características / funcionalidades)
- [ ] pricing (precios / planes)
- [ ] testimonials (testimonios / reseñas)
- [ ] faq (preguntas frecuentes)
- [ ] gallery (galería / portfolio)
- [ ] team (equipo / quiénes somos)
- [ ] stats (estadísticas / contadores / números)
- [ ] newsletter (suscripción a newsletter)
- [ ] contact (formulario de contacto)
- [ ] map (mapa de ubicación)
- [ ] cta_banner (banner de llamada a la acción intermedio)
- [ ] logos (logos de clientes/partners)
- [x] footer (siempre incluido — pie de página)

### P8. ¿Tienes ya preparados los textos para cada sección?
<!-- El contenido se rellenará en archivos separados que se generarán después -->

- [ ] Sí, tengo todos los textos preparados
- [ ] Tengo algunos textos, otros los prepararé después
- [ ] No tengo textos preparados, los prepararé más adelante

### P9. Descripción breve del contenido que irá en cada sección elegida
<!-- Para cada sección que marcaste arriba, describe brevemente qué contenido llevará -->

**Hero:**
_______________________________________________________

**About (si aplica):**
_______________________________________________________

**Services (si aplica):**
_______________________________________________________

**Pricing (si aplica):**
_______________________________________________________

**Testimonials (si aplica):**
_______________________________________________________

**FAQ (si aplica):**
_______________________________________________________

**Gallery (si aplica):**
_______________________________________________________

**Contact (si aplica):**
_______________________________________________________

**Otras secciones:**
_______________________________________________________

---

## SEO y Metadatos

### P10. Título SEO para Google (50-60 caracteres)
<!-- El título que aparecerá en los resultados de búsqueda de Google -->

_______________________________________________________

### P11. Descripción SEO para Google (150-160 caracteres)
<!-- La descripción breve que aparecerá bajo el título en Google -->

_______________________________________________________

### P12. Palabras clave principales
<!-- Separadas por comas. Ejemplo: café especialidad, cafetería Madrid, café artesanal -->

_______________________________________________________

### P13. Open Graph — Título para redes sociales
<!-- El título que se verá al compartir en Facebook, WhatsApp, etc. -->

_______________________________________________________

### P14. Open Graph — Descripción para redes sociales
<!-- La descripción breve al compartir en redes -->

_______________________________________________________

### P15. Open Graph — Imagen para compartir
<!-- Nombre del archivo de imagen en assets-input/ que se usará al compartir -->
<!-- Recomendendación: 1200x630px, formato JPG o PNG -->

_______________________________________________________

### P16. URL canónica del sitio (opcional)
<!-- Si el sitio tendrá varias URLs, indica cuál es la principal -->

_______________________________________________________

### P17. Idioma del sitio HTML
<!-- Código de idioma: es, en, pt, fr, it, de, etc. -->

_______________________________________________________

---

## Estadísticas y Métricas (Opcional)

### P18. ¿Quieres mostrar estadísticas o contadores en la web?
<!-- Ejemplo: "Más de 500 clientes satisfechos", "10 años de experiencia" -->

- [ ] No
- [ ] Sí → Indica los datos que quieres mostrar:

| Dato | Valor |
|------|-------|
| | |
| | |
| | |

---

## Notas del Proyecto

### P19. ¿Hay algo más que quieras especificar sobre este proyecto?
<!-- Cualquier detalle adicional que no haya sido preguntado -->

_______________________________________________________

_______________________________________________________

---

✅ **Has completado el C-MD 2: Proyecto**

Próximo paso: completa `estructura.md`
```

### JSON Resultante: `proyecto.json`

```json
{
  "_schema_version": "1.0.0",
  "_source": "C-MD proyecto.md",
  "ficha": {
    "nombre": "",
    "eslogan": "",
    "url_pensada": ""
  },
  "objetivo": {
    "objetivos": [],
    "descripcion": "",
    "accion_principal": ""
  },
  "contenido": {
    "textos_preparados": "",
    "descripcion_secciones": {
      "hero": "",
      "about": "",
      "services": "",
      "pricing": "",
      "testimonials": "",
      "faq": "",
      "gallery": "",
      "contact": "",
      "otras": ""
    }
  },
  "seo": {
    "titulo": "",
    "descripcion": "",
    "keywords": "",
    "og_titulo": "",
    "og_descripcion": "",
    "og_imagen": "",
    "url_canonica": "",
    "idioma_html": ""
  },
  "estadisticas": {
    "tiene": false,
    "datos": []
  },
  "notas": ""
}
```

---

## 6. C-MD 3: estructura-mi-negocio

> **Archivo**: `sites/mi-cafeteria/data-usr/C-MD/estructura.md`
> **Propósito**: Datos técnicos que el desarrollador/IA necesita para construir el sitio

```markdown
# C-MD 3: Estructura Técnica — Requisitos para Construir el Sitio

> Este cuestionario recopila la información técnica que necesita el sistema
> para generar tu sitio web onepage con Astro + React.

---

## Proveedor de IA para Generación

### P1. ¿Qué proveedor de IA quieres usar para generar el sitio?
<!-- El sistema usará este proveedor para ejecutar los prompts de generación -->

- [ ] Ollama (local, gratuito — requiere Ollama instalado)
- [ ] DeepSeek (cloud, económico — necesita API key)
- [ ] OpenRouter (cloud, 400+ modelos — necesita API key)
- [ ] Anthropic Claude (cloud, premium — necesita API key)
- [ ] Google Gemini (cloud — necesita API key)
- [ ] Mistral (cloud — necesita API key)
- [ ] Cerebras (cloud, ultra-rápido — necesita API key)
- [ ] OpenAI-compatible (cualquier API compatible con OpenAI)
- [ ] LM Studio (local — requiere LM Studio corriendo)

### P2. Modelo de IA preferido (opcional)
<!-- Si conoces el nombre del modelo, escríbelo. Si no, se usará el por defecto -->

_______________________________________________________

### P3. API Key del proveedor seleccionado
<!-- Si tu proveedor requiere API key, escríbela aquí. Se guardará de forma segura en .env -->
<!-- Si no tienes la API key, puedes dejarlo vacío y configurarlo después -->

_______________________________________________________

### P4. API Base URL (solo para proveedores personalizados)
<!-- Si usas OpenAI-compatible u otro proveedor custom, indica la URL base -->
<!-- Ejemplo: https://api.openai.com/v1 -->

_______________________________________________________

---

## Estructura del Sitio

### P5. Secciones seleccionadas
<!-- Copia aquí las secciones que marcaste en C-MD 2 (P7) -->
<!-- Ejemplo: hero, about, services, testimonials, faq, contact, footer -->

_______________________________________________________

### P6. Orden de las secciones
<!-- Indica el orden en que deben aparecer (de arriba a abajo) -->
<!-- Ejemplo: hero → about → services → testimonials → faq → contact → footer -->

_______________________________________________________

---

## Interactividad

### P7. ¿Qué elementos necesitan interactividad (JavaScript/React)?
<!-- Marca los que apliquen -->

- [ ] Menú hamburguesa en móvil
- [ ] Toggle mensual/anual en precios
- [ ] Acordeón desplegable en FAQ
- [ ] Formulario de contacto con validación
- [ ] Contadores animados al hacer scroll
- [ ] Galería con lightbox
- [ ] Carrusel/slider
- [ ] Tabs o pestañas
- [ ] Otro: ___________________________________________

### P8. ¿Prefieres animaciones sutiles o llamativas?

- [ ] Sutiles (transiciones suaves, fade-in al scroll)
- [ ] Llamativas (animaciones más elaboradas, efectos visuales)
- [ ] Sin animaciones (sitio estático)

---

## Accesibilidad (WCAG)

### P9. ¿Necesitas que el sitio cumpla con estándares de accesibilidad WCAG?
<!-- WCAG = Web Content Accessibility Guidelines -->
<!-- Son normas para que la web sea accesible a personas con discapacidad -->

- [ ] No es necesario
- [ ] Nivel A (mínimo — elementos básicos de accesibilidad)
- [ ] Nivel AA (recomendado — equilibrio entre accesibilidad y esfuerzo)
- [ ] Nivel AAA (máximo — accesibilidad completa)

> **¿Qué significa cada nivel?**
> - **Nivel A**: Contraste básico, etiquetas alt en imágenes, navegación por teclado fundamental
> - **Nivel AA**: Contraste mejorado (4.5:1), estructura semántica, labels en formularios, indicadores de foco (recomendado para la mayoría de sitios)
> - **Nivel AAA**: Contraste máximo (7:1), lenguaje claro, sin dependencia del color, alternativas para todo contenido multimedia

---

## Analytics y Seguimiento

### P10. ¿Quieres integrar analytics en tu sitio?
<!-- Herramientas para medir visitas, comportamiento de usuarios, etc. -->

- [ ] No
- [ ] Google Analytics 4 (GA4)
- [ ] Microsoft Clarity / Bing Webmaster
- [ ] Plausible.io (privacy-friendly)
- [ ] Matomo (self-hosted o cloud)
- [ ] Otro: ___________________________________________

### P11. ID de seguimiento
<!-- Si seleccionaste alguna opción, pega aquí tu ID de seguimiento -->
<!-- Ejemplo GA4: G-XXXXXXXXXX | Plausible: tu-dominio.com -->

_______________________________________________________

---

## Deploy

### P12. ¿Dónde planeas desplegar este sitio?
<!-- Esto ayuda a preparar la configuración correcta -->

- [ ] GitHub Pages
- [ ] Cloudflare Pages
- [ ] Netlify
- [ ] Vercel
- [ ] Railway
- [ ] FTP a alojamiento compartido
- [ ] Aún no lo sé

### P13. ¿Tienes ya un dominio comprado?
<!-- Si sí, indica cuál -->

- [ ] No
- [ ] Sí: ___________________________________________

---

## Notas Técnicas

### P14. ¿Hay algún requisito técnico específico?
<!-- Ejemplo: necesito que el sitio cargue en menos de 2 segundos,
     necesito compatibilidad con IE11, etc. -->

_______________________________________________________

_______________________________________________________

---

✅ **Has completado el C-MD 3: Estructura Técnica**

Próximo paso: completa `material-req.md`
```

### JSON Resultante: `estructura.json`

```json
{
  "_schema_version": "1.0.0",
  "_source": "C-MD estructura.md",
  "ia_provider": {
    "proveedor": "",
    "modelo": "",
    "api_key_env": "PROJECT_API_KEY",
    "api_base_url": ""
  },
  "estructura": {
    "secciones": [],
    "orden": []
  },
  "interactividad": {
    "elementos": [],
    "animaciones": ""
  },
  "accesibilidad": {
    "wcag_nivel": ""
  },
  "analytics": {
    "proveedor": "",
    "tracking_id": ""
  },
  "deploy": {
    "plataforma": "",
    "dominio": ""
  },
  "notas_tecnicas": ""
}
```

---

## 7. material-req.md

> **Archivo**: `sites/mi-cafeteria/data-usr/C-MD/material-req.md`
> **Propósito**: Checklist de materiales + preguntas de Branding (E) y Assets (J) con espacio para responder

```markdown
# Material Requerido — Branding y Assets

> Este documento tiene dos funciones:
> 1. Listar todo el material (imágenes, archivos) que necesitas proporcionar
> 2. Recopilar tus preferencias de branding y diseño
>
> Todos los archivos mencionados aquí deben colocarse en:
> **`sites/mi-cafeteria/assets-input/`**

---

## PARTE A: Branding y Estilo Visual

### E1. Logotipo

- [ ] Sí, tengo logotipo
- [ ] No tengo logotipo (se usará texto como logo)

**Si tienes logotipo:**
- Nombre del archivo en `assets-input/`: ___________________________
- Formato preferido: [ ] PNG  [ ] SVG  [ ] WebP

### E2. Paleta de Colores

**Elige tu paleta base** (marca una):

- [ ] **Cálidos** — Marrones, dorados, cremas (restaurante, café, artesanía)
- [ ] **Fríos** — Azules, grises, blancos (tecnología, corporativo)
- [ ] **Vibrantes** — Gradientes llamativos, púrpuras, rosas (startup, creativo)
- [ ] **Monocromático** — Blanco, negro, grises (minimalista, elegante)
- [ ] **Naturaleza** — Verdes, tierras (ecología, salud, bienestar)
- [ ] **Pastel** — Tonos suaves, delicados (infantil, belleza)
- [ ] **Personalizados** — Indica tus colores exactos abajo

**Si elegiste "Personalizados" o quieres ajustar:**

| Uso | Código Hex | Nombre (opcional) |
|-----|------------|-------------------|
| Color primario | #____________ | |
| Color secundario | #____________ | |
| Color de fondo | #____________ | |
| Color de texto | #____________ | |
| Color de acento | #____________ | |

### E3. Estilo Visual

**Elige el estilo que más te representa** (marca uno):

- [ ] **Minimal** — Minimalista y limpio (pocos elementos, mucho espacio)
- [ ] **Modern** — Moderno con gradientes y sombras suaves
- [ ] **Classic** — Clásico y elegante (tipografías serif, simetría)
- [ ] **Bold** — Atrevido y llamativo (colores fuertes, tipografías grandes)
- [ ] **Playful** — Divertido y colorido (formas orgánicas, colores vivos)
- [ ] **Elegant** — Elegante y sofisticado (tonos oscuros, dorados)
- [ ] **Rustic** — Rústico y artesanal (texturas, tonos tierra)

### E4. Fondo del Sitio

- [ ] Claro (fondo blanco o muy claro)
- [ ] Oscuro (fondo negro o muy oscuro)

### E5. Tipografía

**¿Prefieres algún tipo de tipografía?**

- [ ] Sans-serif moderna (limpia, legible, contemporánea)
- [ ] Serif elegante (clásica, editorial, formal)
- [ ] Display llamativa (para títulos, impactante)
- [ ] Monospace (técnica, code, hacker)
- [ ] Sin preferencia (el sistema elegirá)

**Si tienes una fuente específica en mente** (Google Fonts):
- Fuente para títulos: ___________________________
- Fuente para cuerpo de texto: ___________________________

### E6. Web de Referencia

**¿Hay algún sitio web cuyo diseño te guste como referencia?**
<!-- URL de un sitio cuyo estilo visual te gustaría emular -->

_______________________________________________________

**¿Qué te gusta de ese sitio?**

_______________________________________________________

---

## PARTE B: Assets y Archivos Requeridos

### Instrucciones Generales

1. Coloca TODOS los archivos en: **`sites/mi-cafeteria/assets-input/`**
2. Usa nombres descriptivos: `logo.png`, `hero-cafe.jpg`, `gallery-01.webp`
3. Formatos recomendidos:
   - Imágenes: **WebP** (mejor) o **JPG/PNG**
   - Logo: **SVG** (mejor) o **PNG** con fondo transparente
   - Favicon: **ICO** o **PNG** 32x32px
   - OG Image: **JPG** 1200x630px

### Checklist de Archivos

#### Logo
- [ ] Archivo proporcionado: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Fondo transparente (si aplica)

#### Imagen Principal del Hero
- [ ] Archivo proporcionado: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Resolución mínima recomendada: 1920x1080px
- [ ] Si no tienes, se usará un placeholder

#### Galería de Imágenes (si aplica)
- [ ] Archivo 1: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Archivo 2: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Archivo 3: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Archivo 4: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Archivo 5: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Archivo 6: `sites/mi-cafeteria/assets-input/_________________`

#### Fotos del Equipo (si aplica)
- [ ] Persona 1: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Persona 2: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Persona 3: `sites/mi-cafeteria/assets-input/_________________`

#### Favicon
- [ ] Archivo proporcionado: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Tamaño: 32x32px (ICO) o 512x512px (PNG)

#### Imagen para Open Graph (compartir en redes)
- [ ] Archivo proporcionado: `sites/mi-cafeteria/assets-input/_________________`
- [ ] Tamaño recomendado: 1200x630px

#### Otros Archivos
- [ ] Archivo: `sites/mi-cafeteria/assets-input/_________________` — Uso: ____________
- [ ] Archivo: `sites/mi-cafeteria/assets-input/_________________` — Uso: ____________

---

## Resumen de Materiales

### Archivos proporcionados:

| # | Nombre del archivo | Tipo | Uso en el sitio |
|---|-------------------|------|-----------------|
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |
| 6 | | | |

### Archivos que aún necesito preparar:

| # | Descripción | Formato | Tamaño recomendado |
|---|-------------|---------|-------------------|
| 1 | | | |
| 2 | | | |

---

✅ **Has completado Material Requerido**

Próximo paso: completa `faq.md` (si aplica)
```

---

## 8. FAQs — Archivo Independiente

> **Archivo**: `sites/mi-cafeteria/data-usr/C-MD/faq.md`
> **Formato**: Markdown P/R → conversión posterior a JSON-LD FAQPage Schema
> **Obligatorio**: No

```markdown
# Preguntas Frecuentes (FAQ)

> **OPCIONAL**: Solo completa este archivo si necesitas una sección de FAQ
> en tu sitio web.
>
> **Formato**: Escribe cada pregunta precedida de `P:` y la respuesta
> precedida de `R:`. Deja una línea en blanco entre cada par P/R.
>
> **Nota**: Estas FAQs se convertirán automáticamente a formato JSON-LD
> FAQPage Schema de Schema.org para que Google las muestre en los
> resultados de búsqueda.

---

## Instrucciones

1. Escribe `P:` seguido de tu pregunta
2. En la siguiente línea, escribe `R:` seguido de tu respuesta
3. Deja una línea en blanco entre cada par
4. Puedes añadir tantas FAQs como necesites

### Ejemplo

```
P: ¿Aceptan reservas?
R: Sí, puedes reservar por teléfono llamando al +34 912 345 678 o a
través de nuestro formulario de contacto en esta misma web.

P: ¿Cuál es el horario?
R: Lunes a Viernes de 8:00 a 20:00. Sábados de 9:00 a 21:00.
Domingos cerrado.
```

---

## Tus Preguntas Frecuentes

P:

R:

---

P:

R:

---

P:

R:

---

P:

R:

---

P:

R:

---

P:

R:

---

P:

R:

---

P:

R:

---

✅ **Has completado las FAQs** (o las has saltado)

---

## Referencia: FAQPage Schema

Las FAQs que proporciones aquí se convertirán en este formato JSON-LD:

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "¿Aceptan reservas?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Sí, puedes reservar por teléfono..."
      }
    }
  ]
}
```

Este formato ayuda a que Google muestre tus preguntas directamente
en los resultados de búsqueda (rich snippets).
```

---

## 9. Archivos de Contenido (Fase 2)

> **Ubicación**: `sites/mi-cafeteria/data-usr/contenido/`
> **Momento**: Se generan DESPUÉS de completar los C-MD y obtener el JSON
> **Formato**: Mixto — `.md` para contenido puro, `.astro` para layouts

### Filosofía

El sistema **propone secciones** basado en lo que el usuario marcó en C-MD 2.
Luego genera **archivos esqueleto** con placeholders que el usuario rellena
con su contenido preparado.

### Archivos Generados (Ejemplo)

Si el usuario marcó: hero, about, services, testimonials, faq, contact, footer

```
data-usr/contenido/
├── hero.md                    # Título, subtítulo, CTA del hero
├── about.md                   # Texto "Sobre nosotros"
├── servicios.md               # Lista de servicios (título + descripción por servicio)
├── testimonials.md            # Testimonios (texto + nombre por testimonio)
├── contact-layout.astro       # Layout del formulario (estructura, no contenido)
├── footer.md                  # Texto del footer, links
└── sections-config.md         # Resumen de secciones y su orden
```

### Ejemplo: `hero.md` (placeholder)

```markdown
---
section: hero
order: 1
type: static
---

# Hero — Contenido

## Título Principal
<!-- Máximo 60 caracteres recomendado -->

[Escribe aquí el título principal de tu hero]

---

## Subtítulo
<!-- 1-2 frases descriptivas, máximo 160 caracteres -->

[Escribe aquí el subtítulo o descripción breve]

---

## Texto del Botón Principal (CTA)
<!-- 2-4 palabras máximo -->

[Escribe aquí el texto del botón]

---

## Destino del Botón Principal
<!-- Elige una opción y completa -->

- [ ] Ancla interna → #___________ (ejemplo: #contacto, #pricing)
- [ ] URL externa → https://___________
- [ ] Teléfono → tel:___________
- [ ] Email → mailto:___________

---

## Botón Secundario (opcional)

- [ ] Quiero un segundo botón

### Texto del Botón Secundario

[Escribe aquí el texto]

### Destino del Botón Secundario

- [ ] Ancla interna → #___________
- [ ] URL externa → https://___________

---

## Imagen del Hero

<!-- Nombre del archivo en assets-input/ -->

Archivo: _________________

<!-- Si no tienes imagen, se usará un placeholder con gradiente -->
```

### Ejemplo: `about.md` (placeholder)

```markdown
---
section: about
order: 2
type: static
---

# Sobre Nosotros — Contenido

## Título de la Sección

[Sobre Nosotros / Quiénes Somos / Nuestra Historia]

---

## Texto Principal
<!-- 2-4 párrafos. Puedes usar Markdown para formato -->

[Escribe aquí el texto de la sección Sobre Nosotros.
Puedes usar **negrita**, *cursiva*, y saltos de línea.]

---

## Imagen (opcional)

<!-- Nombre del archivo en assets-input/ -->

Archivo: _________________

---

## Elementos Destacados (opcional)
<!-- Puntos clave que quieres resaltar con iconos o números -->

- [ ] Quiero elementos destacados

1. [Icono/Emoji] **[Título]** — Descripción breve
2. [Icono/Emoji] **[Título]** — Descripción breve
3. [Icono/Emoji] **[Título]** — Descripción breve
```

### Ejemplo: `servicios.md` (placeholder)

```markdown
---
section: services
order: 3
type: static
---

# Servicios — Contenido

## Título de la Sección

[Nuestros Servicios / Qué Ofrecemos / Lo Que Hacemos]

---

## Lista de Servicios

<!-- Cada servicio tiene: nombre, descripción breve, icono/emoji opcional -->

### Servicio 1
- **Nombre**: ___________________________
- **Descripción**: ___________________________
- **Icono/Emoji** (opcional): 🔧

### Servicio 2
- **Nombre**: ___________________________
- **Descripción**: ___________________________
- **Icono/Emoji** (opcional): 🎨

### Servicio 3
- **Nombre**: ___________________________
- **Descripción**: ___________________________
- **Icono/Emoji** (opcional): 📦

<!-- Añade más servicios copiando el bloque anterior -->
```

### Ejemplo: `contact-layout.astro` (placeholder de layout)

```astro
---
// contact-layout.astro
// Layout del formulario de contacto
// El usuario NO necesita editar este archivo
// Solo rellena contact.md con los datos
interface Props {
  titulo?: string;
  subtitulo?: string;
  email?: string;
  telefono?: string;
  direccion?: string;
  mapsUrl?: string;
}

const {
  titulo = "Contacto",
  subtitulo = "¿Tienes alguna pregunta? Escríbenos",
  email = "",
  telefono = "",
  direccion = "",
  mapsUrl = ""
} = Astro.props;
---

<section id="contact" class="contact-section">
  <h2>{titulo}</h2>
  <p>{subtitulo}</p>

  <div class="contact-grid">
    <div class="contact-info">
      {email && <p>📧 <a href={`mailto:${email}`}>{email}</a></p>}
      {telefono && <p>📞 <a href={`tel:${telefono}`}>{telefono}</a></p>}
      {direccion && <p>📍 {direccion}</p>}
      {mapsUrl && <a href={mapsUrl} target="_blank" rel="noopener">Ver en mapa →</a>}
    </div>

    <form class="contact-form" action="#" method="POST">
      <label for="nombre">Nombre</label>
      <input type="text" id="nombre" name="nombre" required />

      <label for="email">Email</label>
      <input type="email" id="email" name="email" required />

      <label for="mensaje">Mensaje</label>
      <textarea id="mensaje" name="mensaje" rows="5" required></textarea>

      <button type="submit">Enviar Mensaje</button>
    </form>
  </div>
</section>

<style>
  .contact-section { padding: 4rem 2rem; }
  .contact-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 3rem; }
  .contact-form { display: flex; flex-direction: column; gap: 1rem; }
  .contact-form input,
  .contact-form textarea {
    padding: 0.75rem;
    border: 1px solid #334155;
    border-radius: 8px;
    background: #1e1b4b;
    color: white;
  }
  .contact-form button {
    background: linear-gradient(135deg, #7c3aed, #3b82f6);
    color: white;
    padding: 0.75rem 2rem;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
  }
  @media (max-width: 768px) {
    .contact-grid { grid-template-columns: 1fr; }
  }
</style>
```

---

## 10. JSON Resultantes

### Estructura Final de `data-json/`

```
sites/mi-cafeteria/data-json/
├── perfil.json          ← De C-MD perfil.md
├── proyecto.json        ← De C-MD proyecto.md
└── estructura.json      ← De C-MD estructura.md + material-req.md
```

### Origen de Datos por JSON

| JSON | Fuente Principal | Fuente Secundaria |
|------|-----------------|-------------------|
| `perfil.json` | `C-MD/perfil.md` | — |
| `proyecto.json` | `C-MD/proyecto.md` | `data-usr/contenido/*.md` |
| `estructura.json` | `C-MD/estructura.md` | `C-MD/material-req.md` |

### FAQ JSON-LD (separado)

```
sites/mi-cafeteria/data-json/
└── faq-schema.jsonld    ← De C-MD/faq.md (convertido a FAQPage Schema)
```

---

## 11. API Keys y .env

### Archivo `.env` por Proyecto

**Ubicación**: `sites/mi-cafeteria/.env`

```env
# Proveedor de IA para generación del sitio
# Descomentar y configurar según el proveedor elegido en estructura.md

# Ollama (local, sin API key)
OLLAMA_API_BASE=http://localhost:11434
OLLAMA_MODEL=qwen2.5-coder:32b

# DeepSeek
DEEPSEEK_API_KEY=
DEEPSEEK_API_BASE=https://api.deepseek.com/v1

# OpenRouter
OPENROUTER_API_KEY=

# Anthropic
ANTHROPIC_API_KEY=

# Google
GOOGLE_GENERATIVE_AI_API_KEY=

# Mistral
MISTRAL_API_KEY=

# Cerebras
CEREBRAS_API_KEY=

# OpenAI-compatible
OPENAI_COMPATIBLE_API_KEY=
OPENAI_COMPATIBLE_API_BASE=
OPENAI_COMPATIBLE_MODEL=

# LM Studio
LM_STUDIO_API_BASE=http://localhost:1234/v1

# Proveedor por defecto para este proyecto
DEFAULT_PROVIDER=ollama
```

### `.env.example` por Proyecto

**Ubicación**: `sites/mi-cafeteria/.env.example`

```env
# Copia este archivo a .env y configura tu proveedor de IA

# Elige UNO de los siguientes proveedores:

# Opción 1: Ollama (local, gratuito)
OLLAMA_API_BASE=http://localhost:11434
OLLAMA_MODEL=qwen2.5-coder:32b

# Opción 2: DeepSeek (cloud)
DEEPSEEK_API_KEY=sk-xxx
DEEPSEEK_API_BASE=https://api.deepseek.com/v1

# Opción 3: OpenRouter (cloud, 400+ modelos)
OPENROUTER_API_KEY=sk-or-xxx

# Opción 4: Anthropic Claude (cloud)
ANTHROPIC_API_KEY=sk-ant-xxx

# Opción 5: Google Gemini (cloud)
GOOGLE_GENERATIVE_AI_API_KEY=AIza-xxx

# Opción 6: Mistral (cloud)
MISTRAL_API_KEY=xxx

# Opción 7: Cerebras (cloud, ultra-rápido)
CEREBRAS_API_KEY=csk-xxx

# Opción 8: OpenAI-compatible
OPENAI_COMPATIBLE_API_KEY=sk-xxx
OPENAI_COMPATIBLE_API_BASE=https://api.openai.com/v1
OPENAI_COMPATIBLE_MODEL=gpt-4o

# Opción 9: LM Studio (local)
LM_STUDIO_API_BASE=http://localhost:1234/v1

# Proveedor por defecto
DEFAULT_PROVIDER=ollama
```

### `.gitignore` del Proyecto

**Ubicación**: `sites/mi-cafeteria/.gitignore`

```gitignore
# API Keys — NUNCA commitear
.env

# Dependencias instaladas
dist/node_modules/
dist/.astro/

# Build output (re-generable)
dist/dist/

# Temporales
*.tmp
*.log
```

---

## 12. Flujo de Trabajo Completo

### Paso a Paso para el Usuario

```
┌──────────────────────────────────────────────────────────────────┐
│ PASO 1: Crear Proyecto                                           │
│                                                                  │
│ $ ./scripts/onepage/crear-proyecto.sh mi-cafeteria               │
│                                                                  │
│ → Crea toda la estructura de carpetas                            │
│ → Copia template base a templates/mi-cafeteria/                  │
│ → Genera C-MD vacíos en data-usr/C-MD/                           │
│ → Genera .env.example                                            │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PASO 2: Rellenar Cuestionarios C-MD                              │
│                                                                  │
│ El usuario edita los archivos en data-usr/C-MD/:                 │
│   1. perfil.md          → Info personal/empresa (reutilizable)   │
│   2. proyecto.md        → Info específica del proyecto           │
│   3. estructura.md      → Datos técnicos (IA, deploy, WCAG)      │
│   4. material-req.md    → Branding + checklist de assets         │
│   5. faq.md (opcional)  → FAQs en formato P/R                    │
│                                                                  │
│ El usuario coloca archivos en assets-input/:                     │
│   logo.png, hero-image.jpg, favicon.ico, etc.                    │
│                                                                  │
│ El usuario configura .env con su API key:                        │
│   cp .env.example .env                                           │
│   nano .env  → descomentar proveedor y añadir API key            │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PASO 3: Convertir C-MD a JSON                                    │
│                                                                  │
│ $ ./scripts/onepage/convertir-json.sh mi-cafeteria               │
│                                                                  │
│ → Ejecuta asistente IA que lee C-MD y genera JSON:               │
│   - data-json/perfil.json                                        │
│   - data-json/proyecto.json                                      │
│   - data-json/estructura.json                                    │
│   - data-json/faq-schema.jsonld (si hay FAQs)                    │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PASO 4: Generar Esqueleto de Contenido (Fase 2)                  │
│                                                                  │
│ $ ./scripts/onepage/generar-esqueleto-contenido.sh mi-cafeteria  │
│                                                                  │
│ → Crea archivos placeholder en data-usr/contenido/:              │
│   - hero.md                                                      │
│   - about.md                                                     │
│   - servicios.md                                                 │
│   - testimonials.md                                              │
│   - contact-layout.astro                                         │
│   - footer.md                                                    │
│   - sections-config.md                                           │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PASO 5: Rellenar Contenido                                       │
│                                                                  │
│ El usuario edita los archivos en data-usr/contenido/:            │
│   - hero.md         → Título, subtítulo, CTA                     │
│   - about.md        → Texto "Sobre nosotros"                     │
│   - servicios.md    → Lista de servicios                         │
│   - testimonials.md → Testimonios                                │
│   - footer.md       → Texto del footer                           │
│                                                                  │
│ contact-layout.astro NO necesita edición (solo los datos van     │
│ en los JSON que ya se generaron)                                 │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PASO 6: Generar Sitio con IA                                     │
│                                                                  │
│ $ ./scripts/onepage/generar-sitio.sh mi-cafeteria                │
│                                                                  │
│ → Lee JSON de data-json/                                         │
│ → Lee contenido de data-usr/contenido/                           │
│ → Lee assets de assets-input/                                    │
│ → Inyecta datos en prompts/onepage/01-07                         │
│ → Llama a IA (proveedor configurado en .env)                     │
│ → Genera componentes Astro+React en dist/                        │
│ → npm install + astro build                                      │
│ → Resultado en dist/dist/                                        │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PASO 7: Previsualizar                                            │
│                                                                  │
│ $ ./scripts/onepage/previsualizar.sh mi-cafeteria                │
│                                                                  │
│ → Sirve dist/dist/ en http://localhost:8080                      │
│ → Usuario revisa y valida                                        │
│ → Ctrl+C para detener                                            │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ PASO 8: Desplegar                                                │
│                                                                  │
│ El contenido de dist/dist/ está listo para:                      │
│   - GitHub Pages                                                 │
│   - Cloudflare Pages                                             │
│   - Netlify                                                      │
│   - Vercel                                                       │
│   - Railway                                                      │
│   - FTP a alojamiento compartido                                 │
└──────────────────────────────────────────────────────────────────┘
```

---

## 13. Scripts Necesarios

### `scripts/onepage/crear-proyecto.sh`

**Función**: Crear estructura de carpetas para un nuevo proyecto.

**Entrada**: Nombre del proyecto
**Salida**: `sites/[nombre]/` con toda la estructura, C-MD vacíos, .env.example, template copiado

**Acciones**:
1. Valida nombre
2. Crea directorios: `data-usr/C-MD/`, `data-usr/contenido/`, `data-json/`, `assets-input/`, `templates/`, `dist/`
3. Copia template base a `templates/[nombre]/`
4. Genera C-MD vacíos en `data-usr/C-MD/`
5. Genera `.env` y `.env.example`
6. Genera `.gitignore`
7. Genera `README.md` con instrucciones

---

### `scripts/onepage/generar-cuestionarios.sh`

**Función**: (Opcional) Regenerar C-MD desde plantillas si se actualizan.

**Entrada**: Nombre del proyecto
**Salida**: C-MD actualizados en `data-usr/C-MD/`

---

### `scripts/onepage/generar-esqueleto-contenido.sh`

**Función**: Fase 2 — Crear archivos placeholder de contenido basados en las secciones elegidas.

**Entrada**: `estructura.json` (secciones marcadas)
**Salida**: Archivos `.md` y `.astro` en `data-usr/contenido/`

**Lógica**:
- Lee `estructura.json` → secciones seleccionadas
- Para cada sección: genera el archivo correspondiente con placeholders
- Secciones estáticas → `.md` con frontmatter
- Secciones con layout → `.astro` con estructura y props

---

### `scripts/onepage/convertir-json.sh`

**Función**: Ejecutar asistente IA que lee C-MD y genera JSON.

**Entrada**: C-MD rellenados en `data-usr/C-MD/`
**Salida**: JSON en `data-json/`

**Mecanismo**:
- Usa un prompt de IA especializado que lee los C-MD y produce JSON estructurado
- El usuario ejecuta el script manualmente
- Requiere proveedor de IA configurado en `.env`

---

### `scripts/onepage/generar-sitio.sh`

**Función**: Generar el sitio Astro+React completo.

**Entrada**:
- `data-json/*.json` (datos estructurados)
- `data-usr/contenido/*.md` (contenido)
- `assets-input/` (archivos del usuario)
- `.env` (API keys)

**Salida**: `dist/` con proyecto Astro completo y `dist/dist/` con build final

**Lógica**:
1. Lee los 3 JSON + contenido + assets
2. Construye contexto completo para los prompts
3. Ejecuta prompts/onepage/01-07 secuencialmente
4. Parsea respuestas en archivos Astro+React
5. npm install + astro build
6. Resultado en dist/dist/

---

### `scripts/onepage/previsualizar.sh`

**Función**: Servidor HTTP local para revisión.

**Entrada**: Nombre del proyecto, puerto opcional
**Salida**: http://localhost:8080 (o puerto especificado)

---

## 14. Templates por Proyecto

### Template Base

**Ubicación**: `templates/base-astro-react/`

Contenido mínimo:
```
templates/base-astro-react/
├── astro.config.mjs
├── package.json
├── tsconfig.json
├── public/
│   └── favicon.svg
└── src/
    ├── layouts/
    │   └── Layout.astro    # Layout base con <head>, nav, footer genéricos
    ├── pages/
    │   └── index.astro     # Página vacía
    └── styles/
        └── global.css      # Reset mínimo + variables CSS
```

### Copia por Proyecto

Al crear un proyecto:
```bash
cp -r templates/base-astro-react/ sites/mi-cafeteria/templates/
```

El proyecto puede modificar su copia independientemente de otros proyectos.

### Futuros Templates

```
templates/
├── base-astro-react/       # Base mínima
├── business/               # Template negocio (hero, services, contact)
├── portfolio/              # Template portfolio (hero, gallery, about)
├── restaurant/             # Template restaurante (hero, menu, reservations)
└── landing/                # Template landing page (hero, features, CTA)
```

---

## 15. Notas Técnicas y Pendientes

### Pendientes de Definición

| # | Tema | Estado | Notas |
|---|------|--------|-------|
| P1 | Formato exacto del asistente IA para C-MD → JSON | ⬜ Por definir | ¿Prompt que lee archivos .md y genera .json? |
| P2 | Cómo maneja el script errores en los C-MD | ⬜ Por definir | Validación de campos requeridos antes de convertir |
| P3 | Sistema de versiones de C-MD | ⬜ Por definir | ¿Qué pasa si se actualiza un C-MD y el usuario ya lo rellenó? |
| P4 | Multi-idioma en C-MD | ⬜ Por definir | Traducir cuestionarios a otros idiomas |
| P5 | Integración con CI/CD para deploy automático | ⬜ Futuro | GitHub Actions para deploy al hacer push |
| P6 | Script de deploy por plataforma | ⬜ Futuro | deploy-prepare.sh con opciones por plataforma |

### Decisiones Técnicas ya Tomadas

| Decisión | Elección | Justificación |
|----------|----------|---------------|
| Formularios rellenables en Markdown | Opción A | El usuario trabaja en un editor de texto, sin necesidad de terminal |
| Asistente IA para C-MD → JSON | Manual por usuario | Más flexible que regex bash; puede manejar respuestas complejas |
| Contenido mixto .md + .astro | Opción C | .md para contenido editable, .astro para estructura fija |
| Template copia por proyecto | Opción A | Cada proyecto evoluciona independientemente |
| Todos los 9 proveedores | Completo | Aprovecha toda la infraestructura existente de LocalSite-ai |
| material-req.md con preguntas | Opción B | El usuario tiene todo en un solo archivo para branding + assets |
| FAQs en Markdown P/R | Opción B | Más fácil de rellenar que JSON-LD directo |
| Scripts bash directos | Terminal | Más simple, transparente, sin dependencias extra |

### Riesgos Identificados

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Usuario no rellena C-MD correctamente | Alto | Ejemplos claros en cada campo; validación al convertir a JSON |
| IA genera código con errores | Alto | Build log visible; archivos fuente editables manualmente |
| Tokens insuficientes en IA | Medio | Prompts divididos en 7 pasos; contexto limitado al proyecto actual |
| Assets no encontrados | Bajo | Fallback a placeholders si el archivo no existe |
| API key incorrecta | Alto | Validación al inicio de generar-sitio.sh |

---

> **FIN DEL DOCUMENTO DE TRABAJO**
>
> Este documento es un borrador de trabajo pendiente de revisión.
> No implementar nada hasta aprobación final.
