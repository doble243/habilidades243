---
name: brand-tone-and-style
description: Ajustá el tono visual y estilo de marca. Usá esta skill cuando el usuario diga "quiero que se vea más premium", "esto parece muy genérico", "dale un tono más profesional", "hacelo más moderno", "que se vea más tech", o "ajustá el branding".
---

# Brand Tone and Style

Detectá y ajustá el tono visual para alinearlo con la identidad de marca deseada.

## Frases que Activan esta Skill

- "Quiero que se vea más premium"
- "Esto parece muy genérico"
- "Dale un tono más profesional"
- "Hacelo más moderno" / "Más tech"
- "Que se vea más friendly"
- "Ajustá el branding"
- "Esto no refleja la marca"
- "Necesito un look más [X]"

## Tonos Visuales Disponibles

### 1. Premium / Luxury
- Colores oscuros, dorados, neutros sofisticados
- Tipografía serif o sans-serif elegante
- Mucho espacio en blanco
- Animaciones sutiles y lentas
- Imágenes de alta calidad

### 2. Tech / Modern
- Colores vibrantes, gradientes
- Tipografía geométrica sans-serif
- Bordes redondeados
- Animaciones fluidas
- Glassmorphism, blur effects

### 3. Minimal / Clean
- Paleta limitada (2-3 colores)
- Mucho espacio negativo
- Tipografía simple
- Sin decoraciones innecesarias
- Funcionalidad sobre forma

### 4. Friendly / Playful
- Colores saturados y cálidos
- Tipografía redondeada
- Bordes muy redondeados
- Ilustraciones/iconos custom
- Microinteracciones divertidas

### 5. Corporate / Professional
- Colores institucionales (azul, gris)
- Tipografía seria sans-serif
- Layout estructurado
- Fotografía profesional
- Tono formal en copy

### 6. Startup / Bold
- Colores de alta energía
- Tipografía bold
- CTAs agresivos
- Copy directo
- Movimiento y dinamismo

## Elementos de Tono

### Colores por Tono

```css
/* Premium */
--premium-bg: #0a0a0a;
--premium-text: #fafafa;
--premium-accent: #d4af37;

/* Tech */
--tech-primary: #6366f1;
--tech-secondary: #8b5cf6;
--tech-gradient: linear-gradient(135deg, #6366f1, #8b5cf6);

/* Minimal */
--minimal-bg: #ffffff;
--minimal-text: #171717;
--minimal-accent: #000000;

/* Friendly */
--friendly-primary: #f97316;
--friendly-secondary: #fbbf24;
--friendly-bg: #fffbeb;

/* Corporate */
--corp-primary: #1e40af;
--corp-secondary: #64748b;
--corp-bg: #f8fafc;
```

### Tipografía por Tono

```tsx
// Premium
font-family: 'Playfair Display', serif; // Headings
font-family: 'Inter', sans-serif; // Body

// Tech
font-family: 'Space Grotesk', sans-serif;
font-family: 'JetBrains Mono', monospace; // Code

// Minimal
font-family: 'Helvetica Neue', sans-serif;

// Friendly
font-family: 'Nunito', 'Poppins', sans-serif;

// Corporate
font-family: 'IBM Plex Sans', sans-serif;
```

### Border Radius por Tono

```css
/* Premium - sharp o muy sutil */
--radius: 2px;

/* Tech - medio */
--radius: 8px;

/* Minimal - ninguno o mínimo */
--radius: 0;

/* Friendly - muy redondeado */
--radius: 16px;

/* Corporate - sutil */
--radius: 4px;
```

## Transformación de Tono

### De Genérico a Premium

```tsx
// ❌ Genérico
<div className="bg-white p-4 rounded-lg shadow">
  <h2 className="text-xl font-bold text-gray-800">Our Services</h2>
  <p className="text-gray-600">We offer great services.</p>
</div>

// ✅ Premium
<div className="bg-neutral-950 p-8 border border-neutral-800">
  <h2 className="text-2xl font-light tracking-wide text-white">
    Our Services
  </h2>
  <p className="mt-4 text-neutral-400 leading-relaxed">
    Exceptional experiences, meticulously crafted.
  </p>
</div>
```

### De Genérico a Tech

```tsx
// ❌ Genérico
<button className="bg-blue-500 text-white px-4 py-2 rounded">
  Get Started
</button>

// ✅ Tech
<button className="relative px-6 py-3 
                   bg-gradient-to-r from-indigo-500 to-purple-500
                   text-white font-medium rounded-xl
                   shadow-lg shadow-indigo-500/25
                   hover:shadow-xl hover:shadow-indigo-500/30
                   hover:scale-[1.02]
                   transition-all duration-200">
  Get Started
  <span className="absolute inset-0 rounded-xl 
                   bg-gradient-to-r from-indigo-500 to-purple-500 
                   opacity-0 hover:opacity-20 blur-xl 
                   transition-opacity" />
</button>
```

### De Genérico a Friendly

```tsx
// ❌ Genérico
<div className="border p-4 rounded">
  <h3>Welcome!</h3>
  <p>Thanks for signing up.</p>
</div>

// ✅ Friendly
<div className="p-6 bg-amber-50 rounded-3xl border-2 border-amber-200">
  <h3 className="text-xl font-bold text-amber-900 flex items-center gap-2">
    Welcome! 🎉
  </h3>
  <p className="mt-2 text-amber-700">
    You're all set! Let's get started on something awesome.
  </p>
</div>
```

## Copy y Microtextos

### Por Tono

| Tono | CTA | Error | Empty State |
|------|-----|-------|-------------|
| **Premium** | "Discover" | "We apologize for the inconvenience" | "Nothing here yet" |
| **Tech** | "Get Started →" | "Something went wrong" | "No data found" |
| **Minimal** | "Continue" | "Error occurred" | "Empty" |
| **Friendly** | "Let's go! 🚀" | "Oops! Something broke" | "Nothing here yet! 🤷" |
| **Corporate** | "Learn More" | "An error has occurred" | "No records available" |

## Presentar Resultados al Usuario

```
✓ Tono visual ajustado: PREMIUM

**Cambios aplicados:**

Colores:
- Background: white → neutral-950
- Text: gray-800 → white/neutral-400
- Accents: blue-500 → gold/amber tones

Tipografía:
- Headings: font-light tracking-wide
- Body: leading-relaxed, más espacio
- Reducido bold, más elegancia

Espaciado:
- Más breathing room (+50%)
- Padding generoso
- Secciones con más separación

Detalles:
- Border-radius reducido (sharp = premium)
- Sombras eliminadas (bordes sutiles)
- Transiciones más lentas (300ms)

Copy:
- Tono más sofisticado
- Sin emojis
- Frases cortas y directas

**El proyecto ahora transmite una imagen premium y cuidada.**
```
