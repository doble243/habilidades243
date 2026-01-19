---
name: notion-mcp
description: Usá Notion MCP como cerebro del producto. Usá esta skill cuando el usuario diga "documentación en Notion", "manual en Notion", "centro de ayuda", "base de conocimiento", "wiki del producto", o "documentación viva".
---

# Notion MCP

Usá Notion como cerebro del producto: manual del cliente, centro de ayuda y documentación viva.

## Frases que Activan esta Skill

- "Documentación en Notion"
- "Creá el manual en Notion"
- "Centro de ayuda"
- "Base de conocimiento"
- "Wiki del producto"
- "Documentación viva"
- "Notion como cerebro"

## Arquitectura de Documentación

```
📚 NOTION WORKSPACE
│
├── 📖 Manual del Cliente (público)
│   ├── 🟢 Básico (no técnicos)
│   ├── 🟡 Intermedio (usuarios)
│   └── 🔴 Avanzado (admins)
│
├── 🔧 Centro de Ayuda
│   ├── FAQ
│   ├── Troubleshooting
│   └── Tutoriales
│
└── 📋 Documentación Interna
    ├── Arquitectura
    ├── APIs
    └── Procesos
```

## 1. Configuración Inicial

### Variables de Entorno

```env
NOTION_API_KEY=secret_xxx
NOTION_DATABASE_ID=xxx  # Base principal
NOTION_MANUAL_PAGE_ID=xxx  # Página raíz del manual
```

### Cliente Notion

```typescript
// lib/notion.ts
import { Client } from '@notionhq/client';

export const notion = new Client({
  auth: process.env.NOTION_API_KEY
});

// Helpers
export async function getPage(pageId: string) {
  return notion.pages.retrieve({ page_id: pageId });
}

export async function getBlocks(blockId: string) {
  const blocks = [];
  let cursor: string | undefined;
  
  do {
    const response = await notion.blocks.children.list({
      block_id: blockId,
      start_cursor: cursor,
      page_size: 100
    });
    blocks.push(...response.results);
    cursor = response.next_cursor ?? undefined;
  } while (cursor);
  
  return blocks;
}

export async function createPage(parentId: string, title: string, content: any[]) {
  return notion.pages.create({
    parent: { page_id: parentId },
    properties: {
      title: { title: [{ text: { content: title } }] }
    },
    children: content
  });
}
```

## 2. Estructura del Manual

### Crear Estructura Base

```typescript
// lib/notion-manual.ts
export async function createManualStructure(rootPageId: string) {
  // Nivel Básico
  const basicPage = await createPage(rootPageId, '🟢 Guía Básica', [
    heading1('Empezando'),
    paragraph('Esta guía te ayuda a usar el sistema paso a paso.'),
    divider(),
    heading2('¿Qué es este producto?'),
    paragraph('Explicación simple del propósito...'),
    heading2('Primeros pasos'),
    numberedList([
      'Creá tu cuenta',
      'Completá tu perfil',
      'Explorá el dashboard'
    ]),
    heading2('Funciones principales'),
    bulletList([
      '📦 Gestionar productos',
      '💳 Recibir pagos',
      '📊 Ver reportes'
    ])
  ]);
  
  // Nivel Intermedio
  const intermediatePage = await createPage(rootPageId, '🟡 Guía Intermedia', [
    heading1('Flujos Completos'),
    paragraph('Casos de uso reales y resolución de problemas.'),
    divider(),
    heading2('Caso: Vender un producto'),
    numberedList([
      'Crear el producto con fotos',
      'Configurar precio y stock',
      'Publicar en Mercado Libre',
      'Recibir y procesar el pago',
      'Coordinar envío'
    ]),
    heading2('Problemas comunes'),
    callout('⚠️', 'Si el pago queda pendiente más de 24hs...'),
    heading2('Tips avanzados')
  ]);
  
  // Nivel Avanzado
  const advancedPage = await createPage(rootPageId, '🔴 Guía Avanzada', [
    heading1('Documentación Técnica'),
    paragraph('Arquitectura, integraciones y automatizaciones.'),
    divider(),
    heading2('Arquitectura del Sistema'),
    codeBlock('Frontend: Next.js 14\nBackend: Supabase\nPagos: Mercado Pago\nVentas: Mercado Libre'),
    heading2('APIs e Integraciones'),
    heading2('Automatizaciones'),
    heading2('Configuración Avanzada')
  ]);
  
  return { basicPage, intermediatePage, advancedPage };
}
```

### Helpers para Bloques

```typescript
// lib/notion-blocks.ts
export function heading1(text: string) {
  return {
    type: 'heading_1',
    heading_1: { rich_text: [{ text: { content: text } }] }
  };
}

export function heading2(text: string) {
  return {
    type: 'heading_2',
    heading_2: { rich_text: [{ text: { content: text } }] }
  };
}

export function paragraph(text: string) {
  return {
    type: 'paragraph',
    paragraph: { rich_text: [{ text: { content: text } }] }
  };
}

export function bulletList(items: string[]) {
  return items.map(item => ({
    type: 'bulleted_list_item',
    bulleted_list_item: { rich_text: [{ text: { content: item } }] }
  }));
}

export function numberedList(items: string[]) {
  return items.map(item => ({
    type: 'numbered_list_item',
    numbered_list_item: { rich_text: [{ text: { content: item } }] }
  }));
}

export function callout(emoji: string, text: string) {
  return {
    type: 'callout',
    callout: {
      icon: { emoji },
      rich_text: [{ text: { content: text } }]
    }
  };
}

export function codeBlock(code: string, language = 'plain text') {
  return {
    type: 'code',
    code: {
      language,
      rich_text: [{ text: { content: code } }]
    }
  };
}

export function divider() {
  return { type: 'divider', divider: {} };
}

export function image(url: string, caption?: string) {
  return {
    type: 'image',
    image: {
      type: 'external',
      external: { url },
      caption: caption ? [{ text: { content: caption } }] : []
    }
  };
}
```

## 3. Centro de Ayuda (FAQ)

### Base de Datos de FAQ

```typescript
export async function createFAQDatabase(parentPageId: string) {
  return notion.databases.create({
    parent: { page_id: parentPageId },
    title: [{ text: { content: 'Preguntas Frecuentes' } }],
    properties: {
      Pregunta: { title: {} },
      Respuesta: { rich_text: {} },
      Categoría: {
        select: {
          options: [
            { name: 'Pagos', color: 'green' },
            { name: 'Productos', color: 'blue' },
            { name: 'Cuenta', color: 'purple' },
            { name: 'Envíos', color: 'orange' }
          ]
        }
      },
      Nivel: {
        select: {
          options: [
            { name: 'Básico', color: 'green' },
            { name: 'Intermedio', color: 'yellow' },
            { name: 'Avanzado', color: 'red' }
          ]
        }
      },
      Orden: { number: {} }
    }
  });
}

export async function addFAQ(databaseId: string, faq: {
  pregunta: string;
  respuesta: string;
  categoria: string;
  nivel: string;
}) {
  return notion.pages.create({
    parent: { database_id: databaseId },
    properties: {
      Pregunta: { title: [{ text: { content: faq.pregunta } }] },
      Respuesta: { rich_text: [{ text: { content: faq.respuesta } }] },
      Categoría: { select: { name: faq.categoria } },
      Nivel: { select: { name: faq.nivel } }
    }
  });
}
```

## 4. Sincronización con el Producto

### Actualizar Docs Cuando Cambia el Producto

```typescript
// lib/notion-sync.ts
export async function updateDocumentation(change: {
  type: 'feature' | 'fix' | 'breaking';
  title: string;
  description: string;
  affectedSections: string[];
}) {
  // Agregar a changelog
  await addToChangelog(change);
  
  // Marcar secciones que necesitan review
  for (const section of change.affectedSections) {
    await markForReview(section, change.title);
  }
  
  // Si es breaking change, notificar
  if (change.type === 'breaking') {
    await createBreakingChangeNotice(change);
  }
}

async function addToChangelog(change: any) {
  const changelogPageId = process.env.NOTION_CHANGELOG_PAGE_ID!;
  
  await notion.blocks.children.append({
    block_id: changelogPageId,
    children: [
      heading2(`${change.type === 'feature' ? '✨' : change.type === 'fix' ? '🐛' : '⚠️'} ${change.title}`),
      paragraph(change.description),
      paragraph(`📅 ${new Date().toLocaleDateString('es-AR')}`),
      divider()
    ]
  });
}
```

## 5. API para Frontend

```typescript
// app/api/docs/[slug]/route.ts
import { notion, getBlocks } from '@/lib/notion';
import { NextRequest, NextResponse } from 'next/server';

const DOCS_PAGES: Record<string, string> = {
  'guia-basica': process.env.NOTION_BASIC_PAGE_ID!,
  'guia-intermedia': process.env.NOTION_INTERMEDIATE_PAGE_ID!,
  'guia-avanzada': process.env.NOTION_ADVANCED_PAGE_ID!,
  'faq': process.env.NOTION_FAQ_PAGE_ID!
};

export async function GET(
  req: NextRequest,
  { params }: { params: { slug: string } }
) {
  const pageId = DOCS_PAGES[params.slug];
  
  if (!pageId) {
    return NextResponse.json({ error: 'Página no encontrada' }, { status: 404 });
  }
  
  try {
    const [page, blocks] = await Promise.all([
      notion.pages.retrieve({ page_id: pageId }),
      getBlocks(pageId)
    ]);
    
    return NextResponse.json({
      page,
      blocks,
      lastUpdated: page.last_edited_time
    });
  } catch (error) {
    return NextResponse.json({ error: 'Error cargando docs' }, { status: 500 });
  }
}
```

## 6. Componente de Documentación

```tsx
// components/NotionDoc.tsx
'use client';

import { useEffect, useState } from 'react';

interface NotionDocProps {
  slug: string;
}

export function NotionDoc({ slug }: NotionDocProps) {
  const [doc, setDoc] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(`/api/docs/${slug}`)
      .then(res => res.json())
      .then(setDoc)
      .finally(() => setLoading(false));
  }, [slug]);
  
  if (loading) return <DocSkeleton />;
  if (!doc) return <p>Error cargando documentación</p>;
  
  return (
    <article className="prose prose-sm max-w-none">
      {doc.blocks.map((block: any) => (
        <NotionBlock key={block.id} block={block} />
      ))}
    </article>
  );
}

function NotionBlock({ block }: { block: any }) {
  switch (block.type) {
    case 'heading_1':
      return <h1>{block.heading_1.rich_text[0]?.text.content}</h1>;
    case 'heading_2':
      return <h2>{block.heading_2.rich_text[0]?.text.content}</h2>;
    case 'paragraph':
      return <p>{block.paragraph.rich_text[0]?.text.content}</p>;
    case 'bulleted_list_item':
      return <li>{block.bulleted_list_item.rich_text[0]?.text.content}</li>;
    case 'callout':
      return (
        <div className="bg-amber-50 border-l-4 border-amber-400 p-4 my-4">
          <span className="mr-2">{block.callout.icon?.emoji}</span>
          {block.callout.rich_text[0]?.text.content}
        </div>
      );
    default:
      return null;
  }
}
```

## Checklist de Implementación

### Setup
- [ ] API key de Notion configurada
- [ ] Páginas base creadas
- [ ] IDs guardados en env

### Estructura
- [ ] Manual por niveles (básico/intermedio/avanzado)
- [ ] FAQ database
- [ ] Changelog

### Sincronización
- [ ] Updates automáticos
- [ ] Versionado de docs
- [ ] Notificaciones de cambios

### Frontend
- [ ] Renderizado de Notion blocks
- [ ] Navegación de docs
- [ ] Búsqueda

## Presentar Resultados al Usuario

```
✓ Notion MCP configurado

**Estructura creada:**
- 📖 Manual del Cliente (3 niveles)
- 🔧 Centro de Ayuda (FAQ)
- 📋 Documentación Interna

**Funcionalidades:**
- Docs sincronizados con el producto
- FAQ por categorías
- Changelog automático

**URLs de docs:**
- /docs/guia-basica
- /docs/guia-intermedia
- /docs/guia-avanzada
- /docs/faq

¿Querés que complete el contenido del manual?
```
