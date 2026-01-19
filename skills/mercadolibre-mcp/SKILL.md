---
name: mercadolibre-mcp
description: Integrá Mercado Libre MCP para catálogo, órdenes y stock. Usá esta skill cuando el usuario diga "sincronizá con ML", "publicar en Mercado Libre", "stock ML", "órdenes de ML", "catálogo ML", o "vender en Mercado Libre".
---

# Mercado Libre MCP

Integrá catálogo, órdenes y stock con Mercado Libre usando MCP.

## Frases que Activan esta Skill

- "Sincronizá con Mercado Libre"
- "Publicar productos en ML"
- "Traé órdenes de ML"
- "Sincronizar stock"
- "Catálogo de Mercado Libre"
- "Vender en Mercado Libre"
- "Conectar con ML"

## Arquitectura de Integración

```
┌─────────────────────────────────────────────────────┐
│                   TU SISTEMA                         │
│                                                      │
│   Productos ◄──► Sincronización ◄──► Órdenes        │
│      │                │                  │          │
│      └────────────────┼──────────────────┘          │
│                       ▼                              │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│               MERCADO LIBRE API                      │
│                                                      │
│   Items API    Orders API    Stock API    Users     │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## 1. Autenticación OAuth

### Configuración

```env
# .env.local
ML_APP_ID=tu_app_id
ML_CLIENT_SECRET=tu_client_secret
ML_REDIRECT_URI=https://tuapp.com/api/auth/ml/callback
```

### Flujo OAuth

```typescript
// lib/mercadolibre.ts
const ML_AUTH_URL = 'https://auth.mercadolibre.com.ar/authorization';
const ML_TOKEN_URL = 'https://api.mercadolibre.com/oauth/token';
const ML_API_URL = 'https://api.mercadolibre.com';

export function getAuthUrl(state: string) {
  const params = new URLSearchParams({
    response_type: 'code',
    client_id: process.env.ML_APP_ID!,
    redirect_uri: process.env.ML_REDIRECT_URI!,
    state
  });
  
  return `${ML_AUTH_URL}?${params}`;
}

export async function exchangeCodeForToken(code: string) {
  const response = await fetch(ML_TOKEN_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      grant_type: 'authorization_code',
      client_id: process.env.ML_APP_ID,
      client_secret: process.env.ML_CLIENT_SECRET,
      code,
      redirect_uri: process.env.ML_REDIRECT_URI
    })
  });
  
  return response.json();
}

export async function refreshAccessToken(refreshToken: string) {
  const response = await fetch(ML_TOKEN_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      grant_type: 'refresh_token',
      client_id: process.env.ML_APP_ID,
      client_secret: process.env.ML_CLIENT_SECRET,
      refresh_token: refreshToken
    })
  });
  
  return response.json();
}
```

### Callback de Auth

```typescript
// app/api/auth/ml/callback/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { exchangeCodeForToken } from '@/lib/mercadolibre';
import { createClient } from '@/lib/supabase/server';

export async function GET(req: NextRequest) {
  const code = req.nextUrl.searchParams.get('code');
  const state = req.nextUrl.searchParams.get('state');
  
  if (!code) {
    return NextResponse.redirect('/error?msg=no_code');
  }
  
  try {
    const tokens = await exchangeCodeForToken(code);
    
    const supabase = createClient();
    const { data: { user } } = await supabase.auth.getUser();
    
    // Guardar tokens encriptados
    await supabase
      .from('ml_connections')
      .upsert({
        user_id: user!.id,
        access_token: tokens.access_token,
        refresh_token: tokens.refresh_token,
        ml_user_id: tokens.user_id,
        expires_at: new Date(Date.now() + tokens.expires_in * 1000).toISOString()
      });
    
    return NextResponse.redirect('/dashboard?ml=connected');
    
  } catch (error) {
    console.error('ML auth error:', error);
    return NextResponse.redirect('/error?msg=auth_failed');
  }
}
```

## 2. Sincronización de Productos

### Tabla de Mapeo

```sql
CREATE TABLE ml_products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users NOT NULL,
  local_product_id UUID REFERENCES products(id),
  ml_item_id TEXT UNIQUE,
  ml_status TEXT,
  ml_permalink TEXT,
  last_synced_at TIMESTAMPTZ,
  sync_errors JSONB DEFAULT '[]',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_ml_products_user ON ml_products(user_id);
CREATE INDEX idx_ml_products_item ON ml_products(ml_item_id);
```

### Publicar Producto en ML

```typescript
// lib/ml-products.ts
interface MLItemData {
  title: string;
  category_id: string;
  price: number;
  currency_id: string;
  available_quantity: number;
  buying_mode: string;
  condition: string;
  listing_type_id: string;
  description: { plain_text: string };
  pictures: { source: string }[];
  attributes: { id: string; value_name: string }[];
}

export async function publishToML(
  accessToken: string,
  productData: MLItemData
) {
  const response = await fetch(`${ML_API_URL}/items`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(productData)
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Error publicando en ML');
  }
  
  return response.json();
}

export async function updateMLItem(
  accessToken: string,
  itemId: string,
  updates: Partial<MLItemData>
) {
  const response = await fetch(`${ML_API_URL}/items/${itemId}`, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(updates)
  });
  
  return response.json();
}
```

### Sincronizar desde Tu Sistema a ML

```typescript
// lib/ml-sync.ts
import { createClient } from '@/lib/supabase/server';
import { publishToML, updateMLItem } from './ml-products';
import { getValidToken } from './ml-auth';

export async function syncProductToML(userId: string, productId: string) {
  const supabase = createClient();
  
  // Obtener producto local
  const { data: product } = await supabase
    .from('products')
    .select('*')
    .eq('id', productId)
    .single();
    
  if (!product) throw new Error('Producto no encontrado');
  
  // Obtener token válido
  const token = await getValidToken(userId);
  
  // Verificar si ya existe en ML
  const { data: mlProduct } = await supabase
    .from('ml_products')
    .select('ml_item_id')
    .eq('local_product_id', productId)
    .single();
  
  const mlData = {
    title: product.title.substring(0, 60), // ML limit
    category_id: product.metadata?.ml_category || 'MLA1055', // Default
    price: product.price,
    currency_id: 'ARS',
    available_quantity: product.stock || 1,
    buying_mode: 'buy_it_now',
    condition: 'new',
    listing_type_id: 'gold_special',
    description: { plain_text: product.description || '' },
    pictures: product.images?.map((url: string) => ({ source: url })) || []
  };
  
  let mlItem;
  
  if (mlProduct?.ml_item_id) {
    // Actualizar existente
    mlItem = await updateMLItem(token, mlProduct.ml_item_id, mlData);
  } else {
    // Crear nuevo
    mlItem = await publishToML(token, mlData);
    
    // Guardar mapeo
    await supabase
      .from('ml_products')
      .insert({
        user_id: userId,
        local_product_id: productId,
        ml_item_id: mlItem.id,
        ml_status: mlItem.status,
        ml_permalink: mlItem.permalink
      });
  }
  
  return mlItem;
}
```

## 3. Sincronización de Órdenes

### Tabla de Órdenes ML

```sql
CREATE TABLE ml_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users NOT NULL,
  local_order_id UUID REFERENCES orders(id),
  ml_order_id TEXT UNIQUE,
  ml_status TEXT,
  buyer_nickname TEXT,
  buyer_email TEXT,
  total DECIMAL(10,2),
  shipping_status TEXT,
  ml_data JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Traer Órdenes de ML

```typescript
// lib/ml-orders.ts
export async function fetchMLOrders(accessToken: string, sellerId: string) {
  const response = await fetch(
    `${ML_API_URL}/orders/search?seller=${sellerId}&sort=date_desc`,
    {
      headers: { 'Authorization': `Bearer ${accessToken}` }
    }
  );
  
  return response.json();
}

export async function syncOrdersFromML(userId: string) {
  const supabase = createClient();
  
  const token = await getValidToken(userId);
  const { data: connection } = await supabase
    .from('ml_connections')
    .select('ml_user_id')
    .eq('user_id', userId)
    .single();
    
  const mlOrders = await fetchMLOrders(token, connection.ml_user_id);
  
  for (const order of mlOrders.results) {
    // Verificar si ya existe
    const { data: existing } = await supabase
      .from('ml_orders')
      .select('id')
      .eq('ml_order_id', order.id.toString())
      .single();
      
    if (!existing) {
      // Crear orden local
      const { data: localOrder } = await supabase
        .from('orders')
        .insert({
          customer_id: userId, // O buscar/crear cliente
          total: order.total_amount,
          status: mapMLStatus(order.status),
          metadata: { source: 'mercadolibre', ml_order_id: order.id }
        })
        .select()
        .single();
      
      // Guardar mapeo
      await supabase
        .from('ml_orders')
        .insert({
          user_id: userId,
          local_order_id: localOrder.id,
          ml_order_id: order.id.toString(),
          ml_status: order.status,
          buyer_nickname: order.buyer.nickname,
          total: order.total_amount,
          shipping_status: order.shipping?.status,
          ml_data: order
        });
    }
  }
}

function mapMLStatus(mlStatus: string): string {
  const statusMap: Record<string, string> = {
    'paid': 'paid',
    'confirmed': 'paid',
    'payment_required': 'pending',
    'cancelled': 'cancelled'
  };
  return statusMap[mlStatus] || 'pending';
}
```

## 4. Sincronización de Stock

```typescript
// lib/ml-stock.ts
export async function updateMLStock(
  accessToken: string,
  itemId: string,
  quantity: number
) {
  return fetch(`${ML_API_URL}/items/${itemId}`, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      available_quantity: quantity
    })
  });
}

// Webhook cuando cambia stock local
export async function onLocalStockChange(productId: string, newStock: number) {
  const supabase = createClient();
  
  // Buscar mapeo ML
  const { data: mlProduct } = await supabase
    .from('ml_products')
    .select('ml_item_id, user_id')
    .eq('local_product_id', productId)
    .single();
    
  if (!mlProduct) return; // No está en ML
  
  const token = await getValidToken(mlProduct.user_id);
  
  await updateMLStock(token, mlProduct.ml_item_id, newStock);
  
  // Actualizar last_synced
  await supabase
    .from('ml_products')
    .update({ last_synced_at: new Date().toISOString() })
    .eq('local_product_id', productId);
}
```

## 5. UI de Administración

```tsx
// components/MLProductManager.tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { RefreshCw, ExternalLink, AlertCircle } from 'lucide-react';

interface MLProduct {
  id: string;
  title: string;
  ml_item_id: string | null;
  ml_status: string | null;
  ml_permalink: string | null;
  last_synced_at: string | null;
}

export function MLProductManager({ products }: { products: MLProduct[] }) {
  const [syncing, setSyncing] = useState<string | null>(null);
  
  async function syncProduct(productId: string) {
    setSyncing(productId);
    try {
      const res = await fetch('/api/ml/sync-product', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ productId })
      });
      
      if (res.ok) {
        // Recargar o actualizar estado
        window.location.reload();
      }
    } finally {
      setSyncing(null);
    }
  }
  
  return (
    <div className="space-y-4">
      <h2 className="text-lg font-semibold">Productos en Mercado Libre</h2>
      
      <div className="divide-y">
        {products.map(product => (
          <div key={product.id} className="py-4 flex items-center justify-between">
            <div>
              <p className="font-medium">{product.title}</p>
              <div className="flex items-center gap-2 mt-1">
                {product.ml_item_id ? (
                  <>
                    <Badge variant={product.ml_status === 'active' ? 'default' : 'secondary'}>
                      {product.ml_status}
                    </Badge>
                    <span className="text-xs text-muted-foreground">
                      Sincronizado: {new Date(product.last_synced_at!).toLocaleDateString()}
                    </span>
                  </>
                ) : (
                  <Badge variant="outline">No publicado</Badge>
                )}
              </div>
            </div>
            
            <div className="flex gap-2">
              {product.ml_permalink && (
                <Button variant="ghost" size="sm" asChild>
                  <a href={product.ml_permalink} target="_blank" rel="noopener">
                    <ExternalLink className="h-4 w-4" />
                  </a>
                </Button>
              )}
              
              <Button
                variant="outline"
                size="sm"
                onClick={() => syncProduct(product.id)}
                disabled={syncing === product.id}
              >
                <RefreshCw className={`h-4 w-4 mr-1 ${syncing === product.id ? 'animate-spin' : ''}`} />
                {product.ml_item_id ? 'Sincronizar' : 'Publicar'}
              </Button>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 6. Manejo de Errores

```typescript
// lib/ml-errors.ts
export class MLError extends Error {
  code: string;
  userMessage: string;
  
  constructor(code: string, message: string, userMessage: string) {
    super(message);
    this.code = code;
    this.userMessage = userMessage;
  }
}

export function handleMLError(error: any): MLError {
  const errorMap: Record<string, string> = {
    'invalid_token': 'Tu sesión de ML expiró. Reconectá tu cuenta.',
    'item.status.not_allowed': 'No podés modificar este producto ahora.',
    'item.category_id.invalid': 'La categoría seleccionada no es válida.',
    'item.available_quantity.invalid': 'El stock debe ser mayor a 0.',
    'forbidden': 'No tenés permisos para esta acción.'
  };
  
  const code = error.cause?.[0]?.code || error.error || 'unknown';
  const userMessage = errorMap[code] || 'Error de Mercado Libre. Intentá de nuevo.';
  
  return new MLError(code, error.message, userMessage);
}
```

## Checklist de Implementación

### Auth
- [ ] App registrada en ML Developers
- [ ] OAuth flow completo
- [ ] Refresh token automático
- [ ] Tokens almacenados seguros

### Productos
- [ ] Publicar productos
- [ ] Actualizar productos
- [ ] Mapeo categorías
- [ ] Sincronización bidireccional

### Órdenes
- [ ] Fetch de órdenes
- [ ] Creación de orden local
- [ ] Estados mapeados
- [ ] Notificaciones

### Stock
- [ ] Sync stock a ML
- [ ] Webhook stock local
- [ ] Prevención overselling

## Presentar Resultados al Usuario

```
✓ Mercado Libre integrado

**Conectado:**
- Cuenta ML vinculada
- Tokens almacenados

**Sincronización:**
- Productos publicables a ML
- Órdenes importables
- Stock sincronizado

**Panel de control:**
- Ver productos en ML
- Publicar nuevos
- Ver estado de sync

**Próximos pasos:**
1. Publicar primer producto
2. Configurar categorías
3. Activar sync automático

¿Querés publicar tus productos ahora?
```

## Solución de Problemas

| Problema | Causa | Solución |
|----------|-------|----------|
| Token inválido | Expiró o revocado | Re-autenticar OAuth |
| Categoría inválida | ID incorrecto | Usar Category Predictor API |
| Producto no publica | Falta atributo requerido | Revisar validación de ML |
| Stock no sync | Webhook falla | Revisar logs de sync |

---

## Integración con Supabase (Opcional)

> Esta sección aplica cuando usás Supabase como backend. Si usás VPS u otro backend, ignorá esta sección.

### Esquema SQL Requerido

```sql
-- Tabla de conexiones ML (tokens OAuth)
CREATE TABLE ml_connections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users NOT NULL UNIQUE,
  access_token TEXT NOT NULL,
  refresh_token TEXT NOT NULL,
  ml_user_id TEXT,
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Mapeo de productos locales a ML
CREATE TABLE ml_products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users NOT NULL,
  local_product_id UUID REFERENCES products(id) ON DELETE CASCADE,
  ml_item_id TEXT UNIQUE,
  ml_status TEXT,
  ml_permalink TEXT,
  last_synced_at TIMESTAMPTZ,
  sync_errors JSONB DEFAULT '[]',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Órdenes de ML
CREATE TABLE ml_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users NOT NULL,
  local_order_id UUID REFERENCES orders(id),
  ml_order_id TEXT UNIQUE,
  ml_status TEXT,
  buyer_nickname TEXT,
  buyer_email TEXT,
  total DECIMAL(10,2),
  shipping_status TEXT,
  ml_data JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- RLS
ALTER TABLE ml_connections ENABLE ROW LEVEL SECURITY;
ALTER TABLE ml_products ENABLE ROW LEVEL SECURITY;
ALTER TABLE ml_orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users manage own ML connection"
  ON ml_connections FOR ALL USING (user_id = auth.uid());

CREATE POLICY "Users manage own ML products"
  ON ml_products FOR ALL USING (user_id = auth.uid());

CREATE POLICY "Users view own ML orders"
  ON ml_orders FOR SELECT USING (user_id = auth.uid());

-- Índices
CREATE INDEX idx_ml_connections_user ON ml_connections(user_id);
CREATE INDEX idx_ml_products_local ON ml_products(local_product_id);
CREATE INDEX idx_ml_orders_local ON ml_orders(local_order_id);
```

### Helper: Token Válido con Auto-Refresh

```typescript
// lib/ml-auth.ts
import { createClient } from '@/lib/supabase/server';
import { refreshAccessToken } from './mercadolibre';

export async function getValidToken(userId: string): Promise<string> {
  const supabase = createClient();
  
  const { data: conn } = await supabase
    .from('ml_connections')
    .select('*')
    .eq('user_id', userId)
    .single();
  
  if (!conn) throw new Error('Cuenta ML no conectada');
  
  // Si expira en menos de 5 min, refrescar
  const expiresAt = new Date(conn.expires_at);
  const now = new Date();
  const fiveMin = 5 * 60 * 1000;
  
  if (expiresAt.getTime() - now.getTime() < fiveMin) {
    const tokens = await refreshAccessToken(conn.refresh_token);
    
    await supabase
      .from('ml_connections')
      .update({
        access_token: tokens.access_token,
        refresh_token: tokens.refresh_token,
        expires_at: new Date(Date.now() + tokens.expires_in * 1000).toISOString(),
        updated_at: new Date().toISOString()
      })
      .eq('user_id', userId);
    
    return tokens.access_token;
  }
  
  return conn.access_token;
}
```

### Edge Function: Sync Stock Automático

```typescript
// supabase/functions/ml-stock-sync/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )
  
  const { productId, newStock } = await req.json()
  
  // Buscar mapeo ML
  const { data: mlProduct } = await supabase
    .from('ml_products')
    .select('ml_item_id, user_id')
    .eq('local_product_id', productId)
    .single()
  
  if (!mlProduct) {
    return new Response(JSON.stringify({ skipped: true, reason: 'not_in_ml' }))
  }
  
  // Obtener token
  const { data: conn } = await supabase
    .from('ml_connections')
    .select('access_token')
    .eq('user_id', mlProduct.user_id)
    .single()
  
  // Actualizar en ML
  const res = await fetch(
    `https://api.mercadolibre.com/items/${mlProduct.ml_item_id}`,
    {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${conn.access_token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ available_quantity: newStock })
    }
  )
  
  // Actualizar timestamp de sync
  await supabase
    .from('ml_products')
    .update({ last_synced_at: new Date().toISOString() })
    .eq('local_product_id', productId)
  
  return new Response(JSON.stringify({ success: res.ok }))
})
```

### Trigger: Auto-Sync en Cambio de Stock

```sql
-- Función que llama a la edge function cuando cambia stock
CREATE OR REPLACE FUNCTION notify_stock_change()
RETURNS TRIGGER AS $$
BEGIN
  IF OLD.stock IS DISTINCT FROM NEW.stock THEN
    PERFORM net.http_post(
      url := current_setting('app.supabase_url') || '/functions/v1/ml-stock-sync',
      headers := jsonb_build_object(
        'Authorization', 'Bearer ' || current_setting('app.service_role_key'),
        'Content-Type', 'application/json'
      ),
      body := jsonb_build_object('productId', NEW.id, 'newStock', NEW.stock)
    );
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_product_stock_change
  AFTER UPDATE ON products
  FOR EACH ROW EXECUTE FUNCTION notify_stock_change();
```

### Activar Integración Completa

Usá este comando maestro:
```
@skills/supabase-mcp/SKILL.md @skills/mercadolibre-mcp/SKILL.md
Configurá backend Supabase + sync Mercado Libre completo
```

### Checklist Supabase + ML

- [ ] Tablas `ml_connections`, `ml_products`, `ml_orders` creadas
- [ ] RLS activada en las 3 tablas
- [ ] Variables ML en Supabase Dashboard
- [ ] OAuth callback apunta a tu dominio + guarda en `ml_connections`
- [ ] Edge function `ml-stock-sync` deployada
- [ ] Trigger de stock activo (requiere extensión `pg_net`)
