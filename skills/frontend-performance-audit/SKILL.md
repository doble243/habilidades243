---
name: frontend-performance-audit
description: Auditoría de performance visual y Core Web Vitals. Usá esta skill cuando el usuario diga "mejorá el rendimiento visual", "esto carga lento", "se siente pesado", "optimizá la carga", "el sitio está lento", o "mejorá los web vitals".
---

# Frontend Performance Audit

Auditoría completa de performance visual más allá del código React.

## Frases que Activan esta Skill

- "Mejorá el rendimiento visual"
- "Esto carga lento" / "El sitio está lento"
- "Se siente pesado" / "Se traba"
- "Optimizá la carga"
- "Mejorá los web vitals"
- "El LCP está alto"
- "Hay layout shifts"
- "Las imágenes cargan lento"

## Qué Audita esta Skill

### Core Web Vitals
- **LCP** (Largest Contentful Paint) - < 2.5s
- **CLS** (Cumulative Layout Shift) - < 0.1
- **INP** (Interaction to Next Paint) - < 200ms

### Performance Visual
- Imágenes mal dimensionadas
- Fuentes que bloquean renderizado
- Animaciones costosas
- Overdraw de sombras/blur
- Reflows innecesarios

## Checklist de Auditoría

### Imágenes
- [ ] Formato optimizado (WebP/AVIF)
- [ ] Dimensiones explícitas (width/height)
- [ ] Lazy loading para below-the-fold
- [ ] Srcset para responsive
- [ ] Priority para LCP image

### Fuentes
- [ ] font-display: swap
- [ ] Preload de fuentes críticas
- [ ] Subset de caracteres
- [ ] Formato woff2
- [ ] Fallback system fonts

### CSS/Animaciones
- [ ] No animar width/height/top/left
- [ ] Usar transform y opacity
- [ ] will-change solo cuando necesario
- [ ] Evitar box-shadow animado
- [ ] Reducir blur excesivo

### Layout
- [ ] Dimensiones reservadas para imágenes
- [ ] Skeleton loaders con tamaño fijo
- [ ] No insertar contenido above-the-fold
- [ ] Aspect-ratio para embeds

## Problemas Comunes y Soluciones

### CLS: Layout Shifts

```tsx
// ❌ Causa CLS
<img src="/hero.jpg" alt="Hero" />

// ✅ Sin CLS
<img 
  src="/hero.jpg" 
  alt="Hero"
  width={1200}
  height={600}
  className="aspect-[2/1]"
/>

// ✅ Con Next.js Image
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority // Para LCP
/>
```

### LCP: Carga Lenta

```tsx
// ❌ LCP lento
<img src="/hero.jpg" loading="lazy" />

// ✅ LCP optimizado
<Image
  src="/hero.jpg"
  priority
  fetchPriority="high"
  sizes="100vw"
/>

// Preload en head
<link rel="preload" href="/hero.jpg" as="image" />
```

### Fuentes que Bloquean

```css
/* ❌ Bloquea renderizado */
@font-face {
  font-family: 'CustomFont';
  src: url('/font.woff2');
}

/* ✅ No bloquea */
@font-face {
  font-family: 'CustomFont';
  src: url('/font.woff2') format('woff2');
  font-display: swap;
}
```

```tsx
// Next.js - preload automático
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'] })
```

### Animaciones Costosas

```css
/* ❌ Costoso - causa reflow */
.card:hover {
  width: 110%;
  box-shadow: 0 20px 40px rgba(0,0,0,0.3);
}

/* ✅ Performante - solo compositor */
.card {
  transition: transform 200ms, opacity 200ms;
}
.card:hover {
  transform: scale(1.02);
  opacity: 0.95;
}
```

### Sombras y Blur Pesados

```css
/* ❌ Overdraw costoso */
.overlay {
  backdrop-filter: blur(20px);
  box-shadow: 0 0 100px rgba(0,0,0,0.5);
}

/* ✅ Más liviano */
.overlay {
  backdrop-filter: blur(8px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}
```

## Ejemplo de Output

```
## Auditoría de Performance

### Core Web Vitals Estimados
- LCP: ~3.2s ⚠️ (objetivo: <2.5s)
- CLS: ~0.25 ❌ (objetivo: <0.1)
- INP: ~150ms ✅ (objetivo: <200ms)

### Problemas Detectados

**CRÍTICO:**
1. Hero image sin dimensiones (causa CLS)
2. Fuente custom sin font-display: swap
3. 5 imágenes de 2MB+ sin optimizar

**ALTO:**
4. Lazy loading en imagen above-the-fold
5. Animación de box-shadow en hover
6. backdrop-filter blur(20px) en modal

**MEDIO:**
7. Imágenes en formato PNG (deberían ser WebP)
8. Sin preload de fuente principal

### Correcciones Aplicadas
- ✅ Agregado width/height a todas las imágenes
- ✅ font-display: swap en @font-face
- ✅ priority en hero image
- ✅ Cambiado animación a transform
- ✅ Reducido blur a 8px

### Mejora Estimada
- LCP: 3.2s → ~1.8s
- CLS: 0.25 → ~0.05
```

## Herramientas de Verificación

```bash
# Lighthouse CLI
npx lighthouse https://tu-sitio.com --view

# Web Vitals en código
import { onCLS, onLCP, onINP } from 'web-vitals'
onCLS(console.log)
onLCP(console.log)
onINP(console.log)
```

## Presentar Resultados al Usuario

```
✓ Auditoría de performance completada

**Mejoras aplicadas:**
- Imágenes optimizadas (ahorro ~2MB)
- CLS eliminado con dimensiones explícitas
- LCP mejorado con priority loading
- Animaciones optimizadas para GPU

**Core Web Vitals mejorados:**
- LCP: 3.2s → 1.8s (-44%)
- CLS: 0.25 → 0.05 (-80%)

Ejecutá Lighthouse para verificar las mejoras.
```
