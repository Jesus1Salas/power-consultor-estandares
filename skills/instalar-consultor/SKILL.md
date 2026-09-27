---
name: instalar-consultor
description: "Instala el consultor de estandares en el proyecto. Copia el agente y la skill de sincronizacion desde references/ hacia .kiro/. Solo instala; no consulta ni baja estandares. Ejecutala una vez por proyecto."
---

# Skill: Instalar Consultor de Estándares

Objetivo único: **dejar los archivos del consultor en `.kiro/`**. Esta skill
**solo instala**. No hace preguntas sobre uso, no ofrece menús de opciones, no
consulta estándares ni baja departamentos. Todo eso lo hace el agente
`consultor-estandares` después, cuando el usuario se lo pida.

Un Power empaqueta skills y steering, pero **no** agentes; por eso el agente viaja
como plantilla en `references/` y esta skill lo copia al workspace.

## Procedimiento (ejecutar en orden, sin interacción innecesaria)

### Paso 1 — Copiar los archivos
Lee cada plantilla de `references/` y escríbela **tal cual** en su destino:

| Plantilla (references/) | Destino |
|-------------------------|---------|
| `agents/consultor-estandares.md` | `.kiro/agents/consultor-estandares.md` |
| `skills/sincronizar-departamento/SKILL.md` | `.kiro/skills/sincronizar-departamento/SKILL.md` |

Escribe los archivos directamente. Si alguno ya existe y difiere, avisa y pregunta
antes de sobrescribir; si no existe, créalo sin preguntar.

### Paso 2 — Terminar
Al copiar los archivos, la instalación está lista. Informa en **una sola frase**
qué se creó y que el agente `consultor-estandares` se activa en la próxima sesión
de Kiro. **No** ofrezcas opciones, **no** preguntes qué hacer a continuación,
**no** intentes consultar ni bajar estándares.

Ejemplo de mensaje de cierre (adáptalo, pero mantenlo breve y sin preguntas):
> "Instalado: agente `consultor-estandares` y skill `sincronizar-departamento` en
> `.kiro/`. Se activan al reiniciar Kiro."

## Reglas para el agente que ejecuta esta skill

- **Esta skill es solo de instalación.** No conversa sobre el uso diario, no lista
  departamentos, no consulta el catálogo ni baja artefactos.
- No pidas al agente `consultor-estandares` que haga nada durante la instalación.
- No escribas fuera de `.kiro/agents/` y `.kiro/skills/`.
- Requisito de fondo (solo mencionarlo si el usuario pregunta, no como paso): el
  consultor usará el servidor MCP de GitHub que el equipo ya tenga configurado para
  leer el repo de estándares. Configurar ese MCP no es parte de esta skill.
