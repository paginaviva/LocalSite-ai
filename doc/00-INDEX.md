# Índice de Documentación — LocalSite-ai One-Page Builder

**Fecha de creación**: 15 de abril de 2026  
**Última actualización**: 15 de abril de 2026

---

## 📋 Vista General

Este índice enumera todos los documentos en la carpeta `doc/` organizados en orden de lectura recomendado, indicando la finalidad de cada archivo y sus dependencias.

---

## 📚 Documentos Principales

### 1️⃣ **01-LocalSite-ai-Analisis-Completo.md**
- **Propósito**: Presentación y fundamentos de LocalSite-ai
- **Contenido**: ¿Qué es LocalSite-ai?, para qué sirve, características, arquitectura técnica, stack tecnológico, casos de uso
- **Dependencias**: Ninguna (punto de partida)
- **Tipo**: Análisis introductorio
- **Leer si**: Quieres entender qué es LocalSite-ai y cómo funciona

---

### 2️⃣ **02-Preguntas-Aclaracion-OnePage-Builders.md**
- **Propósito**: Resolver preguntas de diseño arquitectónico del proyecto One-Page Builder
- **Contenido**: Decisiones sobre formato de cuestionarios, conversión de datos, archivos de contenido, scripts
- **Dependencias**: Requiere comprensión de LocalSite-ai (Doc 1)
- **Tipo**: Q&A de decisiones técnicas
- **Leer si**: Necesitas entender las decisiones clave del proyecto

---

### 3️⃣ **03-Plan-Maestro-OnePage-Site-Builder.md**
- **Propósito**: Plan maestro completo del sistema automatizado de generación de sitios one-page
- **Contenido**: Objetivo, arquitectura, estructura de carpetas, scripts (crear-proyecto, recopilar-info, generar-sitio, previsualizar), flujo jerárquico de prompts, decisiones técnicas, roadmap de implementación
- **Dependencias**: Requiere decisiones de Doc 2
- **Tipo**: Documento de especificación técnica
- **Leer si**: Vas a implementar o entender el flujo completo del sistema

---

### 4️⃣ **04-LocalSite-ai-con-Astro-React-Analisis.md**
- **Propósito**: Análisis de viabilidad de integración entre LocalSite-ai, Astro y React
- **Contenido**: ¿Qué es Astro?, complementariedad con React, flujo de trabajo integrado, comparativa, limitaciones, roadmap
- **Dependencias**: Requiere entender LocalSite-ai (Doc 1) y el Plan Maestro (Doc 3)
- **Tipo**: Análisis de arquitectura y viabilidad
- **Leer si**: Planeas integrar LocalSite-ai con frameworks modernos

---

### 5️⃣ **05-BOCADILLO-Cuestionarios-OnePage.md**
- **Propósito**: Documento de trabajo con definición de cuestionarios y estructura de datos para el One-Page Builder
- **Contenido**: Decisiones confirmadas, estructura de carpetas, bloques de cuestionarios (C-MD), archivos de contenido, JSON resultantes, flujo de trabajo, scripts necesarios, templates por proyecto
- **Dependencias**: Requiere las 4 documentaciones anteriores
- **Tipo**: Documento de trabajo / especificación ejecutable
- **Leer si**: Necesitas los detalles de implementación de cuestionarios y estructura de datos

---

## 📦 Documentos en Archivo (Legado)

### **doc-archivados-legado/06-LocalSite-ai-con-Astro-Vue-Analisis.md**
- **Propósito**: Análisis de integración con Astro + Vue (versión anterior)
- **Contenido**: Comparativa de arquitecturas Astro + Vue vs Astro + React
- **Dependencias**: Requiere Doc 1 y Doc 3
- **Tipo**: Análisis comparativo (ARCHIVADO)
- **Nota**: Este documento fue reemplazado por el análisis Astro + React. Se mantiene por referencia histórica.
- **Leer si**: Necesitas comparar opciones framework o entender decisiones históricas

---

## 🔄 Flujo de Lectura Recomendado

**Para principiantes (entender el proyecto):**  
1 → 2 → 3 → 4 → 5

**Para implementadores (coding):**  
1 → 3 → 5 (opcional: 4)

**Para arquitectos (diseño):**  
2 → 3 → 4 → 5

**Para review histórico:**  
1 → 2 → 3 → 4 → 5 → (Archivados)

---

## 📊 Estadísticas

| Métrica | Valor |
|---------|-------|
| Total documentos activos | 5 |
| Total documentos archivados | 1 |
| Carpeta raíz | `doc/` |
| Carpeta archivada | `doc/doc-archivados-legado/` |

---

## 🎯 Propósito General de la Carpeta

La carpeta `doc/` contiene la documentación completa del proyecto **LocalSite-ai One-Page Builder**, un sistema automatizado que permite a usuarios sin experiencia técnica generar sitios web profesionales one-page mediante:

1. Responder cuestionarios interactivos
2. Proporcionar assets (logotipos, imágenes)
3. Confirmación de generación mediante IA
4. Previsualización y despliegue

Los documentos evolucionan desde conceptos fundamentales hasta especificaciones técnicas ejecutables, formando un continuum de aprendizaje y referencia.

---

**Última revisión**: 15 de abril de 2026 | Creado por: GitHub Copilot
