---
name: instalar-consultor
description: Materializa en el workspace las piezas del consultor de estandares que un Power no puede empaquetar (el agente y el hook), copiandolas desde references/. El servidor MCP 'estandares' y el steering de uso ya los provee el Power al instalarse. Ejecutala una vez por proyecto.
---

# Skill: Instalar Consultor de Estándares

Un Power puede empaquetar **skills, steering y servidores MCP**, pero **no**
agentes (`.kiro/agents/`) ni hooks (`.kiro/hooks/`). Esta skill cierra esa brecha:
**materializa** esas piezas en el workspace copiando las plantillas incluidas en
`references/`.

Ejecútala **una sola vez por proyecto** (o cuando quieras restaurar los archivos).

## Lo que YA provee el Power (no lo hace esta skill)

- **Servidor MCP `estandares`** (solo lectura hacia el repo de estándares),
  declarado en el `mcp.json` del Power. Solo debes definir la variable de entorno
  `ESTANDARES_PATH` con la ruta local del repo `estandares-empresa` clonado.
- **Steering de uso** (`consultor-uso.md`), incluido en `dev.kiro/steering/`.
- **Skill de sincronización** `sincronizar-departamento`.

## Qué crea esta skill

Agente (en `.kiro/agents/`):
- `consultor-estandares.md` — responde sobre estándares y baja artefactos por
  departamento (con sus guardrails).

Hook (en `.kiro/hooks/`):
- `confirmar-sobrescritura.json` — `PreToolUse`: confirma antes de sobrescribir un
  artefacto que ya exista en `.kiro/`.

## Procedimiento

1. Lee cada archivo plantilla de la carpeta `references/` de esta skill.
2. Escribe su contenido **tal cual** en la ruta destino del workspace:

   | Plantilla (references/) | Destino en el workspace |
   |-------------------------|-------------------------|
   | `agents/consultor-estandares.md` | `.kiro/agents/consultor-estandares.md` |
   | `hooks/confirmar-sobrescritura.json` | `.kiro/hooks/confirmar-sobrescritura.json` |

3. **Configura la ruta del repo de estándares.** Confirma que existe la variable
   de entorno `ESTANDARES_PATH` apuntando al repo `estandares-empresa` clonado en
   local. Si no está, indícale al usuario cómo definirla (es lo que usa el MCP
   `estandares` para leer el repo). El consultor **no clona** el repo: se clona
   manualmente una vez.
4. Como las escrituras de archivos piden confirmación, agrupa la explicación y
   avisa al usuario de que se van a crear estos archivos antes de escribirlos.
5. **Verifica.** Pide al agente `consultor-estandares` que lea `catalog.json` vía
   MCP y liste los departamentos disponibles. Si responde con el catálogo, la
   instalación fue correcta.
6. Recuerda al usuario que **el agente y el hook se activan en la siguiente
   sesión de Kiro**, y que para bajar estándares debe pedir un departamento (lo
   maneja la skill `sincronizar-departamento`).

## Notas

- No sobrescribas cambios locales sin avisar: si un archivo ya existe y difiere,
  señálalo y pregunta antes de reemplazarlo.
- La instalación **no** baja estándares todavía; solo deja el consultor operativo.
- Nunca se escribe en el repo de estándares durante la instalación (es solo
  lectura).
