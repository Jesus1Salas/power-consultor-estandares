---
name: instalar-consultor
description: Materializa en el workspace las piezas del consultor de estandares que un Power no puede empaquetar (el agente y el hook), copiandolas desde references/. El servidor MCP 'estandares-github' (lectura del repo en GitHub) y el steering de uso ya los provee el Power. Ejecutala una vez por proyecto.
---

# Skill: Instalar Consultor de Estándares

Un Power puede empaquetar **skills, steering y servidores MCP**, pero **no**
agentes (`.kiro/agents/`) ni hooks (`.kiro/hooks/`). Esta skill cierra esa brecha:
**materializa** esas piezas copiando las plantillas incluidas en `references/`.

Ejecútala **una sola vez por proyecto** (o para restaurar los archivos).

## Lo que YA provee el Power (no lo hace esta skill)

- **Servidor MCP `estandares-github`**, declarado en el `mcp.json` del Power. Lee
  el repositorio `Jesus1Salas/estandares-empresa` **directamente en GitHub** (no
  se clona). Requiere la variable de entorno `GITHUB_PERSONAL_ACCESS_TOKEN` con un
  token que tenga **acceso de lectura** al repo privado.
- **Steering de uso** (`consultor-uso.md`).
- **Skill de sincronización** `sincronizar-departamento`.

## Qué crea esta skill

Agente (en `.kiro/agents/`):
- `consultor-estandares.md` — responde sobre estándares (leyendo GitHub) y baja
  artefactos por departamento (con sus guardrails).

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

3. **Verifica el token de GitHub.** Confirma que existe
   `GITHUB_PERSONAL_ACCESS_TOKEN` con acceso de lectura al repo
   `Jesus1Salas/estandares-empresa`. Si falta o no tiene acceso, indícalo: sin él,
   el MCP `estandares-github` no podrá leer el repo. **No se clona nada.**
4. Como las escrituras piden confirmación, agrupa la explicación y avisa antes de
   escribir.
5. **Verifica la conexión.** Pide al agente `consultor-estandares` que lea
   `catalog.json` del repo vía MCP (`get_file_contents`) y liste los departamentos.
   Si responde con el catálogo, la instalación fue correcta.
6. Recuerda al usuario que **el agente y el hook se activan en la siguiente sesión
   de Kiro**, y que para bajar estándares debe pedir un departamento (lo maneja la
   skill `sincronizar-departamento`).

## Notas

- No sobrescribas cambios locales sin avisar: si un archivo ya existe y difiere,
  señálalo y pregunta antes de reemplazarlo.
- La instalación **no** baja estándares todavía; solo deja el consultor operativo.
- Nunca se escribe en el repo de estándares (es solo lectura vía GitHub MCP).
