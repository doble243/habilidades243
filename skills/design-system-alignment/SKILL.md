---
name: design-system-alignment
description: Alineá el proyecto a un design system consistente. Usá esta skill cuando el usuario pida "ordená los estilos", "alineá el diseño", "unificá botones y inputs", "esto no parece un sistema", "normalizá los estilos", o "hacé que todo se vea consistente".
---

# Design System Alignment

Detectá inconsistencias y alineá el proyecto a un design system coherente.

## Frases que Activan esta Skill

- "Ordená los estilos" / "Normalizá los estilos"
- "Alineá el diseño" / "Unificá el diseño"
- "Unificá botones y inputs"
- "Esto no parece un sistema"
- "Hacé que todo se vea consistente"
- "Los estilos están desordenados"
- "Cada componente se ve distinto"
- "Creá un design system"

## Qué Hace esta Skill

### 1. Detectar Sistema Existente
- shadcn/ui
- Radix UI
- MUI / Material UI
- Chakra UI
- Ant Design
- Design tokens custom

### 2. Identificar Inconsistencias
- Tamaños de fuente diferentes
- Radios de borde variados
- Sombras inconsistentes
- Paleta de colores dispersa
- Spacing arbitrario

### 3. Normalizar y Unificar
- Definir tokens de diseño
- Estandarizar componentes
- Eliminar estilos sueltos
- Documentar sistema

## Checklist de Alineación

### Tipografía
- [ ] Font family único (o máximo 2)
- [ ] Escala tipográfica definida (xs, sm, base, lg, xl, 2xl)
- [ ] Line-height consistente
- [ ] Font-weight limitado (regular, medium, semibold, bold)

### Colores
- [ ] Paleta primaria definida
- [ ] Colores semánticos (success, warning, error, info)
- [ ] Grises consistentes
- [ ] Variables CSS / Tailwind config

### Espaciado
- [ ] Sistema de 4px/8px
- [ ] Tokens: xs(4), sm(8), md(16), lg(24), xl(32), 2xl(48)
- [ ] Uso consistente en todo el proyecto

### Bordes y Sombras
- [ ] Border-radius: none, sm, md, lg, full
- [ ] Sombras: sm, md, lg (máximo 3 niveles)
- [ ] Border colors unificados

### Componentes
- [ ] Botones con variantes definidas
- [ ] Inputs con estilos consistentes
- [ ] Cards con estructura uniforme
- [ ] Estados (hover, focus, disabled) estandarizados

## Detección Automática

```
Analizando proyecto...

✓ Detectado: shadcn/ui
✓ Tailwind CSS configurado
⚠ Inconsistencias encontradas:

COLORES:
- blue-500, blue-600, #3b82f6 usados indistintamente
- Grises: gray-100 a gray-900 + slate + zinc mezclados

SPACING:
- p-3, p-4, p-5, p-6 usados sin patrón
- gap-2, gap-3, gap-4, gap-5 mezclados

BORDER-RADIUS:
- rounded, rounded-md, rounded-lg, rounded-xl sin criterio

SOMBRAS:
- shadow, shadow-md, shadow-lg, shadow-xl inconsistentes
```

## Ejemplo de Normalización

### Antes (inconsistente)
```tsx
// Button A
<button className="px-4 py-2 bg-blue-500 rounded-md">

// Button B  
<button className="px-3 py-1.5 bg-blue-600 rounded">

// Button C
<button className="px-5 py-2.5 bg-[#3b82f6] rounded-lg">
```

### Después (alineado)
```tsx
// tailwind.config.js - tokens definidos
const tokens = {
  colors: {
    primary: {
      DEFAULT: 'hsl(var(--primary))',
      foreground: 'hsl(var(--primary-foreground))',
    }
  },
  borderRadius: {
    DEFAULT: '0.5rem', // 8px
  },
  spacing: {
    'btn-x': '1rem',   // 16px
    'btn-y': '0.5rem', // 8px
  }
}

// Todos los botones
<Button variant="primary" size="default">
```

## Estructura de Design Tokens

```css
/* globals.css */
:root {
  /* Colores */
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  --secondary: 210 40% 96%;
  --muted: 210 40% 96%;
  --accent: 210 40% 96%;
  
  /* Radios */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  
  /* Sombras */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
}
```

## Presentar Resultados al Usuario

```
✓ Design System alineado

**Tokens definidos:**
- 5 colores primarios + semánticos
- Escala tipográfica de 7 tamaños
- Sistema de spacing 4px
- 4 niveles de border-radius
- 3 niveles de sombras

**Componentes normalizados:**
- Button (4 variantes, 3 tamaños)
- Input (estados consistentes)
- Card (estructura unificada)

**Archivos actualizados:**
- tailwind.config.js
- globals.css
- components/ui/*

Ahora todo el proyecto sigue un sistema visual coherente.
```
