---
name: instalar-consultor
description: Instala el agente consultor de estandares en el proyecto actual: coloca el agente, la skill de sincronizacion por departamento, el steering de uso, el hook de confirmacion y configura el MCP de solo lectura hacia el repo de estandares.
version: 1.0.0
---

# Skill: Instalar Consultor de Estándares

Objetivo: dejar el proyecto listo para **consultar** y **materializar por
departamento** los estándares de la empresa.

## Pasos

1. **Pide la ruta del repo de estándares.** Dónde está clonado
   `estandares-empresa` en local (o su URL de Git si se usará git-MCP). Se usará
   como `${ESTANDARES_PATH}`.
2. **Configura el MCP (solo lectura).** Fusiona el bloque `mcpServers.estandares`
   de `mcp/mcp.json` en `.kiro/settings/mcp.json` del proyecto, sustituyendo
   `${ESTANDARES_PATH}` por la ruta dada. Si el archivo existe, **fusiona sin
   borrar** otros servidores. El `autoApprove` debe quedar limitado a lectura
   (`read_file`, `list_directory`, `search_files`).
3. **Instala el agente.** Copia `agents/consultor-estandares.md` a `.kiro/agents/`.
4. **Instala la skill de sincronización.** Copia
   `skills/sincronizar-departamento/` a `.kiro/skills/`.
5. **Instala el steering de uso.** Copia `steering/consultor-uso.md` a
   `.kiro/steering/`.
6. **Instala el hook de confirmación.** Copia
   `hooks/confirmar-sobrescritura.json` a `.kiro/hooks/`.
7. **Verifica.** Pide al agente consultor que lea `catalog.json` vía MCP y liste
   los departamentos disponibles. Si responde con el catálogo, la instalación fue
   correcta.

## Resultado esperado

El proyecto queda con: agente consultor, skill de sincronización por departamento,
steering de uso, hook de confirmación y MCP de solo lectura conectado al repo de
estándares.

## Notas

- La instalación **no** baja estándares todavía; solo deja el consultor operativo.
  La bajada se hace luego pidiendo un departamento (skill `sincronizar-departamento`).
- No se escribe nada en el repo de estándares durante la instalación.
