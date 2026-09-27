---
name: instalar-consultor
description: "Instala el consultor de estandares en el proyecto. Copia el agente, el hook y la skill de sincronizacion desde references/ hacia .kiro/. Solo instala; no consulta ni baja estandares. Ejecutala una vez por proyecto."
---

# Skill: Instalar Consultor de Estándares

Objetivo único: **dejar los archivos del consultor en `.kiro/`**. Esta skill
**solo instala**. No hace preguntas sobre uso, no ofrece menús de opciones, no
consulta estándares ni baja departamentos. Todo eso lo hace el agente
`consultor-estandares` después, cuando el usuario se lo pida.

Un Power empaqueta skills y steering, pero **no** agentes ni hooks; por eso estos
viajan como plantillas en `references/` y esta skill los copia al workspace.

## Procedimiento (ejecutar en orden, sin interacción innecesaria)

### Paso 1 — Copiar los 3 archivos
Lee cada plantilla de `references/` y escríbela **tal cual** en su destino:

| Plantilla (references/) | Destino |
|-------------------------|---------|
| `agents/consultor-estandares.md` | `.kiro/agents/consultor-estandares.md` |
| `hooks/confirmar-sobrescritura.json` | `.kiro/hooks/confirmar-sobrescritura.json` |
| `skills/sincronizar-departamento/SKILL.md` | `.kiro/skills/sincronizar-departamento/SKILL.md` |

Escribe los tres archivos directamente. Si alguno ya existe y difiere, avisa y
pregunta antes de sobrescribir; si no existe, créalo sin preguntar.

### Paso 2 — Terminar
Al copiar los tres archivos, la instalación está lista. Informa en **una sola
frase** qué se creó y que el agente `consultor-estandares` se activa en la próxima
sesión de Kiro. **No** ofrezcas opciones, **no** preguntes qué hacer a
continuación, **no** intentes consultar ni bajar estándares.

Ejemplo de mensaje de cierre (adáptalo, pero mantenlo breve y sin preguntas):
> "Instalado: agente `consultor-estandares`, hook de confirmación y skill
> `sincronizar-departamento` en `.kiro/`. Se activan al reiniciar Kiro."

## Reglas para el agente que ejecuta esta skill

- **Esta skill es solo de instalación.** No conversa sobre el uso diario, no lista
  departamentos, no consulta el catálogo ni baja artefactos.
- No pidas al agente `consultor-estandares` que haga nada durante la instalación.
- No escribas fuera de `.kiro/agents/`, `.kiro/hooks/` y `.kiro/skills/`.
- Requisito de fondo (solo mencionarlo si el usuario pregunta, no como paso): el
  consultor usará el servidor MCP de GitHub que el equipo ya tenga configurado para
  leer el repo de estándares. Configurar ese MCP no es parte de esta skill.
