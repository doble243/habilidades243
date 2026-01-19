---
name: visual-polish-pass
description: Pasada final de pulido visual para producción. Usá esta skill cuando el usuario diga "dale una pasada final", "pulí la UI", "dejalo listo para producción", "últimos detalles", "refinamiento final", o "que se vea terminado".
---

# Visual Polish Pass

El último 10% que diferencia un proyecto "funciona" de uno "vende".

## Frases que Activan esta Skill

- "Dale una pasada final"
- "Pulí la UI" / "Pulido final"
- "Dejalo listo para producción"
- "Últimos detalles" / "Detalles finales"
- "Refinamiento final"
- "Que se vea terminado"
- "Esto ya está, solo falta pulir"
- "Hacé el quality pass"

## Qué Revisa esta Skill

### Spacing y Alineación
- Padding inconsistentes
- Margins desiguales
- Elementos desalineados
- Gaps arbitrarios

### Tipografía
- Tamaños de fuente inconsistentes
- Line-height apretado
- Truncado sin ellipsis
- Jerarquía confusa

### Colores y Contraste
- Contrastes pobres (< 4.5:1)
- Colores off-brand
- Grises inconsistentes
- Estados poco claros

### Componentes
- Inputs sin focus visible
- Botones sin jerarquía clara
- Estados hover/active pobres
- Iconos desalineados

### Estados
- Loading sin feedback
- Empty states faltantes
- Error states genéricos
- Disabled poco claro

## Checklist de Pulido

### Spacing (Sistema de 8px)
- [ ] Todos los paddings múltiplos de 4 o 8
- [ ] Gaps consistentes en grids/flex
- [ ] Secciones con breathing room
- [ ] Márgenes uniformes entre elementos similares

### Tipografía
- [ ] Máximo 3-4 tamaños de fuente distintos
- [ ] Line-height: 1.5 para body, 1.2-1.3 para headings
- [ ] Truncate con ellipsis donde corresponda
- [ ] Letter-spacing ajustado en headings grandes

### Bordes y Sombras
- [ ] Border-radius consistente
- [ ] Sombras con propósito (elevación)
- [ ] Borders sutiles (#e5e7eb, no #000)
- [ ] Dividers donde ayudan a separar

### Estados Interactivos
- [ ] Hover claro en todos los clickables
- [ ] Focus ring visible (outline-2 outline-offset-2)
- [ ] Active state (feedback de click)
- [ ] Disabled obviamente disabled (opacity-50 cursor-not-allowed)

### Feedback Visual
- [ ] Cursors correctos (pointer, not-allowed, grab)
- [ ] Transiciones suaves (150-200ms)
- [ ] Loading spinners donde hay espera
- [ ] Toast/notificaciones para acciones

## Antes y Después

### Spacing

```tsx
// ❌ Antes: inconsistente
<div className="p-3">
  <h2 className="mb-2">Title</h2>
  <p className="mb-4">Text</p>
  <button className="mt-3">Action</button>
</div>

// ✅ Después: sistema
<div className="p-6">
  <h2 className="mb-3">Title</h2>
  <p className="mb-6">Text</p>
  <button className="mt-4">Action</button>
</div>
```

### Inputs

```tsx
// ❌ Antes: básico
<input 
  className="border p-2 rounded"
/>

// ✅ Después: pulido
<input 
  className="w-full px-4 py-2.5 
             border border-gray-200 rounded-lg
             text-gray-900 placeholder:text-gray-400
             focus:outline-none focus:ring-2 focus:ring-blue-500/20 focus:border-blue-500
             disabled:bg-gray-50 disabled:text-gray-500 disabled:cursor-not-allowed
             transition-colors duration-150"
/>
```

### Botones

```tsx
// ❌ Antes: sin estados
<button className="bg-blue-500 text-white px-4 py-2 rounded">
  Submit
</button>

// ✅ Después: con jerarquía y estados
<button className="inline-flex items-center justify-center gap-2
                   px-4 py-2.5 
                   bg-blue-600 text-white font-medium rounded-lg
                   shadow-sm shadow-blue-600/25
                   hover:bg-blue-700 
                   focus:outline-none focus:ring-2 focus:ring-blue-500/50 focus:ring-offset-2
                   active:scale-[0.98]
                   disabled:opacity-50 disabled:cursor-not-allowed disabled:active:scale-100
                   transition-all duration-150">
  Submit
</button>
```

### Cards

```tsx
// ❌ Antes: plana
<div className="border p-4 rounded">
  <h3>Card Title</h3>
  <p>Description</p>
</div>

// ✅ Después: con profundidad
<div className="p-6 
                bg-white rounded-xl 
                border border-gray-100
                shadow-sm
                hover:shadow-md hover:border-gray-200
                transition-all duration-200">
  <h3 className="font-semibold text-gray-900">Card Title</h3>
  <p className="mt-2 text-sm text-gray-600 leading-relaxed">Description</p>
</div>
```

## Empty States

```tsx
// ✅ Empty state cuidado
<div className="flex flex-col items-center justify-center py-12 text-center">
  <div className="w-12 h-12 mb-4 rounded-full bg-gray-100 
                  flex items-center justify-center">
    <InboxIcon className="w-6 h-6 text-gray-400" />
  </div>
  <h3 className="font-medium text-gray-900">No items yet</h3>
  <p className="mt-1 text-sm text-gray-500">
    Get started by creating your first item.
  </p>
  <button className="mt-4 text-sm font-medium text-blue-600 hover:text-blue-700">
    Create item →
  </button>
</div>
```

## Focus Management

```tsx
// ✅ Focus visible accesible
<button className="focus:outline-none focus-visible:ring-2 
                   focus-visible:ring-blue-500 focus-visible:ring-offset-2">
  
// ✅ Skip link
<a href="#main" className="sr-only focus:not-sr-only 
                           focus:absolute focus:top-4 focus:left-4
                           focus:z-50 focus:px-4 focus:py-2 
                           focus:bg-white focus:rounded-lg focus:shadow-lg">
  Skip to content
</a>
```

## Presentar Resultados al Usuario

```
✓ Pulido visual completado

**Detalles refinados:**

Spacing:
- Normalizado a sistema de 8px
- Breathing room en secciones (+32px)
- Gaps consistentes en grids

Componentes:
- Inputs con focus rings
- Botones con estados completos
- Cards con hover elegante
- Empty states diseñados

Estados:
- Hover en todos los clickables
- Focus visible para a11y
- Disabled claramente marcado
- Loading feedback agregado

Tipografía:
- Jerarquía clara (3 niveles)
- Line-height optimizado
- Truncate con ellipsis

**El proyecto está listo para producción.**
```
