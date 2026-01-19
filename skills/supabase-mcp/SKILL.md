---
name: supabase-mcp
description: Construí el backend con Supabase MCP. Usá esta skill cuando el usuario diga "configurá Supabase", "hacé el backend", "necesito auth", "base de datos", "RLS", "storage", "edge functions", o "backend moderno".
---

# Supabase MCP Backend

Construí backends modernos, simples y potentes usando el MCP de Supabase.

## Frases que Activan esta Skill

- "Configurá Supabase"
- "Hacé el backend"
- "Necesito autenticación"
- "Creá la base de datos"
- "Configurá RLS"
- "Necesito storage"
- "Edge functions"
- "Backend moderno y rápido"

## Arquitectura Recomendada

```
┌─────────────────────────────────────────────────────┐
│                    FRONTEND                          │
│              (Next.js / React Native)                │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│                 SUPABASE MCP                         │
├─────────────────────────────────────────────────────┤
│  Auth          │  Database      │  Storage          │
│  - Email/Pass  │  - PostgreSQL  │  - Archivos       │
│  - OAuth       │  - RLS         │  - Imágenes       │
│  - Magic Link  │  - Realtime    │  - CDN            │
├─────────────────────────────────────────────────────┤
│              Edge Functions (cuando aportan)         │
└─────────────────────────────────────────────────────┘
```

## 1. Autenticación Simple

### Para Usuarios No-Técnicos

```sql
-- Tabla de perfiles extendida
CREATE TABLE profiles (
  id UUID REFERENCES auth.users PRIMARY KEY,
  email TEXT,
  full_name TEXT,
  avatar_url TEXT,
  role TEXT DEFAULT 'user',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Trigger para crear perfil automáticamente
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO profiles (id, email, full_name)
  VALUES (
    NEW.id,
    NEW.email,
    COALESCE(NEW.raw_user_meta_data->>'full_name', '')
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION handle_new_user();
```

### Flujo de Auth Mobile-First

```typescript
// Configuración simple para usuarios reales
const supabaseAuthConfig = {
  providers: ['email', 'google'], // Solo lo necesario
  redirectTo: `${window.location.origin}/auth/callback`,
  emailConfirmation: true,
  passwordMinLength: 8
};

// Login simple
async function signIn(email: string, password: string) {
  const { data, error } = await supabase.auth.signInWithPassword({
    email,
    password
  });
  
  if (error) {
    // Mensajes claros para usuarios
    if (error.message.includes('Invalid')) {
      return { error: 'Email o contraseña incorrectos' };
    }
    return { error: 'Error al iniciar sesión. Intentá de nuevo.' };
  }
  
  return { user: data.user };
}
```

## 2. Base de Datos Bien Modelada

### Estructura Base para Productos

```sql
-- Usuarios y roles
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  role TEXT DEFAULT 'customer' CHECK (role IN ('admin', 'vendor', 'customer')),
  phone TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Productos/Servicios
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  vendor_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  price DECIMAL(10,2) NOT NULL,
  currency TEXT DEFAULT 'ARS',
  status TEXT DEFAULT 'active' CHECK (status IN ('active', 'inactive', 'sold')),
  images TEXT[],
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Órdenes/Transacciones
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_id UUID REFERENCES profiles(id),
  product_id UUID REFERENCES products(id),
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled')),
  payment_status TEXT DEFAULT 'pending',
  payment_id TEXT, -- ID de Mercado Pago
  total DECIMAL(10,2) NOT NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Índices para performance
CREATE INDEX idx_products_vendor ON products(vendor_id);
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_status ON orders(status);
```

## 3. RLS Segura Sin Fricción

```sql
-- Habilitar RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- PROFILES: Ver propio, admin ve todos
CREATE POLICY "Users view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);

-- PRODUCTS: Públicos para ver, owner para editar
CREATE POLICY "Anyone can view active products"
  ON products FOR SELECT
  USING (status = 'active');

CREATE POLICY "Vendors manage own products"
  ON products FOR ALL
  USING (vendor_id = auth.uid());

-- ORDERS: Solo partes involucradas
CREATE POLICY "Customers view own orders"
  ON orders FOR SELECT
  USING (customer_id = auth.uid());

CREATE POLICY "Vendors view orders of their products"
  ON orders FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM products
      WHERE products.id = orders.product_id
      AND products.vendor_id = auth.uid()
    )
  );
```

## 4. Storage Optimizado

```sql
-- Bucket para imágenes de productos
INSERT INTO storage.buckets (id, name, public)
VALUES ('product-images', 'product-images', true);

-- Políticas de storage
CREATE POLICY "Anyone can view product images"
  ON storage.objects FOR SELECT
  USING (bucket_id = 'product-images');

CREATE POLICY "Authenticated users upload images"
  ON storage.objects FOR INSERT
  WITH CHECK (
    bucket_id = 'product-images'
    AND auth.role() = 'authenticated'
  );

CREATE POLICY "Users delete own images"
  ON storage.objects FOR DELETE
  USING (
    bucket_id = 'product-images'
    AND auth.uid()::text = (storage.foldername(name))[1]
  );
```

### Upload Optimizado para Mobile

```typescript
async function uploadImage(file: File, userId: string) {
  // Comprimir para mobile
  const compressed = await compressImage(file, {
    maxWidth: 1200,
    maxHeight: 1200,
    quality: 0.8
  });
  
  const fileName = `${userId}/${Date.now()}-${file.name}`;
  
  const { data, error } = await supabase.storage
    .from('product-images')
    .upload(fileName, compressed, {
      cacheControl: '3600',
      upsert: false
    });
    
  if (error) throw error;
  
  // Retornar URL pública
  const { data: { publicUrl } } = supabase.storage
    .from('product-images')
    .getPublicUrl(data.path);
    
  return publicUrl;
}
```

## 5. Edge Functions (Solo Cuando Aportan)

### Webhook de Mercado Pago

```typescript
// supabase/functions/mercadopago-webhook/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  try {
    const body = await req.json()
    
    // Verificar firma de MP (seguridad)
    const signature = req.headers.get('x-signature')
    if (!verifySignature(body, signature)) {
      return new Response('Invalid signature', { status: 401 })
    }
    
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    )
    
    // Actualizar orden según evento
    if (body.type === 'payment') {
      const paymentId = body.data.id
      const status = body.action === 'payment.created' ? 'pending' :
                     body.action === 'payment.approved' ? 'paid' : 'failed'
      
      await supabase
        .from('orders')
        .update({ 
          payment_status: status,
          payment_id: paymentId,
          updated_at: new Date().toISOString()
        })
        .eq('payment_id', paymentId)
    }
    
    return new Response(JSON.stringify({ success: true }), {
      headers: { 'Content-Type': 'application/json' }
    })
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    })
  }
})
```

## Checklist de Implementación

### Auth
- [ ] Registro con email/password
- [ ] Login social (Google mínimo)
- [ ] Recuperación de contraseña
- [ ] Perfil de usuario
- [ ] Roles (admin/user)

### Database
- [ ] Esquema modelado
- [ ] RLS habilitado
- [ ] Índices creados
- [ ] Triggers para auditoría

### Storage
- [ ] Buckets configurados
- [ ] Políticas de acceso
- [ ] Compresión de imágenes
- [ ] CDN habilitado

### Edge Functions
- [ ] Solo si aportan valor
- [ ] Webhooks de pagos
- [ ] Integraciones externas

## Presentar Resultados al Usuario

```
✓ Backend Supabase configurado

**Auth:**
- Login con email y Google
- Perfiles automáticos
- Recuperación de contraseña

**Base de datos:**
- 3 tablas principales (profiles, products, orders)
- RLS configurada
- Índices optimizados

**Storage:**
- Bucket para imágenes
- Upload optimizado para mobile
- CDN activo

**Próximos pasos:**
1. Conectar frontend
2. Configurar Mercado Pago
3. Testear flujos

¿Querés que configure algo más?
```

## Solución de Problemas

| Problema | Causa | Solución |
|----------|-------|----------|
| "Row level security" error | RLS bloquea query | Verificar políticas con `auth.uid()` |
| Upload falla | Bucket privado | Crear política de INSERT |
| Auth no redirige | Callback mal configurado | Verificar URL en dashboard |
| Edge function timeout | Proceso largo | Usar background jobs |
