# CLAUDE.md

Este archivo provee guías para agentes de IA (Claude, Cascade/Windsurf, Cursor, Copilot, etc.) cuando trabajan con código en este repositorio.

## Descripción del Repositorio

Una colección de skills para construir productos digitales completos con MCPs (Model Context Protocols). Las skills son instrucciones empaquetadas que extienden las capacidades del agente.

---

## 🚀 Skills MCP Disponibles

Cuando el usuario pida construir un producto, usá estas skills según corresponda:

### Skill Maestra
| Skill | Activar cuando digan | Archivo |
|-------|---------------------|---------|
| `producto-real-mcp` | "producto completo", "sistema real", "listo para clientes" | `skills/producto-real-mcp/SKILL.md` |

### Backend y Datos
| Skill | Activar cuando digan | Archivo |
|-------|---------------------|---------|
| `supabase-mcp` | "backend", "base de datos", "auth", "Supabase" | `skills/supabase-mcp/SKILL.md` |

### Pagos y Ventas (LATAM)
| Skill | Activar cuando digan | Archivo |
|-------|---------------------|---------|
| `mercadopago-mcp` | "pagos", "checkout", "cobrar", "Mercado Pago" | `skills/mercadopago-mcp/SKILL.md` |
| `mercadolibre-mcp` | "Mercado Libre", "sync ML", "catálogo", "stock" | `skills/mercadolibre-mcp/SKILL.md` |

### Documentación
| Skill | Activar cuando digan | Archivo |
|-------|---------------------|---------|
| `notion-mcp` | "documentación en Notion", "centro de ayuda", "wiki" | `skills/notion-mcp/SKILL.md` |
| `manual-multimedia` | "manual del producto", "tutorial", "guía de uso" | `skills/manual-multimedia/SKILL.md` |

### UX y Onboarding
| Skill | Activar cuando digan | Archivo |
|-------|---------------------|---------|
| `mobile-first-design` | "mobile-first", "diseño mobile", "responsive" | `skills/mobile-first-design/SKILL.md` |
| `onboarding-flow` | "onboarding", "guiar al usuario", "sin soporte" | `skills/onboarding-flow/SKILL.md` |

---

## Cómo Usar las Skills

### En Windsurf/Cascade

**Opción 1:** Referenciar con @
```
@skills/producto-real-mcp/SKILL.md construí un marketplace
```

**Opción 2:** Agregar como contexto
- Clic en 📎 (Add Context) → seleccionar el SKILL.md

**Opción 3:** El agente las lee automáticamente de este archivo cuando detecta el caso de uso.

### Prioridades al construir productos
```
1. CLARIDAD    → El usuario entiende qué hace
2. VELOCIDAD   → Carga rápido, responde rápido  
3. ESTÉTICA    → Se ve profesional y cuidado
4. AUTOMATIZACIÓN → Reduce trabajo manual
```

### Validación final
Después de implementar, siempre validar:
> "Validá todo el flujo como si fueras un cliente que no entiende nada de tecnología"

---

## 🎯 Comandos Maestros (Integraciones Completas)

Copiá y pegá estos prompts para activar múltiples skills de una vez:

### Backend + Pagos (Supabase + Mercado Pago)
```
@skills/supabase-mcp/SKILL.md @skills/mercadopago-mcp/SKILL.md
Configurá Supabase como backend + Mercado Pago para cobros. Incluí: auth, tablas con RLS, webhook de pagos como edge function, y flujo completo de checkout.
```

### Backend + Ventas (Supabase + Mercado Libre)
```
@skills/supabase-mcp/SKILL.md @skills/mercadolibre-mcp/SKILL.md
Configurá Supabase + sync bidireccional con Mercado Libre. Incluí: OAuth, tablas de mapeo (ml_connections, ml_products, ml_orders), y sync automático de stock.
```

### Stack Comercial Completo LATAM
```
@skills/supabase-mcp/SKILL.md @skills/mercadopago-mcp/SKILL.md @skills/mercadolibre-mcp/SKILL.md
Montá un sistema comercial completo: backend Supabase, pagos con MP, y catálogo sincronizado con ML. Stock unificado y órdenes centralizadas.
```

### Producto SaaS Listo para Clientes
```
@skills/producto-real-mcp/SKILL.md @skills/supabase-mcp/SKILL.md @skills/mercadopago-mcp/SKILL.md @skills/onboarding-flow/SKILL.md
Construí un SaaS completo: landing, auth, dashboard, pagos integrados, y onboarding guiado. Mobile-first y sin necesidad de soporte.
```

### Producto + Documentación Completa
```
@skills/producto-real-mcp/SKILL.md @skills/notion-mcp/SKILL.md @skills/manual-multimedia/SKILL.md
Entregá producto funcional + documentación en Notion + manual multimedia en 3 niveles (básico, intermedio, avanzado).
```

### E-commerce LATAM Full Stack
```
@skills/producto-real-mcp/SKILL.md @skills/supabase-mcp/SKILL.md @skills/mercadopago-mcp/SKILL.md @skills/mercadolibre-mcp/SKILL.md @skills/mobile-first-design/SKILL.md
E-commerce completo: catálogo propio + ML, checkout MP, stock unificado, diseño mobile-first. Listo para vender.
```

### Solo Frontend (Sin Supabase, para VPS)
```
@skills/mercadopago-mcp/SKILL.md
Integrá Mercado Pago en mi backend existente (no Supabase). Ignorá las secciones de integración con Supabase.
```

```
@skills/mercadolibre-mcp/SKILL.md
Integrá Mercado Libre en mi backend existente (no Supabase). Ignorá las secciones de integración con Supabase.
```

---

## Crear una Nueva Skill

### Estructura de Directorios

```
skills/
  {skill-name}/           # nombre de directorio en kebab-case
    SKILL.md              # Requerido: definición de la skill
    scripts/              # Requerido: scripts ejecutables
      {script-name}.sh    # Scripts Bash (preferido)
  {skill-name}.zip        # Requerido: empaquetado para distribución
```

### Convenciones de Nombres

- **Directorio de skill**: `kebab-case` (ej., `vercel-deploy`, `log-monitor`)
- **SKILL.md**: Siempre en mayúsculas, siempre este nombre exacto
- **Scripts**: `kebab-case.sh` (ej., `deploy.sh`, `fetch-logs.sh`)
- **Archivo zip**: Debe coincidir exactamente con el nombre del directorio: `{skill-name}.zip`

### Formato de SKILL.md

```markdown
---
name: {skill-name}
description: {Una oración describiendo cuándo usar esta skill. Incluí frases disparadoras como "Desplegá mi app", "Revisá los logs", "Mirá el diseño", etc.}
---

# {Título de la Skill}

{Descripción breve de lo que hace la skill.}

## Cómo Funciona

{Lista numerada explicando el flujo de trabajo de la skill}

## Uso

```bash
bash /mnt/skills/user/{skill-name}/scripts/{script}.sh [args]
```

**Argumentos:**
- `arg1` - Descripción (por defecto: X)

**Ejemplos:**
{Mostrá 2-3 patrones de uso comunes}

## Salida

{Mostrá ejemplo de salida que verán los usuarios}

## Presentar Resultados al Usuario

{Template de cómo Claude debería formatear resultados al presentarlos a usuarios}

## Solución de Problemas

{Problemas comunes y soluciones, especialmente errores de red/permisos}
```

### Buenas Prácticas para Eficiencia de Contexto

Las skills se cargan bajo demanda — solo el nombre y descripción se cargan al inicio. El `SKILL.md` completo se carga en contexto solo cuando el agente decide que la skill es relevante. Para minimizar uso de contexto:

- **Mantené SKILL.md bajo 500 líneas** — poné material de referencia detallado en archivos separados
- **Escribí descripciones específicas** — ayuda al agente a saber exactamente cuándo activar la skill
- **Usá revelación progresiva** — referenciá archivos de soporte que se leen solo cuando se necesitan
- **Preferí scripts sobre código inline** — la ejecución de scripts no consume contexto (solo la salida)
- **Las referencias a archivos funcionan un nivel de profundidad** — linkeá directamente desde SKILL.md a archivos de soporte

### Requerimientos de Scripts

- Usá shebang `#!/bin/bash`
- Usá `set -e` para comportamiento fail-fast
- Escribí mensajes de estado a stderr: `echo "Mensaje" >&2`
- Escribí salida legible por máquina (JSON) a stdout
- Incluí un trap de cleanup para archivos temporales
- Referenciá la ruta del script como `/mnt/skills/user/{skill-name}/scripts/{script}.sh`

### Crear el Paquete Zip

Después de crear o actualizar una skill:

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```

### Instalación para Usuarios Finales

Documentá estos dos métodos de instalación para usuarios:

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
Agregá la skill al conocimiento del proyecto o pegá el contenido de SKILL.md en la conversación.

Si la skill requiere acceso a red, indicá a los usuarios que agreguen los dominios requeridos en `claude.ai/settings/capabilities`.
