---
name: task-decomposer
description: Analizá y descomponé tareas antes de codear. Usá esta skill cuando el usuario diga "optimizá todo el frontend", "mejorá performance y UI", "hacé un plan de mejoras", "qué hay que hacer", "priorizá las tareas", o "analizá antes de tocar código".
---

# Task Decomposer

Analizá, planificá y priorizá antes de escribir código.

## Frases que Activan esta Skill

- "Optimizá todo el frontend"
- "Mejorá performance y UI"
- "Hacé un plan de mejoras"
- "Qué hay que hacer"
- "Priorizá las tareas"
- "Analizá antes de tocar código"
- "Dividí esto en pasos"
- "Por dónde empiezo"

## Cuándo Usar esta Skill

### Ideal Para:
- Requests grandes y vagos ("mejorá todo")
- Múltiples áreas a tocar
- Proyectos nuevos o desconocidos
- Antes de refactors grandes
- Cuando no está claro el alcance

### No Necesario Para:
- Cambios puntuales
- Bugs específicos
- Requests con scope claro

## Proceso de Análisis

### 1. Descubrimiento
```
Analizando proyecto...

📁 Estructura detectada:
- Framework: Next.js 14 (App Router)
- Styling: Tailwind CSS + shadcn/ui
- State: Zustand
- Fetching: SWR

📊 Métricas iniciales:
- Componentes: 45
- Páginas: 12
- Bundle size: ~450KB
- Dependencias: 38
```

### 2. Identificación de Áreas

```
🔍 Áreas de mejora detectadas:

PERFORMANCE (Impacto: ALTO)
├── Bundle size elevado (450KB)
├── Imágenes sin optimizar (8 archivos, 12MB)
├── Sin lazy loading en rutas
└── Waterfalls en fetching

UI/UX (Impacto: ALTO)
├── Inconsistencias de spacing
├── Estados hover faltantes
├── Sin loading states
└── Mobile no responsive

CÓDIGO (Impacto: MEDIO)
├── Componentes muy grandes (>300 líneas)
├── Props drilling en 3+ niveles
├── useEffect sin cleanup
└── Tipos any en 12 lugares

ACCESIBILIDAD (Impacto: MEDIO)
├── Sin alt en 5 imágenes
├── Contraste bajo en textos secundarios
├── Sin focus visible en links
└── Sin skip link
```

### 3. Priorización

```
📋 PLAN DE EJECUCIÓN

FASE 1: Quick Wins (30 min) - Impacto inmediato
┌─────────────────────────────────────────────┐
│ 1. Agregar loading states                   │
│ 2. Arreglar contraste de textos             │
│ 3. Agregar alt a imágenes                   │
│ 4. Hover states en botones                  │
└─────────────────────────────────────────────┘

FASE 2: Performance (1-2 hrs) - Mayor ganancia
┌─────────────────────────────────────────────┐
│ 1. Optimizar imágenes (WebP + sizes)        │
│ 2. Dynamic imports en rutas pesadas         │
│ 3. Parallelizar fetching                    │
│ 4. Lazy load below-fold                     │
└─────────────────────────────────────────────┘

FASE 3: UI Polish (1-2 hrs) - Calidad visual
┌─────────────────────────────────────────────┐
│ 1. Normalizar spacing (8px system)          │
│ 2. Consistencia en componentes              │
│ 3. Transiciones y animaciones               │
│ 4. Responsive fixes                         │
└─────────────────────────────────────────────┘

FASE 4: Refactor (2-3 hrs) - Mantenibilidad
┌─────────────────────────────────────────────┐
│ 1. Dividir componentes grandes              │
│ 2. Extraer hooks custom                     │
│ 3. Agregar tipos                            │
│ 4. Cleanup de useEffects                    │
└─────────────────────────────────────────────┘
```

## Matriz de Priorización

```
              ESFUERZO
              Bajo        Alto
         ┌──────────┬──────────┐
    Alto │ HACER    │ PLANEAR  │
IMPACTO  │ PRIMERO  │          │
         ├──────────┼──────────┤
    Bajo │ RELLENO  │ EVITAR   │
         │          │          │
         └──────────┴──────────┘
```

### Clasificación de Tareas

| Tarea | Impacto | Esfuerzo | Prioridad |
|-------|---------|----------|-----------|
| Loading states | Alto | Bajo | 🔴 P0 |
| Optimizar imágenes | Alto | Bajo | 🔴 P0 |
| Hover states | Medio | Bajo | 🟡 P1 |
| Lazy loading | Alto | Medio | 🟡 P1 |
| Refactor componentes | Medio | Alto | 🟢 P2 |
| Tipos TS | Bajo | Alto | 🔵 P3 |

## Formato de Salida

### Resumen Ejecutivo
```
📊 ANÁLISIS COMPLETADO

Estado actual: 6/10
Estado potencial: 9/10

Tiempo estimado total: 5-7 horas
Quick wins disponibles: 4 (30 min)
Bloqueantes: 0

Recomendación: Empezar por Fase 1 (Quick Wins)
para mostrar progreso inmediato.
```

### Árbol de Dependencias
```
Orden de ejecución:

1. Loading states (sin dependencias)
   ↓
2. Optimizar imágenes (sin dependencias)
   ↓
3. Lazy loading (después de imágenes)
   ↓
4. Spacing system (sin dependencias)
   ↓
5. Componentes UI (después de spacing)
   ↓
6. Refactor (al final, puede romper cosas)
```

## Preguntas de Clarificación

Cuando el scope no es claro, preguntar:

```
Antes de empezar, necesito clarificar:

1. ¿Cuál es la prioridad?
   □ Performance (velocidad de carga)
   □ UI/UX (apariencia y usabilidad)
   □ Código (mantenibilidad)
   □ Todo (me das tiempo)

2. ¿Hay deadline?
   □ ASAP (solo quick wins)
   □ Esta semana (fases 1-2)
   □ Sin prisa (plan completo)

3. ¿Qué no debo tocar?
   (componentes, páginas, estilos específicos)
```

## Presentar Resultados al Usuario

```
✓ Análisis completado

## Resumen

He analizado el proyecto y encontré **15 oportunidades de mejora**
en 4 áreas principales.

## Plan Propuesto

**Fase 1 - Quick Wins (30 min)**
- 4 tareas de alto impacto y bajo esfuerzo

**Fase 2 - Performance (1-2 hrs)**
- Optimización de assets e imágenes
- Code splitting y lazy loading

**Fase 3 - UI Polish (1-2 hrs)**
- Consistencia visual
- Estados y transiciones

**Fase 4 - Refactor (2-3 hrs)**
- Mejoras de código
- Tipos y tests

## ¿Cómo seguimos?

1. "Ejecutá la Fase 1" - Empiezo con quick wins
2. "Hacé todo" - Ejecuto el plan completo
3. "Solo performance" - Me enfoco en velocidad
4. "Explicame más sobre [área]" - Detallo específico

¿Qué preferís?
```
