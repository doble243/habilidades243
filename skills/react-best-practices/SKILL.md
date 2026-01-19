---
name: vercel-react-best-practices
description: Guías de optimización de rendimiento para React y Next.js del equipo de Vercel. Usá esta skill cuando escribas, revises o refactorices código React/Next.js para asegurar patrones de rendimiento óptimos. Se activa con tareas que involucran componentes React, páginas Next.js, data fetching, optimización de bundle o mejoras de performance. Frases disparadoras: "revisá este componente", "optimizá el rendimiento", "mejorá la performance", "analizá el bundle", "refactorizá este código".
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
---

# Buenas Prácticas de React - Vercel

Guía completa de optimización de rendimiento para aplicaciones React y Next.js, mantenida por Vercel. Contiene 45 reglas en 8 categorías, priorizadas por impacto para guiar refactorización automatizada y generación de código.

## Frases que Activan esta Skill

Esta skill se activa con frases como:
- "Revisá este componente React"
- "Optimizá el rendimiento de esta página"
- "Mejorá la performance de mi app"
- "Analizá el bundle size"
- "Refactorizá este código Next.js"
- "Revisá por problemas de rendimiento"
- "Hacé que cargue más rápido"
- "Optimizá los tiempos de carga"
- "Revisá el data fetching"
- "Mejorá el SSR/SSG"

## Cuándo Aplicar

Referenciá estas guías cuando:
- Escribas nuevos componentes React o páginas Next.js
- Implementes data fetching (cliente o servidor)
- Revises código por problemas de rendimiento
- Refactorices código React/Next.js existente
- Optimices tamaño del bundle o tiempos de carga

## Categorías de Reglas por Prioridad

| Prioridad | Categoría | Impacto | Prefijo |
|-----------|-----------|---------|---------|
| 1 | Eliminación de Waterfalls | CRÍTICO | `async-` |
| 2 | Optimización de Bundle | CRÍTICO | `bundle-` |
| 3 | Rendimiento Server-Side | ALTO | `server-` |
| 4 | Data Fetching en Cliente | MEDIO-ALTO | `client-` |
| 5 | Optimización de Re-renders | MEDIO | `rerender-` |
| 6 | Rendimiento de Renderizado | MEDIO | `rendering-` |
| 7 | Rendimiento JavaScript | BAJO-MEDIO | `js-` |
| 8 | Patrones Avanzados | BAJO | `advanced-` |

## Referencia Rápida

### 1. Eliminación de Waterfalls (CRÍTICO)

- `async-defer-await` - Mové await a las ramas donde realmente se usa
- `async-parallel` - Usá Promise.all() para operaciones independientes
- `async-dependencies` - Usá better-all para dependencias parciales
- `async-api-routes` - Iniciá promesas temprano, await tarde en API routes
- `async-suspense-boundaries` - Usá Suspense para streamear contenido

### 2. Optimización de Bundle (CRÍTICO)

- `bundle-barrel-imports` - Importá directo, evitá barrel files
- `bundle-dynamic-imports` - Usá next/dynamic para componentes pesados
- `bundle-defer-third-party` - Cargá analytics/logging después de hydration
- `bundle-conditional` - Cargá módulos solo cuando se activa la feature
- `bundle-preload` - Precargá en hover/focus para velocidad percibida

### 3. Rendimiento Server-Side (ALTO)

- `server-cache-react` - Usá React.cache() para deduplicación por request
- `server-cache-lru` - Usá caché LRU para cacheo entre requests
- `server-serialization` - Minimizá datos pasados a client components
- `server-parallel-fetching` - Reestructurá componentes para paralelizar fetches
- `server-after-nonblocking` - Usá after() para operaciones no bloqueantes

### 4. Data Fetching en Cliente (MEDIO-ALTO)

- `client-swr-dedup` - Usá SWR para deduplicación automática de requests
- `client-event-listeners` - Deduplicá event listeners globales

### 5. Optimización de Re-renders (MEDIO)

- `rerender-defer-reads` - No te suscribas a estado solo usado en callbacks
- `rerender-memo` - Extraé trabajo costoso a componentes memoizados
- `rerender-dependencies` - Usá dependencias primitivas en effects
- `rerender-derived-state` - Suscribite a booleanos derivados, no valores crudos
- `rerender-functional-setstate` - Usá setState funcional para callbacks estables
- `rerender-lazy-state-init` - Pasá función a useState para valores costosos
- `rerender-transitions` - Usá startTransition para updates no urgentes

### 6. Rendimiento de Renderizado (MEDIO)

- `rendering-animate-svg-wrapper` - Animá el div wrapper, no el SVG
- `rendering-content-visibility` - Usá content-visibility para listas largas
- `rendering-hoist-jsx` - Extraé JSX estático fuera de componentes
- `rendering-svg-precision` - Reducí precisión de coordenadas SVG
- `rendering-hydration-no-flicker` - Usá inline script para datos client-only
- `rendering-activity` - Usá componente Activity para show/hide
- `rendering-conditional-render` - Usá ternario, no && para condicionales

### 7. Rendimiento JavaScript (BAJO-MEDIO)

- `js-batch-dom-css` - Agrupá cambios CSS via clases o cssText
- `js-index-maps` - Construí Map para lookups repetidos
- `js-cache-property-access` - Cacheá propiedades de objetos en loops
- `js-cache-function-results` - Cacheá resultados de funciones en Map a nivel módulo
- `js-cache-storage` - Cacheá lecturas de localStorage/sessionStorage
- `js-combine-iterations` - Combiná múltiples filter/map en un loop
- `js-length-check-first` - Chequeá length del array antes de comparación costosa
- `js-early-exit` - Retorná temprano de funciones
- `js-hoist-regexp` - Sacá creación de RegExp fuera de loops
- `js-min-max-loop` - Usá loop para min/max en vez de sort
- `js-set-map-lookups` - Usá Set/Map para lookups O(1)
- `js-tosorted-immutable` - Usá toSorted() para inmutabilidad

### 8. Patrones Avanzados (BAJO)

- `advanced-event-handler-refs` - Guardá event handlers en refs
- `advanced-use-latest` - useLatest para refs de callbacks estables

## Cómo Usar

Leé archivos de reglas individuales para explicaciones detalladas y ejemplos de código:

```
rules/async-parallel.md
rules/bundle-barrel-imports.md
rules/_sections.md
```

Cada archivo de regla contiene:
- Explicación breve de por qué importa
- Ejemplo de código incorrecto con explicación
- Ejemplo de código correcto con explicación
- Contexto adicional y referencias

## Documento Compilado Completo

Para la guía completa con todas las reglas expandidas: `AGENTS.md`
