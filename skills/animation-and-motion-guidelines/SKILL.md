---
name: animation-and-motion-guidelines
description: Optimizá animaciones y motion design. Usá esta skill cuando el usuario diga "las animaciones están feas", "esto se siente tosco", "hacelo más fluido", "mejorá las transiciones", "las animaciones se traban", o "agregá microinteracciones".
---

# Animation and Motion Guidelines

Mejorá animaciones, transiciones y microinteracciones para una experiencia fluida.

## Frases que Activan esta Skill

- "Las animaciones están feas" / "Se ven toscas"
- "Esto se siente tosco" / "No es fluido"
- "Hacelo más fluido" / "Suavizá las transiciones"
- "Mejorá las transiciones"
- "Las animaciones se traban"
- "Agregá microinteracciones"
- "El motion está mal"
- "Se siente robótico"

## Principios de Motion Design

### 1. Propósito
Cada animación debe tener un motivo:
- Feedback de acción
- Guiar atención
- Mostrar relación espacial
- Suavizar cambios de estado

### 2. Duración Apropiada
- **Micro** (hover, clicks): 100-200ms
- **Pequeña** (tooltips, dropdowns): 200-300ms
- **Media** (modales, sidebars): 300-400ms
- **Grande** (page transitions): 400-600ms

### 3. Easing Correcto
- **ease-out**: Elementos que entran (aparecen rápido, frenan suave)
- **ease-in**: Elementos que salen (arrancan suave, salen rápido)
- **ease-in-out**: Elementos que se mueven en pantalla

## Checklist de Animaciones

### Performance
- [ ] Solo animar transform y opacity
- [ ] Evitar animaciones de layout (width, height, top, left)
- [ ] No animar box-shadow directamente
- [ ] Usar will-change con moderación
- [ ] Respetar prefers-reduced-motion

### UX
- [ ] Animaciones con propósito claro
- [ ] Duración apropiada al contexto
- [ ] Easing que se siente natural
- [ ] No bloquear interacción
- [ ] Feedback inmediato en acciones

### Accesibilidad
- [ ] prefers-reduced-motion implementado
- [ ] No depender solo de animación para comunicar
- [ ] Evitar parpadeos y flashes
- [ ] Pausable si dura >5s

## Patrones Correctos

### Hover States

```tsx
// ❌ Tosco
<button className="hover:bg-blue-600">
  Click
</button>

// ✅ Fluido
<button className="transition-colors duration-150 ease-out hover:bg-blue-600">
  Click
</button>

// ✅ Con escala sutil
<button className="transition-all duration-150 ease-out 
                   hover:bg-blue-600 hover:scale-[1.02] 
                   active:scale-[0.98]">
  Click
</button>
```

### Modales y Overlays

```tsx
// ❌ Aparece de golpe
{isOpen && <Modal />}

// ✅ Con Framer Motion
<AnimatePresence>
  {isOpen && (
    <>
      <motion.div
        className="fixed inset-0 bg-black/50"
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={{ opacity: 0 }}
        transition={{ duration: 0.2 }}
      />
      <motion.div
        className="modal"
        initial={{ opacity: 0, scale: 0.95, y: 10 }}
        animate={{ opacity: 1, scale: 1, y: 0 }}
        exit={{ opacity: 0, scale: 0.95, y: 10 }}
        transition={{ duration: 0.2, ease: 'easeOut' }}
      >
        {children}
      </motion.div>
    </>
  )}
</AnimatePresence>
```

### Listas y Stagger

```tsx
// ✅ Items que aparecen en secuencia
<motion.ul>
  {items.map((item, i) => (
    <motion.li
      key={item.id}
      initial={{ opacity: 0, x: -10 }}
      animate={{ opacity: 1, x: 0 }}
      transition={{ 
        delay: i * 0.05, // Stagger de 50ms
        duration: 0.2 
      }}
    >
      {item.name}
    </motion.li>
  ))}
</motion.ul>
```

### Loading States

```tsx
// ✅ Skeleton con shimmer
<div className="animate-pulse bg-gray-200 rounded" />

// ✅ Spinner suave
<motion.div
  className="w-6 h-6 border-2 border-blue-500 border-t-transparent rounded-full"
  animate={{ rotate: 360 }}
  transition={{ duration: 1, repeat: Infinity, ease: 'linear' }}
/>
```

## Accesibilidad: Reduced Motion

```tsx
// CSS
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

// Tailwind
<div className="motion-safe:animate-bounce motion-reduce:animate-none" />

// Framer Motion
const prefersReducedMotion = usePrefersReducedMotion()

<motion.div
  animate={{ x: 100 }}
  transition={prefersReducedMotion ? { duration: 0 } : { duration: 0.3 }}
/>
```

## Easing Reference

```css
/* Naturales */
--ease-out: cubic-bezier(0, 0, 0.2, 1);      /* Entradas */
--ease-in: cubic-bezier(0.4, 0, 1, 1);        /* Salidas */
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);  /* Movimiento */

/* Con bounce */
--ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1);

/* Suave */
--ease-smooth: cubic-bezier(0.22, 1, 0.36, 1);
```

## Errores Comunes

### ❌ Animar propiedades costosas
```css
.card:hover {
  width: 110%;           /* Causa reflow */
  box-shadow: 0 20px 40px /* Muy costoso */
}
```

### ✅ Usar transform
```css
.card {
  transition: transform 200ms ease-out;
}
.card:hover {
  transform: scale(1.02);
}
```

### ❌ Duración incorrecta
```css
.tooltip {
  transition: opacity 800ms; /* Muy lento para tooltip */
}
```

### ✅ Duración apropiada
```css
.tooltip {
  transition: opacity 150ms ease-out;
}
```

## Presentar Resultados al Usuario

```
✓ Animaciones optimizadas

**Mejoras aplicadas:**
- Transiciones de 150-300ms (antes: 0 o 500ms+)
- Easing natural (ease-out para entradas)
- Solo transform/opacity animados
- prefers-reduced-motion implementado

**Microinteracciones agregadas:**
- Hover states con feedback visual
- Botones con active:scale-95
- Modales con fade + scale suave
- Loading spinners fluidos

**Performance:**
- Eliminadas animaciones de layout
- Removidos box-shadow animados
- GPU-accelerated transforms

El sitio ahora se siente más pulido y responsivo.
```
