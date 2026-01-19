---
name: codebase-cleanup
description: Limpiá deuda técnica y visual del código. Usá esta skill cuando el usuario diga "limpiá el código", "hay mucha deuda técnica", "sacá lo que no se usa", "organizá el proyecto", "eliminá archivos muertos", o "refactorizá el desorden".
---

# Codebase Cleanup

Detectá y eliminá deuda técnica, archivos muertos y estilos duplicados.

## Frases que Activan esta Skill

- "Limpiá el código"
- "Hay mucha deuda técnica"
- "Sacá lo que no se usa"
- "Organizá el proyecto"
- "Eliminá archivos muertos"
- "Refactorizá el desorden"
- "Esto es un quilombo"
- "Necesita una limpieza"

## Qué Detecta esta Skill

### 1. Archivos Muertos
- Componentes sin importar
- Páginas sin ruta
- Utils no usados
- Assets sin referencia
- Tests sin correr

### 2. Código No Usado
- Funciones exportadas sin uso
- Variables declaradas sin leer
- Imports sin usar
- Props no utilizados
- CSS classes sin aplicar

### 3. Duplicación
- Componentes casi idénticos
- Estilos repetidos
- Lógica duplicada
- Utils redundantes

### 4. Inconsistencias
- Naming conventions mezclados
- Estructuras de carpetas inconsistentes
- Patrones diferentes para lo mismo

## Proceso de Cleanup

### Fase 1: Análisis

```
🔍 Escaneando proyecto...

📁 Estructura:
- src/components/: 45 archivos
- src/utils/: 12 archivos
- src/hooks/: 8 archivos
- src/styles/: 5 archivos

🗑️ Detectado:

ARCHIVOS MUERTOS (8)
├── components/OldButton.tsx (0 imports)
├── components/Deprecated/Header.tsx (0 imports)
├── utils/legacy.ts (0 imports)
├── hooks/useOldAuth.ts (0 imports)
├── styles/old-theme.css (0 imports)
└── ... 3 más

CÓDIGO NO USADO (23 items)
├── 12 funciones exportadas sin usar
├── 8 variables declaradas sin leer
└── 3 componentes internos sin renderizar

DUPLICACIÓN (5 casos)
├── Button.tsx ≈ PrimaryButton.tsx (92% similar)
├── formatDate() existe en 3 archivos
├── useToggle duplicado en 2 hooks
└── ... 2 más

CSS MUERTO (156 clases)
├── .old-header, .legacy-button, etc.
└── Tailwind: 45 clases custom sin usar
```

### Fase 2: Plan de Limpieza

```
📋 PLAN DE CLEANUP

SEGURO (no rompe nada):
✅ Eliminar 8 archivos muertos
✅ Eliminar 12 imports no usados
✅ Eliminar 156 clases CSS muertas
✅ Eliminar variables no leídas

REQUIERE REVISIÓN:
⚠️ Unificar Button + PrimaryButton
⚠️ Consolidar formatDate en utils
⚠️ Merge de hooks duplicados

RIESGOSO (verificar antes):
🔴 Componentes exportados pero sin import visible
🔴 Utils que podrían usarse dinámicamente
```

### Fase 3: Ejecución

```bash
# Detectar dead code
npx ts-prune

# Detectar CSS no usado
npx purgecss --config purgecss.config.js

# Detectar imports no usados
npx eslint --fix --rule 'no-unused-vars: error'

# Análisis de duplicación
npx jscpd ./src
```

## Patrones de Limpieza

### Archivos Muertos

```tsx
// ❌ Archivo sin usar - ELIMINAR
// src/components/OldButton.tsx
export function OldButton() { ... }

// Verificación:
// - Buscar "OldButton" en todo el proyecto
// - Si 0 resultados → eliminar
// - Si hay resultados → verificar si son comentarios/strings
```

### Imports No Usados

```tsx
// ❌ Antes
import { useState, useEffect, useCallback, useMemo } from 'react'
import { format, parse, addDays } from 'date-fns'
import { Button, Card, Modal, Toast } from '@/components'

function Component() {
  const [count, setCount] = useState(0) // Solo usa useState
  return <Button>{format(new Date(), 'PP')}</Button> // Solo usa format y Button
}

// ✅ Después
import { useState } from 'react'
import { format } from 'date-fns'
import { Button } from '@/components/Button'

function Component() {
  const [count, setCount] = useState(0)
  return <Button>{format(new Date(), 'PP')}</Button>
}
```

### Consolidar Duplicados

```tsx
// ❌ Duplicación
// utils/dateFormat.ts
export const formatDate = (d: Date) => format(d, 'PP')

// helpers/dates.ts
export const formatDateString = (d: Date) => format(d, 'PP')

// components/Card.tsx
const formatCardDate = (d: Date) => format(d, 'PP')

// ✅ Consolidado
// utils/date.ts
export const formatDate = (date: Date, pattern = 'PP') => 
  format(date, pattern)

// Uso en todos lados
import { formatDate } from '@/utils/date'
```

### CSS Cleanup

```css
/* ❌ Clases muertas */
.old-header { ... }
.legacy-button { ... }
.unused-modal { ... }

/* Detectar con PurgeCSS o revisión manual */
/* Si la clase no aparece en ningún archivo → eliminar */
```

## Herramientas de Análisis

```bash
# Dead exports
npx ts-prune

# Unused dependencies
npx depcheck

# Duplicated code
npx jscpd ./src --min-lines 5 --reporters console

# Bundle analysis
npx @next/bundle-analyzer

# CSS unused
npx purgecss --content './src/**/*.tsx' --css './src/**/*.css'
```

## Checklist de Cleanup

### Pre-Cleanup
- [ ] Commit actual del código
- [ ] Tests pasando
- [ ] Backup si es necesario

### Durante
- [ ] Eliminar archivos muertos
- [ ] Limpiar imports
- [ ] Consolidar duplicados
- [ ] Remover CSS muerto
- [ ] Actualizar index files

### Post-Cleanup
- [ ] Build exitoso
- [ ] Tests pasando
- [ ] Lint sin errores
- [ ] Revisar diff antes de commit

## Presentar Resultados al Usuario

```
✓ Cleanup completado

## Resumen

| Categoría | Antes | Después | Reducción |
|-----------|-------|---------|-----------|
| Archivos | 78 | 70 | -8 |
| Líneas de código | 12,450 | 11,200 | -10% |
| Clases CSS | 890 | 734 | -17% |
| Dependencias | 42 | 38 | -4 |

## Cambios Realizados

**Eliminado:**
- 8 archivos muertos
- 156 clases CSS sin usar
- 4 dependencias no usadas
- 23 funciones/variables muertas

**Consolidado:**
- Button + PrimaryButton → Button (con variants)
- 3 funciones formatDate → 1 en utils/date.ts
- 2 hooks useToggle → 1 en hooks/useToggle.ts

**Reorganizado:**
- Utils movidos a estructura consistente
- Componentes agrupados por feature

## Próximos Pasos

El proyecto está más limpio. Recomiendo:
1. Configurar ESLint para prevenir dead code
2. Agregar husky para lint pre-commit
3. CI check para imports no usados

¿Querés que configure alguno de estos?
```
