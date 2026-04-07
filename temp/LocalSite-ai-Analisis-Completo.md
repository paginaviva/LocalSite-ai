# Análisis Completo de LocalSite-ai

## Tabla de Contenidos
1. [¿Qué es LocalSite-ai?](#qué-es-localsite-ai)
2. [¿Para Qué Sirve?](#para-qué-sirve)
3. [Características Principales](#características-principales)
4. [Arquitectura Técnica](#arquitectura-técnica)
5. [Proveedores de IA Soportados](#proveedores-de-ia-soportados)
6. [Flujo de Funcionamiento](#flujo-de-funcionamiento)
7. [Stack Tecnológico](#stack-tecnológico)
8. [Estructura del Proyecto](#estructura-del-proyecto)
9. [Casos de Uso](#casos-de-uso)
10. [Ventajas y Limitaciones](#ventajas-y-limitaciones)
11. [Instalación y Configuración](#instalación-y-configuración)
12. [Estado Actual y Roadmap](#estado-actual-y-roadmap)

---

## ¿Qué es LocalSite-ai?

**LocalSite-ai** es una aplicación web moderna construida con **Next.js 15** y **React 19** que utiliza **inteligencia artificial** para generar código HTML, CSS y JavaScript completo y autocontenido a partir de descripciones en lenguaje natural.

En esencia, es una herramienta que permite a los usuarios describir con palabras lo que quieren construir (por ejemplo: *"Crea una página de aterrizaje para una cafetería con un formulario de contacto y una galería de imágenes"*) y la IA genera automáticamente un archivo HTML funcional con todo el código necesario integrado.

> **Versión actual:** 0.5.2  
> **Licencia:** MIT  
> **Desarrollo:** Asistido por IA (Claude Opus 4.6 de Anthropic)

---

## ¿Para Qué Sirve?

### Propósito Principal

LocalSite-ai sirve para **generar páginas web completas a partir de prompts de texto**. El usuario describe lo que quiere y la IA produce un archivo HTML autocontenido que incluye:

- **Estructura HTML** semántica y válida
- **Estilos CSS** integrados (dentro de etiquetas `<style>`)
- **JavaScript** funcional (dentro de etiquetas `<script>`)
- **Diseño responsive** adaptable a móviles, tablets y escritorio

### Público Objetivo

| Usuario | Beneficio |
|---------|-----------|
| **No programadores** | Crear páginas web sin saber codificar |
| **Desarrolladores web** | Prototipado rápido de interfaces |
| **Diseñadores UX/UI** | Validar ideas de diseño rápidamente |
| **Estudiantes** | Aprender estructura web viendo ejemplos generados |
| **Profesionales de marketing** | Crear landing pages sin depender de desarrollo |

---

## Características Principales

### 1. Generación de Código con IA
- Describe en lenguaje natural lo que deseas construir
- La IA interpreta el prompt y genera código funcional
- Soporte para **system prompts personalizados** (default, thinking, custom)

### 2. Vista Previa en Tiempo Real
- **Live preview** del código generado directamente en el navegador
- Múltiples vistas de viewport: **Escritorio, Tablet y Móvil**
- Modo oscuro automático para la vista previa

### 3. Editor de Código Integrado
- Editor basado en **Monaco Editor** (el mismo motor de VS Code)
- Edición directa del código generado
- Opción de guardar cambios o revertir al original

### 4. Múltiples Proveedores de IA
- **Modelos locales**: Ollama, LM Studio (sin costo, privacidad total)
- **Modelos en la nube**: DeepSeek, Anthropic Claude, Google Gemini, Mistral, Cerebras, OpenRouter
- **APIs compatibles con OpenAI**: Cualquier proveedor que siga el estándar OpenAI

### 5. Soporte para Modelos de Razonamiento
- Extrae automáticamente el proceso de pensamiento de modelos como `deepseek-reasoner`
- Visualización del "pensamiento" de la IA mientras genera código
- System prompt especializado para modelos con capacidad de razonamiento (`<think>` tags)

### 6. Interfaz Moderna y Responsiva
- Tema oscuro por defecto
- Paneles redimensionables (código a la izquierda, preview a la derecha)
- Diseño adaptable a móviles con navegación por pestañas
- Construida con **Shadcn UI** y **Tailwind CSS**

### 7. Exportación de Código
- Copiar el código al portapapeles
- Descargar como archivo `.html` autocontenido

---

## Arquitectura Técnica

### Flujo de Datos

```
┌─────────────────────────────────────────────────────────────┐
│                     INTERFAZ DE USUARIO                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │ Welcome View │→ │ Generation   │→ │ Code Panel +     │   │
│  │ (Input +     │  │ View         │  │ Preview Panel    │   │
│  │  Config)     │  │ (Streaming)  │  │ (Resizable)      │   │
│  └──────────────┘  └──────────────┘  └──────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    CUSTOM HOOK                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  useCodeGeneration()                                 │   │
│  │  - Gestiona el estado de generación                  │   │
│  │  - Lee el stream NDJSON del backend                  │   │
│  │  - Actualiza código y razonamiento en tiempo real    │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     API BACKEND                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  /api/generate-code/route.ts                         │   │
│  │  - Recibe prompt, modelo y proveedor                 │   │
│  │  - Usa Vercel AI SDK (streamText)                    │   │
│  │  - Stream NDJSON con tipos 'text' y 'reasoning'      │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  PROVEEDORES DE IA                           │
│  ┌────────────┐ ┌──────────┐ ┌─────────┐ ┌──────────────┐  │
│  │ DeepSeek   │ │ Anthropic│ │ Google  │ │ Ollama       │  │
│  │ OpenRouter │ │ Mistral  │ │ Cerebras│ │ LM Studio    │  │
│  │ OpenAI-compatible       │ │ etc.    │ │              │  │
│  └────────────┘ └──────────┘ └─────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Componentes Clave

| Componente | Ubicación | Función |
|-----------|-----------|---------|
| `page.tsx` | `app/page.tsx` | Punto de entrada principal, gestiona vistas |
| `WelcomeView` | `components/welcome-view.tsx` | Pantalla inicial con input y configuración |
| `GenerationView` | `components/generation-view.tsx` | Vista principal tras generar código |
| `CodePanel` | `components/generation-panels.tsx` | Editor de código (Monaco) |
| `PreviewPanel` | `components/generation-panels.tsx` | Iframe con vista previa en vivo |
| `useCodeGeneration` | `hooks/use-code-generation.ts` | Hook personalizado para generación |
| `generate-code/route.ts` | `app/api/generate-code/` | API backend con streaming |
| `provider.ts` | `lib/providers/provider.ts` | Fábrica de proveedores de IA |
| `config.ts` | `lib/providers/config.ts` | Configuración de proveedores |
| `prompts.ts` | `lib/providers/prompts.ts` | System prompts para la IA |

---

## Proveedores de IA Soportados

### Modelos Locales (Sin Costo, Privacidad Total)

| Proveedor | Descripción | URL por Defecto | Requiere API Key |
|-----------|-------------|-----------------|------------------|
| **Ollama** | Ejecuta modelos localmente (Llama, Qwen, DeepSeek-R1) | `http://localhost:11434` | No |
| **LM Studio** | Interfaz gráfica para modelos locales | `http://localhost:1234/v1` | No |

### Modelos en la Nube

| Proveedor | Modelos Disponibles | Velocidad | Costo |
|-----------|---------------------|-----------|-------|
| **DeepSeek** | deepseek-chat, deepseek-reasoner | Rápido | Económico |
| **Cerebras** | Varios modelos | **Ultra-rápido** (~3 seg) | Variable |
| **Anthropic Claude** | Claude 3/4 series | Rápido | Premium |
| **Google Gemini** | Gemini Pro/Flash | Rápido | Variable |
| **Mistral** | Mistral Large/Small | Rápido | Económico |
| **OpenRouter** | 400+ modelos de múltiples proveedores | Variable | Variable |

### APIs Personalizadas

| Tipo | Descripción |
|------|-------------|
| **OpenAI-Compatible** | Cualquier API compatible con OpenAI (Groq, Together AI, Anyscale, etc.) |

---

## Flujo de Funcionamiento

### Paso a Paso

1. **Inicio (WelcomeView)**
   - El usuario ingresa un prompt describiendo la página web deseada
   - Selecciona un proveedor de IA (Ollama, DeepSeek, etc.)
   - Selecciona un modelo específico
   - Opcionalmente configura el system prompt y tokens máximos

2. **Generación (GenerationView)**
   - Al hacer clic en "GENERATE", se envía la petición al backend
   - El backend (`/api/generate-code`) utiliza el **Vercel AI SDK** con `streamText()`
   - La IA genera código en streaming (NDJSON format)
   - El frontend recibe chunks de texto y razonamiento por separado
   - El código se actualiza en tiempo real en el editor Monaco
   - La vista previa se actualiza con debounce (200ms)

3. **Visualización y Edición**
   - El usuario ve el código en el panel izquierdo (Monaco Editor)
   - Ve la vista previa en el panel derecho (iframe)
   - Puede alternar entre vistas Desktop/Tablet/Móvil
   - Puede activar el modo edición para modificar el código
   - Puede copiar o descargar el archivo HTML generado

4. **Regeneración**
   - Si el resultado no es el esperado, el usuario puede enviar un nuevo prompt
   - El sistema regenera el código manteniendo el contexto

### Streaming NDJSON

El protocolo de comunicación entre frontend y backend usa **NDJSON** (Newline-Delimited JSON):

```json
{"type": "reasoning", "content": "Voy a crear una estructura HTML con header, main y footer..."}
{"type": "text", "content": "<!DOCTYPE html>\n<html>\n<head>"}
{"type": "reasoning", "content": "Ahora agregaré los estilos CSS para el layout responsive..."}
{"type": "text", "content": "<style>body { margin: 0; }</style>"}
```

---

## Stack Tecnológico

### Framework y Librerías Principales

| Tecnología | Versión | Propósito |
|-----------|---------|-----------|
| **Next.js** | 15.2.6 | Framework Reactfull-stack con App Router |
| **React** | 19 | Librería de UI declarativa |
| **TypeScript** | 5 | Tipado estático |
| **Tailwind CSS** | 3.4.17 | Framework CSS utility-first |
| **Shadcn UI** | - | Componentes UI accesibles (Radix UI) |
| **Monaco Editor** | 4.7.0 | Editor de código (motor de VS Code) |

### Vercel AI SDK y Proveedores

| Paquete | Función |
|---------|---------|
| `ai` (v5) | SDK principal para IA con streaming unificado |
| `@ai-sdk/deepseek` | Integración con DeepSeek |
| `@ai-sdk/anthropic` | Integración con Anthropic Claude |
| `@ai-sdk/google` | Integración con Google Gemini |
| `@ai-sdk/mistral` | Integración con Mistral AI |
| `@ai-sdk/cerebras` | Integración con Cerebras |
| `@ai-sdk/openai-compatible` | APIs compatibles con OpenAI |
| `@openrouter/ai-sdk-provider` | Integración con OpenRouter |
| `ai-sdk-ollama` | Integración con Ollama (modelos locales) |

### Otras Dependencias

| Paquete | Función |
|---------|---------|
| `lodash.debounce` | Debounce para optimización de preview |
| `react-resizable-panels` | Paneles redimensionables |
| `sonner` | Notificaciones toast |
| `lucide-react` | Iconos |
| `zod` | Validación de esquemas |
| `cmdk` | Combobox/Command palette |

---

## Estructura del Proyecto

```
LocalSite-ai/
├── app/                          # Next.js App Router
│   ├── api/
│   │   ├── generate-code/       # Endpoint principal de generación
│   │   │   └── route.ts         # Handler POST con streaming
│   │   ├── get-default-provider/# Endpoint para obtener proveedor por defecto
│   │   └── get-models/          # Endpoint para listar modelos disponibles
│   ├── layout.tsx               # Layout raíz de la aplicación
│   ├── page.tsx                 # Página principal (Welcome/Generation)
│   └── globals.css              # Estilos globales
│
├── components/                   # Componentes React
│   ├── ui/                      # Componentes base de Shadcn UI (40 componentes)
│   │   ├── button.tsx
│   │   ├── dialog.tsx
│   │   ├── select.tsx
│   │   └── ...
│   ├── code-editor.tsx          # Wrapper de Monaco Editor
│   ├── generation-view.tsx      # Vista principal post-generación
│   ├── generation-panels.tsx    # Paneles de código y preview
│   ├── loading-screen.tsx       # Pantalla de carga inicial
│   ├── provider-selector.tsx    # Selector de proveedor de IA
│   ├── thinking-indicator.tsx   # Indicador de razonamiento de IA
│   ├── welcome-view.tsx         # Pantalla de bienvenida
│   └── work-steps.tsx           # Indicador de pasos de trabajo
│
├── hooks/                        # Custom React Hooks
│   └── use-code-generation.ts   # Hook principal de generación de código
│
├── lib/                          # Utilidades y configuración
│   ├── providers/
│   │   ├── config.ts            # Configuración de proveedores
│   │   ├── provider.ts          # Fábrica de clientes de IA
│   │   └── prompts.ts           # System prompts
│   └── utils.ts                 # Utilidades generales
│
├── public/                       # Archivos estáticos
│
├── docker-compose.yml            # Configuración Docker
├── Dockerfile                    # Imagen Docker
├── next.config.mjs               # Configuración Next.js
├── tailwind.config.ts            # Configuración Tailwind CSS
├── tsconfig.json                 # Configuración TypeScript
├── package.json                  # Dependencias del proyecto
└── .env.example                  # Variables de entorno de ejemplo
```

---

## Casos de Uso

### 1. Prototipado Rápido
**Escenario:** Un diseñador quiere validar rápidamente una idea de layout.  
**Prompt ejemplo:** *"Create a portfolio page with a hero section, project grid, and contact form"*

### 2. Landing Pages Simples
**Escenario:** Un emprendedor necesita una página de aterrizaje para su producto.  
**Prompt ejemplo:** *"Build a landing page for a SaaS product with pricing table and testimonials"*

### 3. Componentes UI
**Escenario:** Un desarrollador necesita un componente específico.  
**Prompt ejemplo:** *"Create a responsive navigation bar with dropdown menus and mobile hamburger icon"*

### 4. Aprendizaje
**Escenario:** Un estudiante quiere entender cómo se estructura una página web.  
**Prompt ejemplo:** *"Make a simple blog page with semantic HTML5 tags and CSS grid layout"*

### 5. Herramientas Internas
**Escenario:** Un equipo necesita una herramienta interna simple.  
**Prompt ejemplo:** *"Create a todo list app with local storage persistence and drag-and-drop"*

---

## Ventajas y Limitaciones

### ✅ Ventajas

| Ventaja | Descripción |
|---------|-------------|
| **Sin necesidad de programar** | Cualquier persona puede crear páginas web con descripciones en lenguaje natural |
| **Múltiples proveedores** | Flexibilidad para elegir entre modelos locales (gratis) o en la nube (pagos) |
| **Código autocontenido** | El resultado es un único archivo HTML que funciona sin dependencias externas |
| **Edición en vivo** | Posibilidad de modificar el código generado directamente en el navegador |
| **Streaming en tiempo real** | Se ve el código generarse progresivamente, no hay espera ciega |
| **Soporte de razonamiento** | Visualiza el "pensamiento" de la IA para entender sus decisiones |
| **Open Source** | Código disponible bajo licencia MIT, modificable y distribuible |
| **Rendimiento optimizado** | Lazy loading, debounce, caching, batch updates |

### ⚠️ Limitaciones

| Limitación | Descripción |
|------------|-------------|
| **Archivo único** | Solo genera un archivo HTML; no proyectos multi-archivo |
| **Sin frameworks JS** | No genera proyectos React/Vue/Angular con build systems (planeado futuro) |
| **Dependencia de IA** | La calidad del código depende del modelo de IA seleccionado |
| **No persistente** | No guarda proyectos ni historial de generaciones |
| **Modelos locales limitados** | Ollama/LM Studio solo funcionan en localhost (no en hosting cloud sin tunneling) |
| **Sin versión móvil nativa** | No es una app móvil; es responsive web pero no app nativa |
| **IA-asistido** | El código puede requerir ajustes manuales para producción |

---

## Instalación y Configuración

### Requisitos Previos

- **Node.js** (versión 18.17 o superior)
- **npm** o **yarn**
- **Ollama** o **LM Studio** (para modelos locales) **O** una API key de un proveedor cloud

### Pasos de Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/weise25/LocalSite-ai.git
cd LocalSite-ai

# 2. Instalar dependencias
npm install

# 3. Configurar variables de entorno
cp .env.example .env.local
# Editar .env.local con las credenciales del proveedor elegido

# 4. Iniciar servidor de desarrollo
npm run dev

# 5. Abrir en navegador
# http://localhost:3000
```

### Ejemplo de `.env.local`

#### Con DeepSeek (Cloud)
```env
DEEPSEEK_API_KEY=sk-tu_clave_aqui
DEEPSEEK_API_BASE=https://api.deepseek.com/v1
DEFAULT_PROVIDER=deepseek
```

#### Con Ollama (Local)
```env
OLLAMA_API_BASE=http://localhost:11434
DEFAULT_PROVIDER=ollama
```

#### Con API OpenAI-Compatible (ej. Groq)
```env
OPENAI_COMPATIBLE_API_KEY=gsk_tu_clave
OPENAI_COMPATIBLE_API_BASE=https://api.groq.com/openai/v1
DEFAULT_PROVIDER=openai_compatible
```

### Docker

```bash
# Construir y ejecutar con Docker
docker build -t localsite-ai .
docker run -p 3000:3000 localsite-ai

# O con Docker Compose
docker-compose up -d
```

---

## Estado Actual y Roadmap

### ✅ Implementado

- [x] Integración con Ollama para ejecución local de modelos
- [x] Soporte para LM Studio
- [x] DeepSeek como proveedor predefinido
- [x] APIs compatibles con OpenAI
- [x] Soporte para modelos de razonamiento (Qwen3, DeepCoder, etc.)
- [x] Múltiples proveedores cloud (Anthropic, Google, Mistral, Cerebras, OpenRouter)
- [x] Streaming NDJSON con texto y razonamiento separados
- [x] Editor Monaco con lazy loading
- [x] Paneles redimensionables
- [x] Optimizaciones de rendimiento (debounce, caching, batch updates)
- [x] Responsive design (desktop + mobile)

### 🔜 Planificado

- [ ] Soporte para frameworks (React, Vue, Angular)
- [ ] Generación multi-archivo
- [ ] Guardar y cargar proyectos
- [ ] Edición agéntica con diff
- [ ] Toggle tema claro/oscuro
- [ ] Configuración del editor (temas, atajos)
- [ ] Drag-and-drop de componentes UI
- [ ] Historial de generaciones
- [ ] Transcripción y entrada de voz
- [ ] App de escritorio multiplataforma (Electron)

---

## Resumen Ejecutivo

**LocalSite-ai** es una herramienta de **generación de código web asistida por IA** que permite crear páginas HTML completas a partir de descripciones en lenguaje natural. Su mayor fortaleza es la **flexibilidad de proveedores de IA**: desde modelos locales gratuitos (Ollama, LM Studio) hasta servicios cloud premium (DeepSeek, Anthropic, Google). La aplicación ofrece una **experiencia de usuario completa**: desde la pantalla de bienvenida con configuración hasta la vista previa en vivo, edición de código y exportación del resultado.

Es ideal para **prototipado rápido**, **aprendizaje de desarrollo web**, y **creación de páginas simples** sin necesidad de conocimientos de programación. Si bien aún no soporta proyectos multi-archivo o frameworks JavaScript completos, su arquitectura modular y roadmap activo sugieren un desarrollo continuo hacia capacidades más avanzadas.

---

*Documento generado el 7 de abril de 2026 basado en el análisis del código fuente de LocalSite-ai versión 0.5.2.*
