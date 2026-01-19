---
name: mercadopago-mcp
description: Integrá pagos con Mercado Pago MCP. Usá esta skill cuando el usuario diga "integrá pagos", "Mercado Pago", "checkout", "cobrar", "suscripciones", "webhooks de pago", o "pagos LATAM".
---

# Mercado Pago MCP

Integrá pagos sin fricción para LATAM usando el MCP de Mercado Pago.

## Frases que Activan esta Skill

- "Integrá Mercado Pago"
- "Necesito cobrar"
- "Hacé el checkout"
- "Configurá pagos"
- "Suscripciones mensuales"
- "Webhooks de pago"
- "Pagos para Argentina/LATAM"

## Arquitectura de Pagos

```
┌─────────────────────────────────────────────────────┐
│                    USUARIO                           │
│              (Mobile / Web App)                      │
└─────────────────────┬───────────────────────────────┘
                      │ 1. Inicia pago
                      ▼
┌─────────────────────────────────────────────────────┐
│                   TU BACKEND                         │
│            (Supabase Edge Function)                  │
└─────────────────────┬───────────────────────────────┘
                      │ 2. Crea preferencia
                      ▼
┌─────────────────────────────────────────────────────┐
│               MERCADO PAGO API                       │
│                                                      │
│   Preference ──► Checkout ──► Payment ──► Webhook   │
│                                                      │
└─────────────────────┬───────────────────────────────┘
                      │ 3. Notifica resultado
                      ▼
┌─────────────────────────────────────────────────────┐
│                   TU BACKEND                         │
│          (Actualiza orden en Supabase)               │
└─────────────────────────────────────────────────────┘
```

## 1. Configuración Inicial

### Variables de Entorno

```env
# .env.local
MERCADOPAGO_ACCESS_TOKEN=APP_USR-xxx
MERCADOPAGO_PUBLIC_KEY=APP_USR-xxx
MERCADOPAGO_WEBHOOK_SECRET=xxx

# URLs
NEXT_PUBLIC_APP_URL=https://tuapp.com
MERCADOPAGO_SUCCESS_URL=https://tuapp.com/pago/exito
MERCADOPAGO_FAILURE_URL=https://tuapp.com/pago/error
MERCADOPAGO_PENDING_URL=https://tuapp.com/pago/pendiente
```

### Instalación

```bash
npm install mercadopago
```

## 2. Crear Preferencia de Pago

```typescript
// lib/mercadopago.ts
import { MercadoPagoConfig, Preference } from 'mercadopago';

const client = new MercadoPagoConfig({ 
  accessToken: process.env.MERCADOPAGO_ACCESS_TOKEN!
});

interface CreatePaymentParams {
  orderId: string;
  title: string;
  price: number;
  quantity: number;
  buyerEmail: string;
  buyerName: string;
}

export async function createPaymentPreference(params: CreatePaymentParams) {
  const preference = new Preference(client);
  
  const response = await preference.create({
    body: {
      items: [{
        id: params.orderId,
        title: params.title,
        quantity: params.quantity,
        unit_price: params.price,
        currency_id: 'ARS'
      }],
      payer: {
        email: params.buyerEmail,
        name: params.buyerName
      },
      back_urls: {
        success: `${process.env.NEXT_PUBLIC_APP_URL}/pago/exito?order=${params.orderId}`,
        failure: `${process.env.NEXT_PUBLIC_APP_URL}/pago/error?order=${params.orderId}`,
        pending: `${process.env.NEXT_PUBLIC_APP_URL}/pago/pendiente?order=${params.orderId}`
      },
      auto_return: 'approved',
      external_reference: params.orderId,
      notification_url: `${process.env.NEXT_PUBLIC_APP_URL}/api/webhooks/mercadopago`,
      statement_descriptor: 'TU_NEGOCIO',
      expires: true,
      expiration_date_from: new Date().toISOString(),
      expiration_date_to: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString() // 24hs
    }
  });
  
  return {
    preferenceId: response.id,
    initPoint: response.init_point, // URL de checkout
    sandboxInitPoint: response.sandbox_init_point
  };
}
```

## 3. API Endpoint para Crear Pago

```typescript
// app/api/pagos/crear/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createPaymentPreference } from '@/lib/mercadopago';
import { createClient } from '@/lib/supabase/server';

export async function POST(req: NextRequest) {
  try {
    const supabase = createClient();
    const { data: { user } } = await supabase.auth.getUser();
    
    if (!user) {
      return NextResponse.json(
        { error: 'Debés iniciar sesión para pagar' },
        { status: 401 }
      );
    }
    
    const body = await req.json();
    const { productId, quantity = 1 } = body;
    
    // Obtener producto
    const { data: product, error } = await supabase
      .from('products')
      .select('*')
      .eq('id', productId)
      .single();
      
    if (error || !product) {
      return NextResponse.json(
        { error: 'Producto no encontrado' },
        { status: 404 }
      );
    }
    
    // Crear orden en DB
    const { data: order, error: orderError } = await supabase
      .from('orders')
      .insert({
        customer_id: user.id,
        product_id: productId,
        total: product.price * quantity,
        status: 'pending',
        payment_status: 'pending'
      })
      .select()
      .single();
      
    if (orderError) {
      return NextResponse.json(
        { error: 'Error al crear la orden' },
        { status: 500 }
      );
    }
    
    // Crear preferencia en MP
    const preference = await createPaymentPreference({
      orderId: order.id,
      title: product.title,
      price: product.price,
      quantity,
      buyerEmail: user.email!,
      buyerName: user.user_metadata?.full_name || 'Cliente'
    });
    
    // Guardar preference ID
    await supabase
      .from('orders')
      .update({ metadata: { preferenceId: preference.preferenceId } })
      .eq('id', order.id);
    
    return NextResponse.json({
      orderId: order.id,
      checkoutUrl: preference.initPoint
    });
    
  } catch (error) {
    console.error('Error creando pago:', error);
    return NextResponse.json(
      { error: 'Error procesando el pago' },
      { status: 500 }
    );
  }
}
```

## 4. Webhook para Notificaciones

```typescript
// app/api/webhooks/mercadopago/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { MercadoPagoConfig, Payment } from 'mercadopago';
import { createClient } from '@supabase/supabase-js';

const client = new MercadoPagoConfig({ 
  accessToken: process.env.MERCADOPAGO_ACCESS_TOKEN!
});

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    
    // Solo procesar pagos
    if (body.type !== 'payment') {
      return NextResponse.json({ received: true });
    }
    
    const paymentId = body.data.id;
    const payment = new Payment(client);
    const paymentData = await payment.get({ id: paymentId });
    
    // Mapear estados de MP a nuestros estados
    const statusMap: Record<string, string> = {
      'approved': 'paid',
      'pending': 'pending',
      'in_process': 'pending',
      'rejected': 'failed',
      'cancelled': 'cancelled',
      'refunded': 'refunded'
    };
    
    const orderId = paymentData.external_reference;
    const newStatus = statusMap[paymentData.status || 'pending'] || 'pending';
    
    // Actualizar orden
    const { error } = await supabase
      .from('orders')
      .update({
        payment_status: newStatus,
        payment_id: paymentId.toString(),
        status: newStatus === 'paid' ? 'paid' : 'pending',
        metadata: {
          mp_status: paymentData.status,
          mp_status_detail: paymentData.status_detail,
          mp_payment_method: paymentData.payment_method_id,
          updated_at: new Date().toISOString()
        }
      })
      .eq('id', orderId);
      
    if (error) {
      console.error('Error actualizando orden:', error);
      return NextResponse.json({ error: 'DB error' }, { status: 500 });
    }
    
    // Si pagó, enviar notificación (opcional)
    if (newStatus === 'paid') {
      // await sendPaymentConfirmation(orderId);
    }
    
    return NextResponse.json({ success: true });
    
  } catch (error) {
    console.error('Webhook error:', error);
    return NextResponse.json(
      { error: 'Webhook processing failed' },
      { status: 500 }
    );
  }
}
```

## 5. Componente de Checkout Mobile-First

```tsx
// components/CheckoutButton.tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Loader2, CreditCard } from 'lucide-react';

interface CheckoutButtonProps {
  productId: string;
  price: number;
  disabled?: boolean;
}

export function CheckoutButton({ productId, price, disabled }: CheckoutButtonProps) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  async function handleCheckout() {
    setLoading(true);
    setError(null);
    
    try {
      const res = await fetch('/api/pagos/crear', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ productId, quantity: 1 })
      });
      
      const data = await res.json();
      
      if (!res.ok) {
        setError(data.error || 'Error al procesar');
        return;
      }
      
      // Redirigir a checkout de MP
      window.location.href = data.checkoutUrl;
      
    } catch (err) {
      setError('Error de conexión. Intentá de nuevo.');
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div className="space-y-2">
      <Button
        onClick={handleCheckout}
        disabled={disabled || loading}
        className="w-full h-14 text-lg font-semibold"
        size="lg"
      >
        {loading ? (
          <>
            <Loader2 className="mr-2 h-5 w-5 animate-spin" />
            Procesando...
          </>
        ) : (
          <>
            <CreditCard className="mr-2 h-5 w-5" />
            Pagar ${price.toLocaleString('es-AR')}
          </>
        )}
      </Button>
      
      {error && (
        <p className="text-sm text-red-500 text-center">{error}</p>
      )}
      
      <p className="text-xs text-muted-foreground text-center">
        Pagá seguro con Mercado Pago
      </p>
    </div>
  );
}
```

## 6. Páginas de Resultado

```tsx
// app/pago/exito/page.tsx
import { CheckCircle } from 'lucide-react';
import { Button } from '@/components/ui/button';
import Link from 'next/link';

export default function PagoExitoso({ searchParams }: { searchParams: { order: string } }) {
  return (
    <div className="min-h-screen flex items-center justify-center p-4">
      <div className="text-center space-y-6 max-w-md">
        <div className="mx-auto w-20 h-20 bg-green-100 rounded-full flex items-center justify-center">
          <CheckCircle className="w-12 h-12 text-green-600" />
        </div>
        
        <h1 className="text-2xl font-bold text-gray-900">
          ¡Pago exitoso!
        </h1>
        
        <p className="text-gray-600">
          Tu pago fue procesado correctamente. 
          Te enviamos un email con los detalles.
        </p>
        
        <div className="pt-4 space-y-3">
          <Button asChild className="w-full">
            <Link href={`/ordenes/${searchParams.order}`}>
              Ver mi pedido
            </Link>
          </Button>
          
          <Button variant="outline" asChild className="w-full">
            <Link href="/">
              Volver al inicio
            </Link>
          </Button>
        </div>
      </div>
    </div>
  );
}
```

## 7. Suscripciones (Si Aplica)

```typescript
// lib/mercadopago-subscriptions.ts
import { MercadoPagoConfig, PreApproval } from 'mercadopago';

const client = new MercadoPagoConfig({ 
  accessToken: process.env.MERCADOPAGO_ACCESS_TOKEN!
});

interface CreateSubscriptionParams {
  planId: string;
  userEmail: string;
  amount: number;
  frequency: number; // meses
  description: string;
}

export async function createSubscription(params: CreateSubscriptionParams) {
  const preapproval = new PreApproval(client);
  
  const response = await preapproval.create({
    body: {
      reason: params.description,
      auto_recurring: {
        frequency: params.frequency,
        frequency_type: 'months',
        transaction_amount: params.amount,
        currency_id: 'ARS'
      },
      back_url: `${process.env.NEXT_PUBLIC_APP_URL}/suscripcion/confirmada`,
      payer_email: params.userEmail,
      external_reference: params.planId
    }
  });
  
  return {
    subscriptionId: response.id,
    initPoint: response.init_point
  };
}
```

## Estados de Pago

| Estado MP | Tu Estado | Acción UI |
|-----------|-----------|-----------|
| `approved` | `paid` | Mostrar éxito, habilitar producto |
| `pending` | `pending` | Mostrar "esperando confirmación" |
| `in_process` | `pending` | Mostrar "procesando" |
| `rejected` | `failed` | Mostrar error, permitir reintentar |
| `cancelled` | `cancelled` | Mostrar cancelado |
| `refunded` | `refunded` | Mostrar reembolsado |

## Checklist de Implementación

### Configuración
- [ ] Access token configurado
- [ ] Public key para SDK frontend
- [ ] URLs de retorno configuradas
- [ ] Webhook URL registrada en MP

### Backend
- [ ] Endpoint crear preferencia
- [ ] Webhook funcionando
- [ ] Estados mapeados correctamente
- [ ] Logs de pagos

### Frontend
- [ ] Botón de checkout
- [ ] Páginas de resultado (éxito/error/pendiente)
- [ ] Feedback visual durante proceso
- [ ] Manejo de errores amigable

### Testing
- [ ] Probado con sandbox
- [ ] Tarjetas de prueba testeadas
- [ ] Webhook probado con ngrok
- [ ] Flujo completo validado

## Presentar Resultados al Usuario

```
✓ Mercado Pago integrado

**Configurado:**
- Checkout de pago único
- Webhook para notificaciones
- Estados sincronizados con tu DB

**Flujo:**
1. Usuario hace clic en "Pagar"
2. Redirige a checkout de MP
3. MP notifica resultado via webhook
4. Orden actualizada automáticamente

**URLs configuradas:**
- Éxito: /pago/exito
- Error: /pago/error
- Pendiente: /pago/pendiente

**Próximos pasos:**
1. Registrar webhook en panel de MP
2. Probar con tarjetas de sandbox
3. Activar modo producción

¿Necesitás suscripciones también?
```

## Solución de Problemas

| Problema | Causa | Solución |
|----------|-------|----------|
| Webhook no llega | URL no accesible | Usar ngrok para desarrollo |
| "Invalid token" | Token incorrecto | Verificar access_token |
| Pago pendiente forever | Webhook no procesa | Revisar logs del endpoint |
| Redirect falla | URL mal formada | Verificar back_urls |

---

## Integración con Supabase (Opcional)

> Esta sección aplica cuando usás Supabase como backend. Si usás VPS u otro backend, ignorá esta sección.

### Esquema SQL Requerido

```sql
-- Agregar a tu schema de Supabase (además de profiles y products)
CREATE TABLE IF NOT EXISTS orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_id UUID REFERENCES profiles(id),
  product_id UUID REFERENCES products(id),
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled')),
  payment_status TEXT DEFAULT 'pending',
  payment_id TEXT,
  total DECIMAL(10,2) NOT NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- RLS para orders
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users view own orders"
  ON orders FOR SELECT
  USING (customer_id = auth.uid());

CREATE POLICY "Service role manages orders"
  ON orders FOR ALL
  USING (auth.jwt()->>'role' = 'service_role');

-- Índices
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_payment ON orders(payment_id);
```

### Edge Function para Webhook (Alternativa)

```typescript
// supabase/functions/mercadopago-webhook/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )
  
  const body = await req.json()
  if (body.type !== 'payment') {
    return new Response(JSON.stringify({ received: true }))
  }
  
  // Fetch payment details from MP
  const paymentRes = await fetch(
    `https://api.mercadopago.com/v1/payments/${body.data.id}`,
    { headers: { 'Authorization': `Bearer ${Deno.env.get('MERCADOPAGO_ACCESS_TOKEN')}` } }
  )
  const payment = await paymentRes.json()
  
  const statusMap: Record<string, string> = {
    'approved': 'paid', 'pending': 'pending', 'rejected': 'failed'
  }
  
  await supabase
    .from('orders')
    .update({
      payment_status: statusMap[payment.status] || 'pending',
      payment_id: payment.id.toString(),
      status: payment.status === 'approved' ? 'paid' : 'pending',
      metadata: { mp_status: payment.status, mp_detail: payment.status_detail }
    })
    .eq('id', payment.external_reference)
  
  return new Response(JSON.stringify({ success: true }))
})
```

### Activar Integración Completa

Usá este comando maestro:
```
@skills/supabase-mcp/SKILL.md @skills/mercadopago-mcp/SKILL.md
Configurá backend Supabase + pagos Mercado Pago integrados
```

### Checklist Supabase + MP

- [ ] Tabla `orders` creada con RLS
- [ ] Variables de entorno en Supabase Dashboard (Settings > Edge Functions)
- [ ] Edge function deployada: `supabase functions deploy mercadopago-webhook`
- [ ] Webhook URL registrada en MP: `https://<project>.supabase.co/functions/v1/mercadopago-webhook`
- [ ] Probado flujo completo: crear orden → pagar → webhook actualiza
