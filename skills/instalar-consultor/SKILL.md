---
name: instalar-consultor
description: "Deja el consultor de estandares operativo en el proyecto. Materializa el agente, el hook y la skill de sincronizacion desde references/ hacia .kiro/, y verifica que exista un servidor MCP de GitHub configurado en el equipo (el Power NO trae servidor MCP propio). Ejecutala una vez por proyecto."
---

# Skill: Instalar Consultor de Estándares

Un Power puede empaquetar **skills y steering**, pero **no** agentes
(`.kiro/agents/`) ni hooks (`.kiro/hooks/`). Además, en este diseño el Power
**no trae servidor MCP propio**: el consultor lee el repo de estándares usando el
**servidor MCP de GitHub que el equipo ya tiene configurado**.

Esta skill: (a) materializa el agente y el hook, y (b) verifica el acceso a GitHub.
Ejecútala **una sola vez por proyecto**.

## Qué crea esta skill

Agente (en `.kiro/agents/`):
- `consultor-estandares.md` — responde sobre estándares (leyendo GitHub) y baja
  artefactos por departamento.

Hook (en `.kiro/hooks/`):
- `confirmar-sobrescritura.json` — `PreToolUse`: confirma antes de sobrescribir un
  artefacto que ya exista en `.kiro/`.

Skill (en `.kiro/skills/`):
- `sincronizar-departamento/SKILL.md` — procedimiento de bajada por departamento,
  que el agente invoca como recurso local.

## Procedimiento

### Paso 1 — Materializar agente, hook y skill
Lee cada plantilla de `references/` y escríbela **tal cual** en su destino:

| Plantilla (references/) | Destino |
|-------------------------|---------|
| `agents/consultor-estandares.md` | `.kiro/agents/consultor-estandares.md` |
| `hooks/confirmar-sobrescritura.json` | `.kiro/hooks/confirmar-sobrescritura.json` |
| `skills/sincronizar-departamento/SKILL.md` | `.kiro/skills/sincronizar-departamento/SKILL.md` |

> La skill `sincronizar-departamento` se materializa en `.kiro/skills/` del
> proyecto (no se deja solo en el Power) para que el agente `consultor-estandares`
> la vea como recurso local y pueda invocarla sin que el usuario tenga que saber
> que existe un Power.

Agrupa la explicación y avisa antes de escribir (las escrituras piden confirmación).

### Paso 2 — Verificar el MCP de GitHub del equipo
Confirma que el entorno tiene un **servidor MCP de GitHub** que exponga
`get_file_contents` y pueda leer `Jesus1Salas/estandares-empresa`.

- **Si ya existe** (el equipo usa GitHub MCP para su trabajo diario): no hay nada
  que configurar. El consultor lo reutiliza.
- **Si NO existe**, guía al usuario a añadirlo a `.kiro/settings/mcp.json` **de su
  equipo** (config local, no del Power). Plantilla sugerida:

  ```json
  {
    "mcpServers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_PERSONAL_ACCESS_TOKEN": "PON_AQUI_TU_TOKEN"
        },
        "disabled": false,
        "autoApprove": ["get_file_contents", "search_code", "search_repositories"]
      }
    }
  }
  ```

  Notas importantes para el usuario:
  - El **token es personal y local**; no viaja en el Power ni en ningún repo.
  - El token necesita **lectura** del repo de estándares (scope `repo` en un PAT
    clásico, o fine-grained con `Contents: Read`). Si el repo es privado, el token
    debe pertenecer al dueño o a un colaborador.
  - **Aviso de seguridad:** esta versión de Kiro puede no expandir variables de
    entorno en `mcp.json`; por eso la plantilla lleva el token en el propio archivo.
    Mantén ese `mcp.json` fuera de control de versiones y **rota el token** si se
    expone. Cuando la expansión de variables funcione en tu entorno, usa
    `${GITHUB_PERSONAL_ACCESS_TOKEN}` en su lugar.
  - Tras editar el `mcp.json`, **reinicia Kiro** para que cargue el servidor.

### Paso 3 — Verificar la conexión
Pide al agente `consultor-estandares` que lea `catalog.json` del repo vía el MCP de
GitHub y liste los departamentos. Si responde con el catálogo, la instalación fue
correcta.

### Paso 4 — Cierre
Recuerda que **el agente y el hook se activan en la siguiente sesión de Kiro**, y
que para bajar estándares hay que pedir un departamento (skill
`sincronizar-departamento`).

## Notas

- No sobrescribas cambios locales sin avisar.
- La instalación **no** baja estándares todavía; solo deja el consultor operativo.
- Nunca se escribe en el repo de estándares (solo lectura vía GitHub MCP).
