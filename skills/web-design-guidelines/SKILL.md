---
name: web-design-guidelines
description: Revisá código de UI para verificar cumplimiento con buenas prácticas de interfaces web. Usá esta skill cuando te pidan "revisá mi UI", "mirá la interfaz", "auditá el diseño", "chequeá la accesibilidad", "revisá la experiencia de usuario", "mejorá la estética", "hacé el diseño más refinado", "que se vea más premium", "optimizá el UX", o "revisá contra buenas prácticas".
metadata:
  author: vercel
  version: "1.0.0"
  argument-hint: <archivo-o-patron>
---

# Guías de Interfaces Web

Revisá archivos para verificar cumplimiento con las Guías de Interfaces Web.

## Frases que Activan esta Skill

Esta skill se activa con frases como:
- "Revisá mi UI" / "Mirá la interfaz"
- "Auditá el diseño" / "Chequeá el diseño"
- "Revisá la experiencia de usuario" / "Optimizá el UX"
- "Mejorá la estética" / "Hacé el diseño más refinado"
- "Que se vea más premium" / "Que se vea más profesional"
- "Chequeá la accesibilidad" / "Revisá accesibilidad"
- "Revisá contra buenas prácticas"
- "Analizá la UI/UX"
- "Hacé el diseño más prolijo"
- "Mejorá el look and feel"

## Cómo Funciona

1. Obtené las últimas guías desde la URL fuente de abajo
2. Leé los archivos especificados (o pedile al usuario archivos/patrón)
3. Verificá contra todas las reglas de las guías obtenidas
4. Mostrá hallazgos en formato conciso `archivo:línea`

## Fuente de las Guías

Obtené las guías actualizadas antes de cada revisión:

```
https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
```

Usá WebFetch para obtener las últimas reglas. El contenido obtenido contiene todas las reglas e instrucciones de formato de salida.

## Uso

Cuando un usuario provee un archivo o patrón como argumento:
1. Obtené las guías desde la URL fuente de arriba
2. Leé los archivos especificados
3. Aplicá todas las reglas de las guías obtenidas
4. Mostrá hallazgos usando el formato especificado en las guías

Si no se especifican archivos, preguntale al usuario cuáles archivos revisar.
