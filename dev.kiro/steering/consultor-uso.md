---
inclusion: always
---

# Uso del Consultor de Estándares

Este proyecto tiene instalado el **consultor de estándares** de la empresa, que
lee el repositorio central `Jesus1Salas/estandares-empresa` **directamente en
GitHub**, usando el **servidor MCP de GitHub que el equipo ya tiene configurado**
(solo lectura). El Power no trae servidor MCP propio y no se clona el repo.

## Requisito

- Un **servidor MCP de GitHub** configurado en el equipo (el mismo que se usa para
  el trabajo diario) con acceso de **lectura** al repo de estándares. Si no existe,
  la skill `instalar-consultor` guía cómo añadirlo.

## Consultar

Pregunta directamente por cualquier convención o estándar
("¿cómo nombramos las ramas?", "¿qué dice el estándar de calidad de datos?",
"¿qué hay disponible para QA?"). El consultor responde **citando el archivo
fuente** (`id` y `ruta`). Si algo no está documentado, lo dice; no inventa.

## Bajar (materializar) por departamento

Los estándares se bajan **por departamento**, leyéndolos de GitHub:

- Departamentos exclusivos entre sí: `comercial`, `qa`, `datos`, `infra`, `pmo`.
- **Un solo departamento por proyecto**, registrado en
  `.kiro/estandares.lock.json`.
- `global` (patrones de código, commits, seguridad, arquitectura, etc.) es
  **transversal**: se recomienda bajarlo junto al departamento, pero solo si lo
  pides.

Pide, por ejemplo: *"baja los estándares de QA"* o *"baja datos junto con global"*.

## Cambiar de departamento (override)

Si el proyecto ya adoptó un departamento y pides otro, el consultor **no lo baja**
salvo que confirmes el **override**, que **elimina lo materializado** del
departamento anterior (lo registrado en el lock) antes de bajar el nuevo.

## Reglas

- La fuente de verdad es el repo `estandares-empresa`; aquí **no se edita** ningún
  estándar bajado. Si algo debe cambiar, propón el cambio por **Pull Request** en
  el repo central.
- El consultor **nunca** escribe en el repo de estándares: solo lo lee (vía el MCP
  de GitHub del equipo) y copia hacia `.kiro/` de este proyecto. Aunque el token
  del equipo pueda escribir, el consultor tiene prohibido hacerlo.
