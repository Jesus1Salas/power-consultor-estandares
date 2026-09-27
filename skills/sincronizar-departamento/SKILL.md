---
name: sincronizar-departamento
description: Baja (materializa) el paquete de estandares de un departamento del repositorio central al proyecto actual, respetando el candado exclusivo (un solo departamento por proyecto), el override con borrado de lo anterior, y global como transversal opcional. Solo lectura sobre el repo fuente; solo escribe dentro de .kiro/ del consumidor.
version: 1.0.0
---

# Skill: Sincronizar Departamento

Objetivo: copiar al proyecto **todos los artefactos de un departamento** del repo
de estándares (vía MCP de solo lectura), aplicando la regla de **un solo
departamento por proyecto**. `global` es transversal y opcional.

## Taxonomía de departamentos

- **Exclusivos entre sí:** `comercial`, `qa`, `datos`, `infra`, `pmo`.
  Un proyecto solo puede tener **uno** de estos adoptado a la vez.
- **Transversal:** `global`. No consume el candado; puede acompañar a cualquier
  departamento. Se recomienda bajarlo junto al departamento, pero solo si el
  usuario lo solicita.

El departamento de cada artefacto se deriva del campo `ambito` en `catalog.json`.

## Reglas duras (no negociables)

- **Solo lectura sobre el repo de estándares.** Nunca crear, editar ni borrar
  nada en el repo fuente. Solo `read_file` / `list_directory` / `search_files`.
- **Solo escritura dentro de `.kiro/`** del proyecto consumidor.
- **Nunca** bajar un artefacto que no esté en `catalog.json`.

## Estado: el candado (`.kiro/estandares.lock.json`)

Registra el departamento adoptado y los artefactos materializados con su versión:

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
Identifica el **departamento** solicitado (`comercial` | `qa` | `datos` | `infra`
| `pmo`) y si el usuario quiere **también `global`**. Si el usuario no lo
menciona, **recomiéndalo** pero no lo bajes sin su confirmación.

### Paso 1 — Leer el catálogo
Lee `catalog.json` vía MCP. Filtra los artefactos cuyo `ambito` coincida con el
departamento solicitado (y `global` si aplica) y que sean `materializable: true`.

### Paso 2 — Evaluar el candado
Lee `.kiro/estandares.lock.json` si existe:

- **No existe / sin departamento:** procede a materializar (Paso 4).
- **Mismo departamento que el solicitado:** es una actualización/completado;
  procede (Paso 4), respetando confirmación de sobrescritura.
- **Departamento distinto:** **DETENTE**. Informa que el proyecto ya adoptó
  `<departamento actual>` y que los departamentos son exclusivos. Ofrece el
  **override** (Paso 3). No bajes nada hasta que el usuario confirme.

### Paso 3 — Override (solo con confirmación explícita del usuario)
Si el usuario confirma cambiar de departamento:

1. Lee del lock la lista de `artefactos` materializados del departamento anterior.
2. **Elimina cada archivo/carpeta `destino`** de esos artefactos que esté
   registrado en el lock (solo lo registrado; no toques archivos que el usuario
   haya creado por su cuenta). Para skills/agentes que son carpetas, elimina la
   carpeta materializada.
3. Deja `global` si el nuevo departamento también lo lleva; si no, y `global`
   estaba solo por acompañar al anterior, pregunta si conservarlo o quitarlo.
4. Vacía el lock (o reinícialo) antes de registrar el nuevo departamento.
5. Continúa en el Paso 4 con el nuevo departamento.

> Si algún archivo no puede eliminarse, infórmalo y continúa; nunca borres fuera
> de lo registrado en el lock ni fuera de `.kiro/`.

### Paso 4 — Materializar el departamento
Para cada artefacto seleccionado (departamento + global si aplica):

1. Obtén `ruta`, `destino`, `version`, `tipo` y `materializable` del catálogo.
   Si `materializable` es `false`, omítelo (solo consultable).
2. Si el `destino` ya existe en `.kiro/`, **confirma antes de sobrescribir**
   (mostrando versión local vs. nueva). El hook `PreToolUse` refuerza esto.
3. Copia desde `ruta` (repo, vía MCP read) a `destino` (proyecto):
   - `steering` / `hook`: archivo único.
   - `skill` / `agent` que sean carpeta: copia la carpeta completa.
4. **Caso `mcp`:** no sobrescribas `.kiro/settings/mcp.json`; **fusiona** el bloque.

### Paso 5 — Actualizar el candado
Escribe `.kiro/estandares.lock.json` con:
- `departamento`: el adoptado.
- `global`: `true`/`false` según se bajó.
- `artefactos`: mapa `id -> version` de todo lo materializado.

### Paso 6 — Confirmar al usuario
Informa: departamento adoptado, si se incluyó `global`, lista de `id`/`version`
traídos y rutas destino. Recuerda que para cambiar de departamento se requiere
override (que borra lo materializado del anterior).

## Reglas para el agente

- Un solo departamento exclusivo a la vez; `global` es transversal opcional.
- El override solo procede con confirmación explícita y borra únicamente lo
  registrado en el lock.
- Solo lectura del repo fuente; solo escritura en `.kiro/` del consumidor.
- Nunca materializes algo ausente del catálogo.
