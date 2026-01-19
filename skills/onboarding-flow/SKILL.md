---
name: onboarding-flow
description: Diseñá onboarding automático para clientes. Usá esta skill cuando el usuario diga "onboarding", "guiar al usuario", "primer uso", "tour guiado", "sin soporte humano", o "experiencia de bienvenida".
---

# Onboarding Flow Automático

Diseñá flujos de onboarding guiados para que el cliente no necesite soporte humano para empezar.

## Frases que Activan esta Skill

- "Creá el onboarding"
- "Guiar al usuario nuevo"
- "Tour del primer uso"
- "Sin soporte humano"
- "Experiencia de bienvenida"
- "Activación de usuarios"

## Arquitectura del Onboarding

```
┌─────────────────────────────────────────────────────┐
│                 NUEVO USUARIO                        │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              1. BIENVENIDA                           │
│         ¿Qué querés lograr?                         │
│         [Vender] [Comprar] [Explorar]               │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              2. CONFIGURACIÓN                        │
│         Según objetivo seleccionado                 │
│         (Perfil, preferencias, integraciones)       │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              3. PRIMERA ACCIÓN                       │
│         Guiado paso a paso                          │
│         (Con tooltips y highlights)                 │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              4. CELEBRACIÓN                          │
│         ¡Listo! + Próximos pasos                   │
│         + Link al manual                            │
└─────────────────────────────────────────────────────┘
```

## 1. Flujo de Bienvenida

```tsx
// components/onboarding/WelcomeStep.tsx
'use client';

import { motion } from 'framer-motion';
import { Button } from '@/components/ui/button';
import { Package, ShoppingCart, Compass } from 'lucide-react';

interface WelcomeStepProps {
  userName: string;
  onSelect: (goal: 'sell' | 'buy' | 'explore') => void;
}

export function WelcomeStep({ userName, onSelect }: WelcomeStepProps) {
  return (
    <motion.div 
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      className="min-h-screen flex flex-col justify-center p-6"
    >
      <div className="text-center space-y-4 mb-8">
        <h1 className="text-2xl font-bold">
          ¡Hola, {userName}! 👋
        </h1>
        <p className="text-muted-foreground">
          ¿Qué te gustaría hacer primero?
        </p>
      </div>
      
      <div className="space-y-3">
        <GoalCard
          icon={<Package className="h-6 w-6" />}
          title="Quiero vender"
          description="Publicá tus productos y empezá a recibir pagos"
          onClick={() => onSelect('sell')}
        />
        
        <GoalCard
          icon={<ShoppingCart className="h-6 w-6" />}
          title="Quiero comprar"
          description="Explorá productos y hacé tu primera compra"
          onClick={() => onSelect('buy')}
        />
        
        <GoalCard
          icon={<Compass className="h-6 w-6" />}
          title="Solo explorar"
          description="Mirá cómo funciona sin compromiso"
          onClick={() => onSelect('explore')}
        />
      </div>
    </motion.div>
  );
}

function GoalCard({ icon, title, description, onClick }: GoalCardProps) {
  return (
    <button
      onClick={onClick}
      className="w-full p-4 rounded-xl border-2 text-left
                 hover:border-primary hover:bg-primary/5
                 active:scale-[0.98] transition-all"
    >
      <div className="flex items-start gap-4">
        <div className="p-2 rounded-lg bg-primary/10 text-primary">
          {icon}
        </div>
        <div>
          <h3 className="font-semibold">{title}</h3>
          <p className="text-sm text-muted-foreground">{description}</p>
        </div>
      </div>
    </button>
  );
}
```

## 2. Pasos Según Objetivo

```tsx
// lib/onboarding-steps.ts
export const ONBOARDING_FLOWS = {
  sell: [
    {
      id: 'profile',
      title: 'Completá tu perfil',
      description: 'Los compradores confían más en vendedores con perfil completo',
      component: 'ProfileStep',
      required: true
    },
    {
      id: 'first-product',
      title: 'Publicá tu primer producto',
      description: 'Te guiamos paso a paso',
      component: 'FirstProductStep',
      required: true
    },
    {
      id: 'payment-setup',
      title: 'Configurá cómo recibir pagos',
      description: 'Conectá Mercado Pago para cobrar',
      component: 'PaymentSetupStep',
      required: true
    },
    {
      id: 'ml-connect',
      title: 'Conectá Mercado Libre',
      description: 'Opcional: sincronizá tu catálogo',
      component: 'MLConnectStep',
      required: false
    }
  ],
  
  buy: [
    {
      id: 'profile',
      title: 'Completá tu perfil',
      description: 'Para recibir tus compras',
      component: 'ProfileStep',
      required: true
    },
    {
      id: 'browse',
      title: 'Explorá productos',
      description: 'Encontrá lo que buscás',
      component: 'BrowseStep',
      required: false
    }
  ],
  
  explore: [
    {
      id: 'tour',
      title: 'Tour rápido',
      description: 'Te mostramos lo más importante',
      component: 'QuickTourStep',
      required: false
    }
  ]
};
```

## 3. Componente de Progreso

```tsx
// components/onboarding/OnboardingProgress.tsx
'use client';

import { Check } from 'lucide-react';
import { cn } from '@/lib/utils';

interface OnboardingProgressProps {
  steps: { id: string; title: string; completed: boolean }[];
  currentStep: number;
}

export function OnboardingProgress({ steps, currentStep }: OnboardingProgressProps) {
  return (
    <div className="px-6 py-4">
      {/* Barra de progreso */}
      <div className="h-1 bg-gray-200 rounded-full overflow-hidden mb-4">
        <div 
          className="h-full bg-primary transition-all duration-500"
          style={{ width: `${((currentStep + 1) / steps.length) * 100}%` }}
        />
      </div>
      
      {/* Indicadores */}
      <div className="flex justify-between">
        {steps.map((step, index) => (
          <div 
            key={step.id}
            className={cn(
              "flex flex-col items-center",
              index <= currentStep ? "text-primary" : "text-muted-foreground"
            )}
          >
            <div className={cn(
              "w-8 h-8 rounded-full flex items-center justify-center text-sm font-medium",
              step.completed 
                ? "bg-primary text-white" 
                : index === currentStep
                  ? "border-2 border-primary"
                  : "border-2 border-gray-200"
            )}>
              {step.completed ? <Check className="h-4 w-4" /> : index + 1}
            </div>
            <span className="text-xs mt-1 hidden sm:block">{step.title}</span>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 4. Tooltips Guiados

```tsx
// components/onboarding/GuidedTooltip.tsx
'use client';

import { useEffect, useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { X } from 'lucide-react';

interface GuidedTooltipProps {
  targetSelector: string;
  title: string;
  description: string;
  position?: 'top' | 'bottom' | 'left' | 'right';
  onNext: () => void;
  onSkip: () => void;
  step: number;
  totalSteps: number;
}

export function GuidedTooltip({
  targetSelector,
  title,
  description,
  position = 'bottom',
  onNext,
  onSkip,
  step,
  totalSteps
}: GuidedTooltipProps) {
  const [coords, setCoords] = useState({ top: 0, left: 0 });
  
  useEffect(() => {
    const target = document.querySelector(targetSelector);
    if (target) {
      const rect = target.getBoundingClientRect();
      // Highlight del elemento
      target.classList.add('ring-2', 'ring-primary', 'ring-offset-2', 'relative', 'z-50');
      
      setCoords({
        top: rect.bottom + 8,
        left: rect.left + rect.width / 2
      });
      
      return () => {
        target.classList.remove('ring-2', 'ring-primary', 'ring-offset-2', 'relative', 'z-50');
      };
    }
  }, [targetSelector]);
  
  return (
    <>
      {/* Overlay */}
      <div className="fixed inset-0 bg-black/50 z-40" onClick={onSkip} />
      
      {/* Tooltip */}
      <motion.div
        initial={{ opacity: 0, y: 10 }}
        animate={{ opacity: 1, y: 0 }}
        className="fixed z-50 w-72 p-4 bg-white rounded-xl shadow-xl"
        style={{ top: coords.top, left: coords.left, transform: 'translateX(-50%)' }}
      >
        <button 
          onClick={onSkip}
          className="absolute top-2 right-2 text-gray-400 hover:text-gray-600"
        >
          <X className="h-4 w-4" />
        </button>
        
        <h4 className="font-semibold mb-1">{title}</h4>
        <p className="text-sm text-muted-foreground mb-4">{description}</p>
        
        <div className="flex items-center justify-between">
          <span className="text-xs text-muted-foreground">
            {step} de {totalSteps}
          </span>
          <div className="flex gap-2">
            <button 
              onClick={onSkip}
              className="text-sm text-muted-foreground hover:text-gray-900"
            >
              Saltar
            </button>
            <button
              onClick={onNext}
              className="px-3 py-1 bg-primary text-white text-sm rounded-lg"
            >
              {step === totalSteps ? 'Listo' : 'Siguiente'}
            </button>
          </div>
        </div>
      </motion.div>
    </>
  );
}
```

## 5. Celebración Final

```tsx
// components/onboarding/CompletionStep.tsx
'use client';

import { motion } from 'framer-motion';
import { PartyPopper, BookOpen, MessageCircle, ArrowRight } from 'lucide-react';
import { Button } from '@/components/ui/button';
import Link from 'next/link';
import confetti from 'canvas-confetti';
import { useEffect } from 'react';

export function CompletionStep({ userGoal }: { userGoal: string }) {
  useEffect(() => {
    // Confetti al montar
    confetti({
      particleCount: 100,
      spread: 70,
      origin: { y: 0.6 }
    });
  }, []);
  
  return (
    <motion.div
      initial={{ opacity: 0, scale: 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      className="min-h-screen flex flex-col justify-center items-center p-6 text-center"
    >
      <div className="w-20 h-20 bg-primary/10 rounded-full flex items-center justify-center mb-6">
        <PartyPopper className="h-10 w-10 text-primary" />
      </div>
      
      <h1 className="text-2xl font-bold mb-2">
        ¡Todo listo! 🎉
      </h1>
      
      <p className="text-muted-foreground mb-8">
        Ya configuraste todo lo necesario para empezar
      </p>
      
      <div className="w-full space-y-3 mb-8">
        <NextStepCard
          icon={<ArrowRight className="h-5 w-5" />}
          title={userGoal === 'sell' ? 'Ir a mis productos' : 'Explorar productos'}
          href={userGoal === 'sell' ? '/productos' : '/explorar'}
          primary
        />
        
        <NextStepCard
          icon={<BookOpen className="h-5 w-5" />}
          title="Ver el manual completo"
          href="/docs/guia-basica"
        />
        
        <NextStepCard
          icon={<MessageCircle className="h-5 w-5" />}
          title="¿Tenés dudas? Chateá con soporte"
          href="/soporte"
        />
      </div>
      
      <p className="text-xs text-muted-foreground">
        Podés volver a ver este tour en Configuración → Ayuda
      </p>
    </motion.div>
  );
}

function NextStepCard({ icon, title, href, primary }: NextStepCardProps) {
  return (
    <Link
      href={href}
      className={cn(
        "flex items-center gap-3 p-4 rounded-xl transition-all",
        primary 
          ? "bg-primary text-white" 
          : "border hover:border-primary hover:bg-primary/5"
      )}
    >
      {icon}
      <span className="font-medium">{title}</span>
    </Link>
  );
}
```

## 6. Persistencia del Estado

```typescript
// lib/onboarding-state.ts
import { createClient } from '@/lib/supabase/client';

interface OnboardingState {
  goal: 'sell' | 'buy' | 'explore' | null;
  completedSteps: string[];
  currentStep: number;
  completed: boolean;
  skipped: boolean;
}

export async function getOnboardingState(userId: string): Promise<OnboardingState> {
  const supabase = createClient();
  
  const { data } = await supabase
    .from('user_onboarding')
    .select('*')
    .eq('user_id', userId)
    .single();
    
  return data || {
    goal: null,
    completedSteps: [],
    currentStep: 0,
    completed: false,
    skipped: false
  };
}

export async function updateOnboardingState(
  userId: string, 
  updates: Partial<OnboardingState>
) {
  const supabase = createClient();
  
  await supabase
    .from('user_onboarding')
    .upsert({
      user_id: userId,
      ...updates,
      updated_at: new Date().toISOString()
    });
}

export async function completeOnboardingStep(userId: string, stepId: string) {
  const state = await getOnboardingState(userId);
  
  await updateOnboardingState(userId, {
    completedSteps: [...state.completedSteps, stepId],
    currentStep: state.currentStep + 1
  });
}
```

## 7. Adaptación por Nivel

```tsx
// components/onboarding/LevelSelector.tsx
export function LevelSelector({ onSelect }: { onSelect: (level: string) => void }) {
  return (
    <div className="space-y-3">
      <p className="text-center text-muted-foreground mb-4">
        ¿Qué tan familiarizado estás con este tipo de herramientas?
      </p>
      
      <LevelCard
        emoji="🌱"
        title="Soy nuevo en esto"
        description="Explicame todo paso a paso"
        level="basic"
        onSelect={onSelect}
      />
      
      <LevelCard
        emoji="🌿"
        title="Tengo algo de experiencia"
        description="Mostrame lo esencial"
        level="intermediate"
        onSelect={onSelect}
      />
      
      <LevelCard
        emoji="🌳"
        title="Soy experto"
        description="Solo lo técnico, por favor"
        level="advanced"
        onSelect={onSelect}
      />
    </div>
  );
}
```

## Checklist de Implementación

### Flujo
- [ ] Bienvenida con selección de objetivo
- [ ] Pasos según objetivo
- [ ] Progreso visible
- [ ] Celebración final

### UX
- [ ] Skip disponible siempre
- [ ] Volver atrás posible
- [ ] Estado persistido
- [ ] Retomar donde dejó

### Contenido
- [ ] Textos claros y cortos
- [ ] Imágenes/GIFs de apoyo
- [ ] Links al manual
- [ ] Sin jerga técnica

## Presentar Resultados al Usuario

```
✓ Onboarding automático creado

**Flujos configurados:**
- Vendedores: 4 pasos
- Compradores: 2 pasos
- Exploradores: 1 paso (tour)

**Funcionalidades:**
- Selección de objetivo inicial
- Progreso persistido
- Tooltips guiados
- Celebración con confetti 🎉

**Métricas a trackear:**
- % completitud por flujo
- Paso donde abandonan
- Tiempo promedio

El cliente puede empezar sin soporte humano.
```
