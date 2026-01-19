---
name: bundle-and-assets-optimizer
description: Optimizá bundle size, imágenes y fonts. Usá esta skill cuando el usuario diga "reducí el bundle", "las imágenes pesan mucho", "optimizá los assets", "el build es muy pesado", o "mejorá el tiempo de carga".
---

# Bundle and Assets Optimizer

Optimización integral de bundle size, imágenes, fuentes y assets.

## Frases que Activan esta Skill

- "Reducí el bundle" / "El bundle es muy grande"
- "Las imágenes pesan mucho"
- "Optimizá los assets"
- "El build es muy pesado"
- "Mejorá el tiempo de carga"
- "Tree shaking" / "Code splitting"
- "Lazy loading de imágenes"
- "Comprimí las fuentes"

## Qué Optimiza esta Skill

### 1. Bundle JavaScript
- Tree shaking efectivo
- Code splitting
- Dynamic imports
- Eliminación de dead code

### 2. Imágenes
- Conversión a WebP/AVIF
- Compresión sin pérdida visible
- Responsive images (srcset)
- Lazy loading inteligente

### 3. Fuentes
- Subsetting de caracteres
- Formato woff2
- font-display optimizado
- Preload estratégico

### 4. CSS
- Purge de estilos no usados
- Minificación
- Critical CSS inline

## Checklist de Optimización

### Bundle JS
- [ ] Imports específicos (no barrel files)
- [ ] Dynamic import para rutas
- [ ] Dynamic import para componentes pesados
- [ ] Analizar bundle con @next/bundle-analyzer

### Imágenes
- [ ] Convertir PNG/JPG → WebP
- [ ] Imágenes > 100KB comprimidas
- [ ] width/height explícitos
- [ ] loading="lazy" para below-fold
- [ ] Usar next/image o similar

### Fuentes
- [ ] Solo caracteres necesarios (latin)
- [ ] Formato woff2 únicamente
- [ ] Máximo 2-3 weights
- [ ] font-display: swap

### CSS
- [ ] PurgeCSS / Tailwind purge activo
- [ ] No CSS inline excesivo
- [ ] Critical CSS para above-fold

## Optimización de Imports

### Barrel Files (evitar)

```tsx
// ❌ Importa TODO el barrel
import { Button } from '@/components'
import { format } from 'date-fns'
import { debounce } from 'lodash'

// ✅ Imports específicos
import { Button } from '@/components/ui/button'
import format from 'date-fns/format'
import debounce from 'lodash/debounce'
```

### Dynamic Imports

```tsx
// ❌ Import estático de componente pesado
import HeavyChart from '@/components/HeavyChart'

// ✅ Dynamic import
import dynamic from 'next/dynamic'
const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false
})
```

### Route-based Splitting

```tsx
// Next.js App Router - automático por carpeta
// pages/dashboard/page.tsx se carga solo cuando se visita

// Para componentes dentro de la página:
const DashboardStats = dynamic(() => import('./DashboardStats'))
```

## Optimización de Imágenes

### Conversión de Formato

```tsx
// ❌ Formatos pesados
<img src="/hero.png" /> // 2.4MB
<img src="/photo.jpg" /> // 800KB

// ✅ Formatos modernos
<Image
  src="/hero.webp" // 400KB (-83%)
  alt="Hero"
  width={1200}
  height={600}
/>

// Con fallback
<picture>
  <source srcset="/hero.avif" type="image/avif" />
  <source srcset="/hero.webp" type="image/webp" />
  <img src="/hero.jpg" alt="Hero" />
</picture>
```

### Responsive Images

```tsx
<Image
  src="/hero.jpg"
  alt="Hero"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  fill
  className="object-cover"
/>
```

## Optimización de Fuentes

### Subset y Formato

```css
/* ❌ Fuente completa */
@font-face {
  font-family: 'Inter';
  src: url('/inter.ttf'); /* 300KB */
}

/* ✅ Subset optimizado */
@font-face {
  font-family: 'Inter';
  src: url('/inter-latin.woff2') format('woff2'); /* 20KB */
  font-display: swap;
  unicode-range: U+0000-00FF; /* Latin básico */
}
```

### Next.js Font Optimization

```tsx
import { Inter } from 'next/font/google'

const inter = Inter({ 
  subsets: ['latin'],
  display: 'swap',
  preload: true,
  weight: ['400', '500', '600'] // Solo los necesarios
})
```

## Análisis de Bundle

```bash
# Instalar analyzer
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
})
module.exports = withBundleAnalyzer(nextConfig)

# Ejecutar
ANALYZE=true npm run build
```

## Ejemplo de Output

```
## Análisis de Assets

### Bundle JavaScript
- Total: 450KB → 280KB (-38%)
- Chunks optimizados: 12
- Dynamic imports agregados: 5

### Imágenes
- Total: 8.5MB → 1.2MB (-86%)
- Convertidas a WebP: 15 archivos
- Lazy loading aplicado: 12 imágenes

### Fuentes
- Total: 450KB → 45KB (-90%)
- Subset aplicado: latin only
- Weights reducidos: 6 → 3

### CSS
- Total: 120KB → 35KB (-71%)
- Clases purgadas: 2,400
```

## Presentar Resultados al Usuario

```
✓ Optimización de assets completada

**Reducción total: 65%**

| Asset      | Antes   | Después | Ahorro |
|------------|---------|---------|--------|
| JavaScript | 450KB   | 280KB   | -38%   |
| Imágenes   | 8.5MB   | 1.2MB   | -86%   |
| Fuentes    | 450KB   | 45KB    | -90%   |
| CSS        | 120KB   | 35KB    | -71%   |

**Cambios aplicados:**
- 5 dynamic imports para code splitting
- 15 imágenes convertidas a WebP
- Fuentes con subset latin
- PurgeCSS activado

Tiempo de carga estimado: 4.2s → 1.5s
```
