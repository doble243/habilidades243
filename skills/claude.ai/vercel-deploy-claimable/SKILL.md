---
name: vercel-deploy
description: Desplegá aplicaciones y sitios web a Vercel. Usá esta skill cuando el usuario pida acciones de deployment como "Desplegá mi app", "Hacé deploy de esto", "Subilo a producción", "Publicá esto", "Poné esto en vivo", "Dame el link del deploy", o "Deploy this to production". No requiere autenticación - devuelve URL de preview y link de deployment reclamable.
metadata:
  author: vercel
  version: "1.0.0"
---

# Vercel Deploy

Desplegá cualquier proyecto a Vercel instantáneamente. No requiere autenticación.

## Frases que Activan esta Skill

Esta skill se activa con frases como:
- "Desplegá mi app" / "Hacé deploy de esto"
- "Subilo a producción" / "Publicá esto"
- "Poné esto en vivo" / "Dame el link del deploy"
- "Hacé un deploy de preview"
- "Subí esto a Vercel"
- "Lanzá esta app"
- "Deploy this to production"

## Cómo Funciona

1. Empaqueta tu proyecto en un tarball (excluye `node_modules` y `.git`)
2. Auto-detecta el framework desde `package.json`
3. Sube al servicio de deployment
4. Devuelve **URL de Preview** (sitio en vivo) y **URL de Claim** (transferir a tu cuenta de Vercel)

## Uso

```bash
bash /mnt/skills/user/vercel-deploy/scripts/deploy.sh [path]
```

**Argumentos:**
- `path` - Directorio a desplegar, o un archivo `.tgz` (por defecto: directorio actual)

**Ejemplos:**

```bash
# Desplegar directorio actual
bash /mnt/skills/user/vercel-deploy/scripts/deploy.sh

# Desplegar proyecto específico
bash /mnt/skills/user/vercel-deploy/scripts/deploy.sh /path/to/project

# Desplegar tarball existente
bash /mnt/skills/user/vercel-deploy/scripts/deploy.sh /path/to/project.tgz
```

## Salida

```
Preparando deployment...
Framework detectado: nextjs
Creando paquete de deployment...
Desplegando...
✓ ¡Deployment exitoso!

URL de Preview: https://skill-deploy-abc123.vercel.app
URL de Claim:   https://vercel.com/claim-deployment?code=...
```

El script también produce JSON a stdout para uso programático:

```json
{
  "previewUrl": "https://skill-deploy-abc123.vercel.app",
  "claimUrl": "https://vercel.com/claim-deployment?code=...",
  "deploymentId": "dpl_...",
  "projectId": "prj_..."
}
```

## Detección de Framework

El script auto-detecta frameworks desde `package.json`. Frameworks soportados incluyen:

- **React**: Next.js, Gatsby, Create React App, Remix, React Router
- **Vue**: Nuxt, Vitepress, Vuepress, Gridsome
- **Svelte**: SvelteKit, Svelte, Sapper
- **Otros Frontend**: Astro, Solid Start, Angular, Ember, Preact, Docusaurus
- **Backend**: Express, Hono, Fastify, NestJS, Elysia, h3, Nitro
- **Build Tools**: Vite, Parcel
- **Y más**: Blitz, Hydrogen, RedwoodJS, Storybook, Sanity, etc.

Para proyectos HTML estáticos (sin `package.json`), el framework se setea como `null`.

## Proyectos HTML Estáticos

Para proyectos sin `package.json`:
- Si hay un solo archivo `.html` que no se llama `index.html`, se renombra automáticamente
- Esto asegura que la página se sirva en la URL raíz (`/`)

## Presentar Resultados al Usuario

Siempre mostrá ambas URLs:

```
✓ ¡Deployment exitoso!

URL de Preview: https://skill-deploy-abc123.vercel.app
URL de Claim:   https://vercel.com/claim-deployment?code=...

Mirá tu sitio en la URL de Preview.
Para transferir este deployment a tu cuenta de Vercel, visitá la URL de Claim.
```

## Solución de Problemas

### Error de Red (Network Egress)

Si el deployment falla por restricciones de red (común en claude.ai), decile al usuario:

```
El deployment falló por restricciones de red. Para solucionarlo:

1. Andá a https://claude.ai/settings/capabilities
2. Agregá *.vercel.com a los dominios permitidos
3. Intentá desplegar de nuevo
```
