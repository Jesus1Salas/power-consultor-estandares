---
inclusion: always
---

# Uso del Consultor de Estándares

Este proyecto tiene disponible el **consultor de estándares** de la empresa, que
lee el repositorio central `Jesus1Salas/estandares-empresa` **directamente en
GitHub**, usando el **servidor MCP de GitHub que el equipo ya tiene configurado**
(solo lectura). El Power no trae servidor MCP propio y no se clona el repo.

## Primer paso: verificar que el consultor está instalado

El Power aporta este steering y las skills, pero el **agente** `consultor-estandares`
debe existir como archivo en `.kiro/agents/`. Por eso:

- **Al inicio de una interacción relacionada con estándares** (consultar una
  convención, bajar un departamento, o si el usuario menciona "el consultor"),
  **comprueba si existe el archivo `.kiro/agents/consultor-estandares.md`**.
- **Si NO existe:** ofrece instalarlo de inmediato ejecutando la skill
  **`instalar-consultor`** (que crea el agente y la skill `sincronizar-departamento`
  en `.kiro/`). Dilo de forma breve, por ejemplo: *"El consultor de estándares aún
  no está instalado en este proyecto. ¿Lo instalo ahora?"*. Si el usuario acepta,
  ejecuta `instalar-consultor`. Tras instalar, recuerda que el agente se activa en
  la próxima sesión de Kiro.
- **Si ya existe:** procede normalmente (consultar o bajar), preferiblemente con el
  agente `consultor-estandares` seleccionado.

> Esta verificación hace que el usuario no necesite saber que existe una skill de
> instalación: el sistema detecta que falta y la ofrece.

## Consultar

Pregunta directamente por cualquier convención o estándar
("¿cómo nombramos las ramas?", "¿qué dice el estándar de calidad de datos?",
"¿qué hay disponible para QA?"). El consultor responde **citando el archivo
fuente** (`id` y `ruta`). Si algo no está documentado, lo dice; no inventa.

## Bajar (materializar) por departamento

Los estándares se bajan **por departamento**, leyéndolos de GitHub:

- Departamentos exclusivos entre sí: `comercial`, `qa`, `desarrollo`, `infra`, `pmo`.
- **Un solo departamento por proyecto**, registrado en
  `.kiro/estandares.lock.json`.
- `global` (patrones de código, commits, seguridad, arquitectura, etc.) es
  **transversal**: se recomienda bajarlo junto al departamento, pero solo si lo
  pides.

Pide, por ejemplo: *"baja los estándares de QA"* o *"baja desarrollo junto con
global"*.

## Cambiar de departamento (override)

Si el proyecto ya adoptó un departamento y pides otro, el consultor **no lo baja**
salvo que confirmes el **override**, que **elimina lo materializado** del
departamento anterior (lo registrado en el lock) antes de bajar el nuevo.

## Requisito (acceso a GitHub)

- Un **servidor MCP de GitHub** configurado en el equipo (el mismo que se usa para
  el trabajo diario) con acceso de **lectura** al repo de estándares.

## Reglas

- La fuente de verdad es el repo `estandares-empresa`; aquí **no se edita** ningún
  estándar bajado. Si algo debe cambiar, propón el cambio por **Pull Request** en
  el repo central.
- El consultor **nunca** escribe en el repo de estándares: solo lo lee (vía el MCP
  de GitHub del equipo) y copia hacia `.kiro/` de este proyecto. Aunque el token
  del equipo pueda escribir, el consultor tiene prohibido hacerlo.
