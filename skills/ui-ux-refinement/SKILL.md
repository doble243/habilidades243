---
name: ui-ux-refinement
description: Refiná y mejorá la UI/UX existente. Usá esta skill cuando el usuario pida "refiná la estética", "hacé el diseño más prolijo", "mejorá la jerarquía visual", "esto se ve muy básico, subilo de nivel", "elevá el diseño", "hacelo más premium", o "mejorá el look and feel".
---

# UI/UX Refinement

Revisá, analizá y refiná la UI existente para elevarla a nivel producto cuidado/premium.

## Frases que Activan esta Skill

- "Refiná la estética" / "Mejorá la estética"
- "Hacé el diseño más prolijo" / "Esto se ve desprolijo"
- "Mejorá la jerarquía visual"
- "Esto se ve muy básico, subilo de nivel"
- "Elevá el diseño" / "Hacelo más premium"
- "Mejorá el look and feel"
- "Esto parece un prototipo, refinalo"
- "Dale más cuidado visual"

## Qué Hace esta Skill

### 1. Revisar UI Existente
- Analizar estructura visual actual
- Identificar inconsistencias
- Detectar problemas de jerarquía

### 2. Proponer Mejoras Concretas
- Ajustes de spacing (padding, margin, gap)
- Mejoras de jerarquía visual (tamaños, pesos, colores)
- Ritmo visual (repetición, alineación, proximidad)
- Contraste y legibilidad

### 3. Elevar a Nivel Premium
- Refinamiento de detalles
- Microinteracciones sutiles
- Estados hover/focus elegantes
- Transiciones suaves

## Checklist de Refinamiento

### Spacing y Layout
- [ ] Padding consistente (8px grid system)
- [ ] Margins uniformes entre secciones
- [ ] Gap apropiado en flex/grid
- [ ] Alineación correcta de elementos

### Jerarquía Visual
- [ ] Títulos con peso y tamaño apropiado
- [ ] Texto secundario claramente diferenciado
- [ ] CTAs destacados visualmente
- [ ] Información agrupada lógicamente

### Contraste y Color
- [ ] Contraste mínimo 4.5:1 para texto
- [ ] Colores usados con intención
- [ ] Estados claros (hover, active, disabled)
- [ ] Fondos que no compiten con contenido

### Detalles Premium
- [ ] Border-radius consistente
- [ ] Sombras sutiles y con propósito
- [ ] Iconografía alineada y del tamaño correcto
- [ ] Espaciado generoso ("room to breathe")

## Ejemplo de Output

```
## Análisis de UI Actual

**Problemas detectados:**
1. Spacing inconsistente entre cards (16px, 24px, 20px)
2. Jerarquía de títulos confusa (h2 y h3 mismo tamaño)
3. Botones sin estados hover definidos
4. Contraste pobre en texto secundario

## Refinamientos Aplicados

### Spacing
- Normalizado gap entre cards a 24px
- Padding interno de cards: 20px → 24px
- Margen entre secciones: 48px consistente

### Jerarquía
- h2: text-2xl font-semibold
- h3: text-lg font-medium
- Texto secundario: text-sm text-muted-foreground

### Estados
- Hover en cards: scale-[1.02] shadow-lg transition-all
- Botones: hover:brightness-110 active:scale-95

### Contraste
- Texto secundario: gray-500 → gray-600
- Links: blue-500 → blue-600
```

## Reglas de Refinamiento

### DO (Hacer)
- Usar sistema de 4px/8px para spacing
- Mantener jerarquía clara con máximo 3 niveles
- Aplicar transiciones sutiles (150-300ms)
- Dejar espacio "para respirar"

### DON'T (Evitar)
- Sombras excesivas o muy oscuras
- Demasiados colores distintos
- Animaciones que distraen
- Elementos compitiendo por atención

## Integración con Tailwind

```tsx
// Antes: básico
<div className="p-4 bg-white rounded">
  <h2 className="text-xl">Título</h2>
  <p className="text-gray-500">Descripción</p>
</div>

// Después: refinado
<div className="p-6 bg-white rounded-xl shadow-sm border border-gray-100 
               hover:shadow-md transition-shadow duration-200">
  <h2 className="text-xl font-semibold text-gray-900">Título</h2>
  <p className="mt-2 text-sm text-gray-600 leading-relaxed">Descripción</p>
</div>
```

## Presentar Resultados al Usuario

```
✓ Refinamiento de UI completado

**Cambios aplicados:**
- Spacing normalizado al sistema de 8px
- Jerarquía visual mejorada
- Estados hover/focus añadidos
- Contraste optimizado para accesibilidad

**Archivos modificados:**
- components/Card.tsx
- components/Button.tsx
- app/page.tsx

El diseño ahora se ve más pulido y profesional.
```
