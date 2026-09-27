---
name: sincronizar-departamento
description: Baja (materializa) el paquete de estandares de un departamento leyendo el repositorio central DIRECTAMENTE en GitHub (via MCP, solo lectura) y escribiendo los artefactos en .kiro/ del proyecto. Respeta el candado exclusivo (un solo departamento por proyecto), el override con borrado de lo anterior, y global como transversal opcional. Nunca escribe en el repo fuente.
version: 2.0.0
---

# Skill: Sincronizar Departamento

Objetivo: traer al proyecto **todos los artefactos de un departamento** leyéndolos
**directamente del repo en GitHub** (no hay clone local), aplicando la regla de
**un solo departamento por proyecto**. `global` es transversal y opcional.

## Fuente (fija)

- Servidor MCP: `estandares-github` (operaciones de **lectura**).
- Repo: **`Jesus1Salas/estandares-empresa`**, rama `main`.
- Índice: `catalog.json` en la raíz.

## Taxonomía de departamentos

- **Exclusivos entre sí:** `comercial`, `qa`, `datos`, `infra`, `pmo`.
  Un proyecto solo puede tener **uno** adoptado a la vez.
- **Transversal:** `global`. No consume el candado; puede acompañar a cualquier
  departamento. Se recomienda bajarlo junto al departamento, pero solo si el
  usuario lo solicita.

El departamento de cada artefacto se deriva del campo `ambito` en `catalog.json`.

## Reglas duras (no negociables)

- **Solo lectura sobre el repo de estándares.** Usa únicamente `get_file_contents`
  (y `search_code` si hace falta). **Nunca** `create_or_update_file`,
  `push_files`, `create_pull_request`, `delete_file` ni cualquier operación que
  modifique el repo fuente.
- **Solo escritura dentro de `.kiro/`** del proyecto consumidor.
- **Nunca** materializar un artefacto que no esté en `catalog.json`.

## Estado: el candado (`.kiro/estandares.lock.json`)

```json
{
  "departamento": "qa",
  "global": true,
  "artefactos": {
    "steering.qa.estrategia-pruebas": "1.0.0",
    "skill.qa.generador-casos-prueba": "1.0.0"
  }
}
```

## Procedimiento

### Paso 0 — Determinar la petición
Identifica el **departamento** (`comercial` | `qa` | `datos` | `infra` | `pmo`) y
si el usuario quiere **también `global`**. Si no lo menciona, **recomiéndalo**
pero no lo bajes sin confirmación.

### Paso 1 — Leer el catálogo (remoto)
Lee `catalog.json` con `get_file_contents` (owner `Jesus1Salas`, repo
`estandares-empresa`, path `catalog.json`, ref `main`). Filtra los artefactos
cuyo `ambito` coincida con el departamento (y `global` si aplica) y que sean
materializables (`materializable: true` o `"merge"` para los `mcp`).

### Paso 2 — Evaluar el candado
Lee `.kiro/estandares.lock.json` si existe:

- **No existe / sin departamento:** procede (Paso 4).
- **Mismo departamento:** actualización/completado; procede (Paso 4), con
  confirmación de sobrescritura.
- **Departamento distinto:** **DETENTE**. Informa que el proyecto ya adoptó
  `<departamento actual>` y que son exclusivos. Ofrece el **override** (Paso 3).
  No bajes nada hasta confirmación.

### Paso 3 — Override (solo con confirmación explícita)
1. Lee del lock la lista de `artefactos` del departamento anterior.
2. **Elimina cada `destino`** registrado en el lock (solo lo registrado; no toques
   lo que el usuario creó por su cuenta). Carpetas (skills/agentes) completas.
3. Ajusta `global` según el nuevo departamento (pregunta si conservarlo).
4. Reinicia el lock antes de registrar el nuevo departamento.
5. Continúa en el Paso 4.

### Paso 4 — Materializar (leer de GitHub → escribir en .kiro/)
Para cada artefacto seleccionado (departamento + global si aplica):

1. Del catálogo: `ruta`, `destino`, `version`, `tipo`, `materializable`.
2. **Lee el contenido remoto** con `get_file_contents` sobre `ruta`:
   - `steering` / `hook`: un archivo → escribe en `destino`.
   - `skill` / `agent` que son **carpeta**: lista el directorio remoto
     (`get_file_contents` sobre la carpeta) y baja **cada archivo** (incluye
     `SKILL.md`/`.md`, `README.md`, y todo lo de `assets/`), recreando la
     estructura en `destino`.
3. Si el `destino` ya existe en `.kiro/`, **confirma antes de sobrescribir**
   (versión local vs. nueva). El hook `PreToolUse` lo refuerza.
4. **Caso `mcp`** (`materializable: "merge"`): no sobrescribas
   `.kiro/settings/mcp.json`; **fusiona** el bloque `mcpServers.<nombre>` sin
   borrar otros servidores.

### Paso 5 — Actualizar el candado
Escribe `.kiro/estandares.lock.json` con `departamento`, `global` (bool) y el mapa
`artefactos` (`id -> version`) de todo lo materializado.

### Paso 6 — Confirmar al usuario
Informa: departamento adoptado, si se incluyó `global`, lista de `id`/`version` y
rutas destino. Recuerda que cambiar de departamento requiere override (borra lo
del anterior).

## Reglas para el agente

- Un solo departamento exclusivo a la vez; `global` transversal opcional.
- Override solo con confirmación; borra únicamente lo registrado en el lock.
- Lectura del repo fuente solo por GitHub MCP; escritura solo en `.kiro/`.
- Nunca materializar algo ausente del catálogo.
