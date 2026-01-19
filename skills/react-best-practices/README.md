# Buenas Prácticas de React

Un repositorio estructurado para crear y mantener Buenas Prácticas de React optimizadas para agentes e LLMs.

## Estructura

- `rules/` - Archivos de reglas individuales (uno por regla)
  - `_sections.md` - Metadata de secciones (títulos, impactos, descripciones)
  - `_template.md` - Template para crear nuevas reglas
  - `area-description.md` - Archivos de reglas individuales
- `src/` - Scripts de build y utilidades
- `metadata.json` - Metadata del documento (versión, organización, resumen)
- __`AGENTS.md`__ - Salida compilada (generado)
- __`test-cases.json`__ - Casos de test para evaluación de LLM (generado)

## Comenzar

1. Instalar dependencias:
   ```bash
   pnpm install
   ```

2. Compilar AGENTS.md desde reglas:
   ```bash
   pnpm build
   ```

3. Validar archivos de reglas:
   ```bash
   pnpm validate
   ```

4. Extraer casos de test:
   ```bash
   pnpm extract-tests
   ```

## Crear una Nueva Regla

1. Copiá `rules/_template.md` a `rules/area-description.md`
2. Elegí el prefijo de área apropiado:
   - `async-` para Eliminación de Waterfalls (Sección 1)
   - `bundle-` para Optimización de Bundle (Sección 2)
   - `server-` para Rendimiento Server-Side (Sección 3)
   - `client-` para Data Fetching en Cliente (Sección 4)
   - `rerender-` para Optimización de Re-renders (Sección 5)
   - `rendering-` para Rendimiento de Renderizado (Sección 6)
   - `js-` para Rendimiento JavaScript (Sección 7)
   - `advanced-` para Patrones Avanzados (Sección 8)
3. Completá el frontmatter y contenido
4. Asegurate de tener ejemplos claros con explicaciones
5. Ejecutá `pnpm build` para regenerar AGENTS.md y test-cases.json

## Estructura de Archivo de Regla

Cada archivo de regla debería seguir esta estructura:

```markdown
---
title: Título de la Regla Acá
impact: MEDIUM
impactDescription: Descripción opcional
tags: tag1, tag2, tag3
---

## Título de la Regla Acá

Explicación breve de la regla y por qué importa.

**Incorrecto (descripción de qué está mal):**

```typescript
// Ejemplo de código malo
```

**Correcto (descripción de qué está bien):**

```typescript
// Ejemplo de código bueno
```

Texto explicativo opcional después de los ejemplos.

Referencia: [Link](https://example.com)

## Convención de Nombres de Archivos

- Archivos que empiezan con `_` son especiales (excluidos del build)
- Archivos de reglas: `area-description.md` (ej., `async-parallel.md`)
- La sección se infiere automáticamente del prefijo del nombre de archivo
- Las reglas se ordenan alfabéticamente por título dentro de cada sección
- Los IDs (ej., 1.1, 1.2) se auto-generan durante el build

## Niveles de Impacto

- `CRITICAL` - Máxima prioridad, ganancias de rendimiento mayores
- `HIGH` - Mejoras de rendimiento significativas
- `MEDIUM-HIGH` - Ganancias moderadas-altas
- `MEDIUM` - Mejoras de rendimiento moderadas
- `LOW-MEDIUM` - Ganancias bajas-medias
- `LOW` - Mejoras incrementales

## Scripts

- `pnpm build` - Compila reglas en AGENTS.md
- `pnpm validate` - Valida todos los archivos de reglas
- `pnpm extract-tests` - Extrae casos de test para evaluación de LLM
- `pnpm dev` - Build y validación

## Contribuir

Al agregar o modificar reglas:

1. Usá el prefijo de nombre de archivo correcto para tu sección
2. Seguí la estructura de `_template.md`
3. Incluí ejemplos claros de malo/bueno con explicaciones
4. Agregá tags apropiados
5. Ejecutá `pnpm build` para regenerar AGENTS.md y test-cases.json
6. Las reglas se ordenan automáticamente por título - ¡no necesitás manejar números!

## Reconocimientos

Creado originalmente por [@shuding](https://x.com/shuding) en [Vercel](https://vercel.com).
