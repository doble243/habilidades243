---
name: mobile-first-design
description: Diseñá mobile-first siempre. Usá esta skill cuando el usuario diga "hacelo mobile-first", "optimizá para móvil", "diseño responsive", "UI mobile", "funcione en celular", o "mobile siempre".
---

# Mobile First Design

Todo lo que construyas debe ser mobile-first. Si no funciona bien en mobile, se considera incorrecto.

## Frases que Activan esta Skill

- "Hacelo mobile-first"
- "Optimizá para móvil"
- "Diseño responsive"
- "UI mobile"
- "Que funcione en celular"
- "Mobile siempre"
- "Primero mobile"

## Principios Fundamentales

```
❌ DESKTOP FIRST          ✅ MOBILE FIRST
─────────────────────    ─────────────────────
Diseñar para 1920px      Diseñar para 375px
Agregar breakpoints      Expandir a desktop
Esconder en mobile       Mostrar lo esencial
Hover como principal     Touch como principal
```

## 1. Breakpoints Estratégicos

```css
/* Mobile First: empezar sin media query */
.component {
  /* Estilos base = mobile */
  padding: 16px;
  font-size: 16px;
}

/* Tablet */
@media (min-width: 640px) {
  .component {
    padding: 24px;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .component {
    padding: 32px;
    font-size: 14px; /* Puede ser menor en desktop */
  }
}
```

### Tailwind Mobile-First

```tsx
// ✅ CORRECTO: Mobile first
<div className="p-4 sm:p-6 lg:p-8">
  <h1 className="text-xl sm:text-2xl lg:text-3xl">
    Título
  </h1>
</div>

// ❌ INCORRECTO: Desktop first
<div className="p-8 md:p-6 sm:p-4">
  ...
</div>
```

## 2. Navegación con Una Mano

### Bottom Navigation (Recomendado)

```tsx
// components/BottomNav.tsx
export function BottomNav() {
  return (
    <nav className="fixed bottom-0 left-0 right-0 bg-white border-t 
                    safe-area-pb z-50 lg:hidden">
      <div className="flex justify-around items-center h-16">
        <NavItem href="/" icon={<Home />} label="Inicio" />
        <NavItem href="/productos" icon={<Package />} label="Productos" />
        <NavItem href="/ventas" icon={<DollarSign />} label="Ventas" />
        <NavItem href="/perfil" icon={<User />} label="Perfil" />
      </div>
    </nav>
  );
}

function NavItem({ href, icon, label }: NavItemProps) {
  const isActive = usePathname() === href;
  
  return (
    <Link 
      href={href}
      className={cn(
        "flex flex-col items-center justify-center w-16 h-full",
        "active:scale-95 transition-transform",
        isActive ? "text-primary" : "text-muted-foreground"
      )}
    >
      <span className="h-6 w-6">{icon}</span>
      <span className="text-xs mt-1">{label}</span>
    </Link>
  );
}
```

### Zonas de Alcance del Pulgar

```
┌─────────────────────────────┐
│      DIFÍCIL DE ALCANZAR    │  ← Evitar acciones frecuentes
│                             │
├─────────────────────────────┤
│                             │
│      ALCANCE MEDIO          │  ← Contenido principal
│                             │
├─────────────────────────────┤
│                             │
│      ZONA NATURAL           │  ← Acciones principales aquí
│      (Pulgar)               │
└─────────────────────────────┘
```

## 3. Inputs Grandes y Claros

```tsx
// components/MobileInput.tsx
export function MobileInput({ label, ...props }: InputProps) {
  return (
    <div className="space-y-2">
      <label className="text-sm font-medium text-gray-700">
        {label}
      </label>
      <input
        {...props}
        className={cn(
          // Tamaño mínimo para touch (44x44px)
          "h-12 w-full px-4",
          // Texto legible
          "text-base", // 16px mínimo para evitar zoom en iOS
          // Visual claro
          "border-2 rounded-xl",
          "focus:border-primary focus:ring-2 focus:ring-primary/20",
          // Feedback táctil
          "active:scale-[0.99] transition-transform"
        )}
      />
    </div>
  );
}
```

### Teclados Correctos

```tsx
// Email
<input type="email" inputMode="email" autoComplete="email" />

// Teléfono
<input type="tel" inputMode="tel" autoComplete="tel" />

// Números
<input type="text" inputMode="numeric" pattern="[0-9]*" />

// Búsqueda
<input type="search" enterKeyHint="search" />

// Precio
<input type="text" inputMode="decimal" />
```

## 4. Feedback Visual Inmediato

### Estados de Carga

```tsx
// components/LoadingButton.tsx
export function LoadingButton({ loading, children, ...props }: ButtonProps) {
  return (
    <Button 
      disabled={loading} 
      className="relative h-12 min-w-[120px]"
      {...props}
    >
      {loading && (
        <Loader2 className="absolute h-5 w-5 animate-spin" />
      )}
      <span className={loading ? 'opacity-0' : ''}>
        {children}
      </span>
    </Button>
  );
}
```

### Skeleton Screens

```tsx
// components/ProductCardSkeleton.tsx
export function ProductCardSkeleton() {
  return (
    <div className="animate-pulse space-y-3">
      <div className="aspect-square bg-gray-200 rounded-xl" />
      <div className="h-4 bg-gray-200 rounded w-3/4" />
      <div className="h-4 bg-gray-200 rounded w-1/2" />
    </div>
  );
}
```

### Pull to Refresh

```tsx
// components/PullToRefresh.tsx
'use client';

import { useRef, useState } from 'react';

export function PullToRefresh({ onRefresh, children }: Props) {
  const [refreshing, setRefreshing] = useState(false);
  const startY = useRef(0);
  
  const handleTouchStart = (e: TouchEvent) => {
    startY.current = e.touches[0].clientY;
  };
  
  const handleTouchEnd = async (e: TouchEvent) => {
    const pullDistance = e.changedTouches[0].clientY - startY.current;
    
    if (pullDistance > 80 && window.scrollY === 0) {
      setRefreshing(true);
      await onRefresh();
      setRefreshing(false);
    }
  };
  
  return (
    <div 
      onTouchStart={handleTouchStart}
      onTouchEnd={handleTouchEnd}
    >
      {refreshing && (
        <div className="flex justify-center py-4">
          <Loader2 className="h-6 w-6 animate-spin text-primary" />
        </div>
      )}
      {children}
    </div>
  );
}
```

## 5. Performance en Dispositivos Modestos

### Lazy Loading

```tsx
// Imágenes
<Image
  src={product.image}
  alt={product.title}
  loading="lazy"
  placeholder="blur"
  blurDataURL={shimmer}
/>

// Componentes pesados
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false
});
```

### Optimización de Listas

```tsx
// Para listas largas, usar virtualización
import { useVirtualizer } from '@tanstack/react-virtual';

function ProductList({ products }: { products: Product[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: products.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 120, // altura estimada de cada item
    overscan: 5
  });
  
  return (
    <div ref={parentRef} className="h-screen overflow-auto">
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualItem => (
          <ProductCard 
            key={virtualItem.key}
            product={products[virtualItem.index]}
            style={{
              position: 'absolute',
              top: virtualItem.start,
              height: virtualItem.size
            }}
          />
        ))}
      </div>
    </div>
  );
}
```

### Reducir JavaScript

```tsx
// Preferir Server Components cuando sea posible
// app/products/page.tsx (Server Component por defecto)
export default async function ProductsPage() {
  const products = await getProducts(); // Fetch en servidor
  
  return (
    <div className="grid grid-cols-2 gap-4 p-4">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

## 6. Gestos Táctiles

### Swipe Actions

```tsx
// components/SwipeableCard.tsx
'use client';

import { motion, useMotionValue, useTransform } from 'framer-motion';

export function SwipeableCard({ onDelete, children }: Props) {
  const x = useMotionValue(0);
  const background = useTransform(
    x,
    [-100, 0],
    ['rgb(239, 68, 68)', 'rgb(255, 255, 255)']
  );
  
  return (
    <motion.div
      style={{ x, background }}
      drag="x"
      dragConstraints={{ left: -100, right: 0 }}
      onDragEnd={(_, info) => {
        if (info.offset.x < -80) {
          onDelete();
        }
      }}
      className="relative rounded-xl p-4"
    >
      <div className="absolute right-4 top-1/2 -translate-y-1/2">
        <Trash2 className="h-5 w-5 text-white" />
      </div>
      {children}
    </motion.div>
  );
}
```

## 7. Checklist Mobile-First

### Layout
- [ ] Diseño base para 375px
- [ ] Bottom navigation visible
- [ ] Safe areas respetadas
- [ ] Sin scroll horizontal

### Touch
- [ ] Targets mínimo 44x44px
- [ ] Espaciado entre elementos táctiles
- [ ] Gestos intuitivos
- [ ] Sin dependencia de hover

### Performance
- [ ] Imágenes optimizadas (WebP)
- [ ] Lazy loading implementado
- [ ] Bundle < 200KB inicial
- [ ] FCP < 2s en 3G

### UX
- [ ] Feedback visual inmediato
- [ ] Estados de carga claros
- [ ] Errores amigables
- [ ] Inputs con teclado correcto

## Presentar Resultados al Usuario

```
✓ Diseño Mobile-First aplicado

**Optimizaciones:**
- Bottom navigation implementada
- Inputs de 48px de altura
- Targets táctiles de 44x44px mínimo
- Lazy loading en imágenes

**Performance:**
- Bundle inicial: 180KB
- FCP: 1.8s en 3G simulado
- Sin scroll horizontal

**Breakpoints:**
- Base: 375px (mobile)
- sm: 640px (tablet)
- lg: 1024px (desktop)

Todo funciona con una sola mano 👍
```
