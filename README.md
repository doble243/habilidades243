# Agent Skills

> 🌐 [English version](README.en.md)

Una colección de skills para agentes de IA enfocados en código. Las skills son instrucciones y scripts empaquetados que extienden las capacidades del agente.

Las skills siguen el formato de [Agent Skills](https://agentskills.io/).

> **Nota:** Esta es una adaptación al español del repositorio original [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).

## Skills Disponibles

### react-best-practices

Guías de optimización de rendimiento para React y Next.js del equipo de ingeniería de Vercel. Contiene más de 40 reglas en 8 categorías, priorizadas por impacto.

**Usá esta skill cuando:**
- Escribas nuevos componentes React o páginas de Next.js
- Implementes data fetching (cliente o servidor)
- Revises código buscando problemas de rendimiento
- Quieras optimizar el tamaño del bundle o tiempos de carga

**Frases que activan esta skill:**
- "Revisá este componente React"
- "Optimizá el rendimiento de esta página"
- "Mejorá la performance de mi app"
- "Analizá el bundle size"
- "Refactorizá este código Next.js"

**Categorías cubiertas:**
- Eliminación de waterfalls (Crítico)
- Optimización del bundle (Crítico)
- Rendimiento server-side (Alto)
- Data fetching en cliente (Medio-Alto)
- Optimización de re-renders (Medio)
- Rendimiento de renderizado (Medio)
- Micro-optimizaciones de JavaScript (Bajo-Medio)

### web-design-guidelines

Revisá código de UI para verificar cumplimiento con buenas prácticas de interfaces web. Audita tu código contra más de 100 reglas de accesibilidad, rendimiento y UX.

**Frases que activan esta skill:**
- "Revisá mi UI"
- "Mirá la interfaz"
- "Auditá el diseño"
- "Chequeá la accesibilidad"
- "Revisá la experiencia de usuario"
- "Mejorá la estética"
- "Hacé el diseño más refinado"
- "Que se vea más premium"
- "Optimizá el UX"
- "Revisá contra buenas prácticas"

**Categorías cubiertas:**
- Accesibilidad (aria-labels, HTML semántico, handlers de teclado)
- Estados de foco (focus visible, patrones focus-visible)
- Formularios (autocomplete, validación, manejo de errores)
- Animaciones (prefers-reduced-motion, transforms GPU-friendly)
- Tipografía (comillas curvas, elipsis, tabular-nums)
- Imágenes (dimensiones, lazy loading, alt text)
- Rendimiento (virtualización, layout thrashing, preconnect)
- Navegación y Estado (URL refleja estado, deep-linking)
- Modo Oscuro y Temas (color-scheme, theme-color meta)
- Touch e Interacción (touch-action, tap-highlight)
- Locale e i18n (Intl.DateTimeFormat, Intl.NumberFormat)

### vercel-deploy-claimable

Desplegá aplicaciones y sitios web a Vercel instantáneamente. Diseñada para usar con claude.ai y Claude Desktop para hacer deploys directamente desde conversaciones. Los deployments son "reclamables" - los usuarios pueden transferir la propiedad a su propia cuenta de Vercel.

**Frases que activan esta skill:**
- "Desplegá mi app"
- "Hacé deploy de esto"
- "Subilo a producción"
- "Publicá esto"
- "Poné esto en vivo"
- "Dame el link del deploy"
- "Deploy this to production"

**Características:**
- Auto-detecta más de 40 frameworks desde `package.json`
- Devuelve URL de preview (sitio en vivo) y URL de claim (transferir propiedad)
- Maneja proyectos HTML estáticos automáticamente
- Excluye `node_modules` y `.git` de las subidas

**Cómo funciona:**
1. Empaqueta tu proyecto en un tarball
2. Detecta el framework (Next.js, Vite, Astro, etc.)
3. Sube al servicio de deployment
4. Devuelve URL de preview y URL de claim

**Salida:**
```
¡Deployment exitoso!

URL de Preview: https://skill-deploy-abc123.vercel.app
URL de Claim:   https://vercel.com/claim-deployment?code=...
```

---

## 🆕 Nuevas Skills para Agilizar Procesos

### ui-ux-refinement ⭐

Refiná y elevá la UI existente a nivel producto cuidado/premium. No solo audita, **propone y aplica mejoras concretas**.

**Frases que activan esta skill:**
- "Refiná la estética"
- "Hacé el diseño más prolijo"
- "Mejorá la jerarquía visual"
- "Esto se ve muy básico, subilo de nivel"
- "Elevá el diseño"

**Qué hace:**
- Ajusta spacing, jerarquía visual, ritmo, contraste
- Propone mejoras concretas en Tailwind/CSS
- Eleva el diseño de "funciona" a "premium"

---

### design-system-alignment ⭐

Alineá el proyecto a un design system consistente. Detecta si usás shadcn/ui, MUI, Chakra y normaliza todo.

**Frases que activan esta skill:**
- "Ordená los estilos"
- "Alineá el diseño"
- "Unificá botones y inputs"
- "Esto no parece un sistema"

**Qué hace:**
- Detecta design system existente
- Normaliza tamaños, radios, sombras, colores
- Elimina "estilos sueltos"

---

### frontend-performance-audit ⭐

Auditoría de performance visual real: Core Web Vitals, imágenes, fuentes, animaciones.

**Frases que activan esta skill:**
- "Mejorá el rendimiento visual"
- "Esto carga lento"
- "Se siente pesado"
- "Mejorá los web vitals"

**Qué cubre:**
- CLS (layout shifts)
- LCP (carga del contenido principal)
- Imágenes mal dimensionadas
- Fuentes que bloquean
- Animaciones costosas

---

### bundle-and-assets-optimizer ⭐

Optimizá bundle size, imágenes, fonts y assets. Lo que nadie toca pero impacta directo en UX.

**Frases que activan esta skill:**
- "Reducí el bundle"
- "Las imágenes pesan mucho"
- "Optimizá los assets"
- "El build es muy pesado"

**Qué hace:**
- Detecta imágenes enormes → propone WebP/AVIF
- Lazy loading real
- Font-display y subset de fuentes
- Tree-shaking y dynamic imports

---

### animation-and-motion-guidelines

Optimizá animaciones y motion design para una experiencia fluida.

**Frases que activan esta skill:**
- "Las animaciones están feas"
- "Esto se siente tosco"
- "Hacelo más fluido"
- "Mejorá las transiciones"

**Qué hace:**
- Detecta animaciones innecesarias/costosas
- Mejora easing y timing
- Implementa prefers-reduced-motion
- Agrega microinteracciones elegantes

---

### visual-polish-pass ⭐

El último 10% que diferencia un proyecto "funciona" de uno "vende".

**Frases que activan esta skill:**
- "Dale una pasada final"
- "Pulí la UI"
- "Dejalo listo para producción"
- "Últimos detalles"

**Qué revisa:**
- Padding inconsistentes
- Tipografías mal usadas
- Contrastes pobres
- Estados hover/focus pobres
- Empty states faltantes

---

### brand-tone-and-style

Ajustá el tono visual para alinearlo con la identidad de marca.

**Frases que activan esta skill:**
- "Quiero que se vea más premium"
- "Esto parece muy genérico"
- "Dale un tono más profesional"
- "Hacelo más tech/moderno/friendly"

**Tonos disponibles:**
- Premium/Luxury
- Tech/Modern
- Minimal/Clean
- Friendly/Playful
- Corporate/Professional

---

### task-decomposer 🧠

Analizá y planificá antes de codear. Ideal para requests grandes.

**Frases que activan esta skill:**
- "Optimizá todo el frontend"
- "Mejorá performance y UI"
- "Hacé un plan de mejoras"
- "Por dónde empiezo"

**Qué hace:**
- Analiza el proyecto
- Propone plan priorizado por impacto
- Identifica quick wins
- Ordena tareas por dependencias

---

### codebase-cleanup 🧠

Limpiá deuda técnica y visual del código.

**Frases que activan esta skill:**
- "Limpiá el código"
- "Hay mucha deuda técnica"
- "Sacá lo que no se usa"
- "Eliminá archivos muertos"

**Qué detecta:**
- Archivos muertos (componentes sin importar)
- Código no usado (funciones, variables, imports)
- Estilos duplicados
- CSS clases sin usar

---

## Instalación

```bash
npx add-skill vercel-labs/agent-skills
```

## Uso

Las skills están disponibles automáticamente una vez instaladas. El agente las usará cuando detecte tareas relevantes.

**Ejemplos de uso:**
```
Desplegá mi app
```
```
Revisá este componente React por problemas de rendimiento
```
```
Ayudame a optimizar esta página de Next.js
```
```
Mirá el diseño de la interfaz
```
```
Mejorá la experiencia de usuario de esta pantalla
```

## Estructura de una Skill

Cada skill contiene:
- `SKILL.md` - Instrucciones para el agente
- `scripts/` - Scripts de ayuda para automatización (opcional)
- `references/` - Documentación de soporte (opcional)

---

## 🚀 Prompts Esenciales

Prompts puntuales para aplicar lo indispensable en cualquier proyecto.

### Auditoría Rápida (Siempre Aplicar)

```
Hacé una auditoría completa del proyecto:
1. Revisá la estructura y organización del código
2. Identificá problemas de rendimiento críticos
3. Verificá buenas prácticas de React/Next.js
4. Evaluá la UI/UX y accesibilidad
5. Detectá code smells y deuda técnica

Dame un resumen ejecutivo con prioridades.
```

### 🔥 Boostear Proyecto (Mejora Integral)

```
Boosteá este proyecto al máximo:

**Rendimiento:**
- Eliminá waterfalls y optimizá fetching paralelo
- Reducí bundle size con imports específicos
- Aplicá lazy loading donde corresponda

**Código:**
- Refactorizá componentes siguiendo buenas prácticas
- Optimizá re-renders innecesarios
- Mejorá tipado y manejo de errores

**UI/UX:**
- Pulí la interfaz siguiendo guidelines de diseño web
- Mejorá feedback visual y estados de loading
- Asegurá consistencia visual

**Deploy:**
- Preparalo para producción
- Desplegalo y dame los links

Ejecutá todo lo que puedas automáticamente.
```

### 💡 Sugerencias de Boost

```
Analizá el proyecto y dame sugerencias concretas para boostearlo:

1. **Quick wins** - Mejoras de alto impacto con poco esfuerzo
2. **Optimizaciones de rendimiento** - Qué está lento y cómo arreglarlo
3. **Mejoras de UX** - Qué mejoraría la experiencia del usuario
4. **Deuda técnica** - Qué refactorizar para mantener el código sano
5. **Features recomendadas** - Qué agregaría valor al proyecto

Priorizá por impacto/esfuerzo. No implementes nada todavía, solo listá las sugerencias.
```

### Prompts Específicos por Área

| Área | Prompt |
|------|--------|
| **Rendimiento** | `Optimizá el rendimiento de este proyecto aplicando todas las reglas críticas` |
| **Bundle** | `Reducí el bundle size al mínimo posible` |
| **UI** | `Mejorá la UI para que se vea más profesional y pulida` |
| **UX** | `Mejorá la experiencia de usuario con mejor feedback y estados` |
| **Deploy** | `Preparalo para producción y desplegalo` |

---

## Licencia

MIT
# habilidades243
