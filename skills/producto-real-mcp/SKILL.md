---
name: producto-real-mcp
description: Construí productos digitales reales con MCPs. Usá esta skill cuando el usuario diga "producto completo", "sistema real", "listo para clientes", "producto SaaS", "MVP completo", o "producto para vender".
---

# Producto Real MCP

Construí productos digitales reales para clientes reales, integrando todos los MCPs necesarios.

## Frases que Activan esta Skill

- "Construí un producto completo"
- "Sistema listo para clientes"
- "Producto SaaS real"
- "MVP completo"
- "Producto para vender"
- "Modo producto real"

## Filosofía

```
PRIORIDADES (en orden):
1. CLARIDAD    → El usuario entiende qué hace
2. VELOCIDAD   → Carga rápido, responde rápido
3. ESTÉTICA    → Se ve profesional y cuidado
4. AUTOMATIZACIÓN → Reduce trabajo manual
```

## Stack Recomendado

```
┌─────────────────────────────────────────────────────┐
│                    FRONTEND                          │
│                   Next.js 14                         │
│         React + Tailwind + shadcn/ui                │
│              Mobile-First SIEMPRE                    │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│                 SUPABASE MCP                         │
│   Auth + Database + Storage + Edge Functions        │
└─────────────────────┬───────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
┌───────────┐  ┌───────────┐  ┌───────────┐
│ MERCADO   │  │ MERCADO   │  │  NOTION   │
│   PAGO    │  │   LIBRE   │  │   MCP     │
│   MCP     │  │    MCP    │  │           │
│ (Pagos)   │  │ (Ventas)  │  │  (Docs)   │
└───────────┘  └───────────┘  └───────────┘
```

## Backend Base (Supabase MCP)

1. **Activá la skill** `@skills/supabase-mcp/SKILL.md` para que el agente tome control del backend.
2. **Provisioná Supabase**:
   - `supabase init` en el repo local
   - Configurá `.env.local` con `NEXT_PUBLIC_SUPABASE_URL` y `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - Guardá la `SERVICE_ROLE_KEY` solo en el entorno de server / edge functions
3. **Esquema mínimo obligatorio** (usa el SQL del skill `supabase-mcp`): `profiles`, `products`, `orders`, más índices y RLS activada desde el día 1.
4. **Storage**: bucket `product-images` público para lectura y políticas de insert/delete autenticadas.
5. **Edge Functions**: crear carpeta `supabase/functions` y registrar funciones como `mercadopago-webhook` cuando haya integraciones.
6. **Checklist rápido** antes de seguir con frontend:
   - Migraciones aplicadas (`supabase db push`)
   - RLS habilitada en todas las tablas
   - Policies para CRUD del usuario real
   - Scripts de semilla para datos demo

> Si falta algo del backend, se reenvía al sub-skill `supabase-mcp` hasta que todo el checklist quede ✅.

## Checklist de Producto Real

### 1. Fundamentos

```markdown
□ Propuesta de valor clara en < 10 palabras
□ Landing page que convierte
□ Auth simple (email + Google)
□ Perfil de usuario completo
□ Dashboard principal útil
```

### 2. Core Features

```markdown
□ CRUD de la entidad principal
□ Búsqueda y filtros
□ Estados claros (activo, pendiente, etc)
□ Notificaciones importantes
□ Historial de acciones
```

### 3. Pagos (Mercado Pago MCP)

```markdown
□ Checkout sin fricción
□ Estados de pago visibles
□ Webhooks procesando
□ Emails de confirmación
□ Historial de transacciones
```

### 4. Integraciones (Si Aplica)

```markdown
□ Mercado Libre conectado
□ Sync bidireccional
□ Manejo de errores claro
□ Panel de estado de sync
```

### 5. Documentación (Notion MCP)

```markdown
□ Manual nivel básico
□ Manual nivel intermedio
□ Manual nivel avanzado
□ FAQ actualizado
□ Changelog vivo
```

### 6. Onboarding

```markdown
□ Flujo guiado para nuevos
□ Selección de objetivo
□ Primera acción completada
□ Links a documentación
□ Sin necesidad de soporte
```

### 7. Mobile First

```markdown
□ Todo funciona en 375px
□ Bottom navigation
□ Inputs de 48px altura
□ Sin dependencia de hover
□ Performance < 3s en 3G
```

## Estructura de Proyecto

```
app/
├── (auth)/
│   ├── login/
│   ├── register/
│   └── forgot-password/
├── (dashboard)/
│   ├── layout.tsx          # Con bottom nav
│   ├── page.tsx             # Dashboard principal
│   ├── productos/
│   ├── ventas/
│   ├── pagos/
│   └── configuracion/
├── (public)/
│   ├── page.tsx             # Landing
│   └── docs/
├── api/
│   ├── webhooks/
│   │   ├── mercadopago/
│   │   └── mercadolibre/
│   ├── pagos/
│   └── sync/
└── onboarding/

components/
├── ui/                      # shadcn/ui
├── layout/
│   ├── BottomNav.tsx
│   ├── Header.tsx
│   └── Sidebar.tsx
├── onboarding/
├── products/
└── payments/

lib/
├── supabase/
├── mercadopago.ts
├── mercadolibre.ts
├── notion.ts
└── utils.ts
```

## Flujo de Implementación

### Fase 1: Base (Día 1)

```bash
# 1. Crear proyecto
npx create-next-app@latest mi-producto --typescript --tailwind --app

# 2. Instalar dependencias
npm install @supabase/supabase-js mercadopago @notionhq/client
npx shadcn-ui@latest init

# 3. Configurar Supabase
# - Crear proyecto en supabase.com
# - Copiar keys a .env.local
# - Crear tablas base
```

### Fase 2: Auth + Core (Día 2)

```typescript
// Implementar:
// - Login/Register
// - Perfil de usuario
// - CRUD de entidad principal
// - Layout con navegación
```

### Fase 3: Pagos (Día 3)

```typescript
// Implementar:
// - Checkout de Mercado Pago
// - Webhook de notificaciones
// - Páginas de resultado
// - Historial de pagos
```

### Fase 4: Integraciones (Día 4)

```typescript
// Implementar:
// - OAuth de Mercado Libre
// - Sync de productos
// - Panel de administración
```

### Fase 5: Documentación + Onboarding (Día 5)

```typescript
// Implementar:
// - Estructura en Notion
// - Manual por niveles
// - Flujo de onboarding
// - Tour guiado
```

## Validación como Cliente

Después de implementar, validá todo el flujo como si fueras un cliente que no entiende nada de tecnología:

### Test de Usuario Nuevo

```markdown
□ ¿Puedo entender qué hace el producto en 5 segundos?
□ ¿Puedo registrarme sin ayuda?
□ ¿El onboarding me guía correctamente?
□ ¿Puedo completar la acción principal?
□ ¿Entiendo los estados y mensajes?
□ ¿Puedo pagar sin confundirme?
□ ¿Sé dónde buscar ayuda?
□ ¿Todo funciona en mi celular?
```

### Test de Flujo Crítico

```markdown
□ Registro → Onboarding → Primera acción → Pago → Confirmación
□ Cada paso tiene feedback visual
□ Los errores son claros y accionables
□ Puedo volver atrás sin perder datos
□ Recibo confirmaciones por email
```

## Métricas de Éxito

### Performance

| Métrica | Objetivo |
|---------|----------|
| FCP | < 1.5s |
| LCP | < 2.5s |
| TTI | < 3.5s |
| CLS | < 0.1 |

### Conversión

| Métrica | Objetivo |
|---------|----------|
| Registro completado | > 80% |
| Onboarding completado | > 60% |
| Primera acción | > 50% |
| Primera compra | > 20% |

### Soporte

| Métrica | Objetivo |
|---------|----------|
| Tickets por usuario | < 0.5 |
| Tiempo de resolución | < 24h |
| CSAT | > 4.5/5 |

## Presentar Resultados

```
✓ Producto listo para clientes reales

**Stack implementado:**
- Frontend: Next.js 14 + Tailwind + shadcn/ui
- Backend: Supabase (Auth, DB, Storage)
- Pagos: Mercado Pago integrado
- Ventas: Mercado Libre conectado
- Docs: Notion como cerebro

**Funcionalidades:**
- Auth completo (email + Google)
- CRUD de productos
- Checkout de pagos
- Sync con ML
- Onboarding automático
- Manual multimedia (3 niveles)

**Mobile-first:**
- Bottom navigation
- Inputs optimizados
- Performance < 2s

**Documentación:**
- Manual básico ✓
- Manual intermedio ✓
- Manual avanzado ✓
- FAQ ✓

**Validación:**
- Flujo testeado como usuario no-técnico
- Sin necesidad de soporte para empezar

🚀 Listo para recibir clientes
```

## Skills Relacionadas

Para implementar cada parte, usá las skills específicas:

- `supabase-mcp` → Backend y auth
- `mercadopago-mcp` → Pagos
- `mercadolibre-mcp` → Ventas y stock
- `notion-mcp` → Documentación
- `manual-multimedia` → Manuales por niveles
- `mobile-first-design` → UI/UX mobile
- `onboarding-flow` → Experiencia de bienvenida

## Comando Final de Validación

Después de implementar todo, pedile al agente:

> "Ahora validá todo el flujo como si fueras un cliente que no entiende nada de tecnología"

Esto activa una revisión completa de UX y detecta fricciones.
