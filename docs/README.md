# Documentación de Warlock Oráculo

| Documento | Qué contiene |
| --- | --- |
| [`product/Warlock-Oraculo-Especificacion-Maestra.md`](product/Warlock-Oraculo-Especificacion-Maestra.md) | Fuente canónica: visión, personas, las cinco salas, dirección visual, arquitectura, modelo de datos, API/contratos, el backlog completo (14 epics, 84 issues, 420 subissues) y la instrucción maestra para agentes. |
| [`adr/`](adr) | Decisiones de arquitectura (ADR-001 a ADR-012, sección 6.2 de la especificación). Se llenan en `WO-002`. |
| [`design/`](design) | Biblia visual, sistema de diseño y manifests de personajes/salones (epic E10). |
| [`api/`](api) | Documentación generada de la API una vez publicado el contrato OpenAPI (`WO-009`). |
| [`operations/`](operations) | Runbooks de despliegue, backup/restore y rotación de claves (epic E14). |
| [`agents/`](agents) | Guía extendida de ejecución para agentes (`WO-003`), complementa [`AGENTS.md`](../AGENTS.md) en la raíz. |

El backlog ejecutable **no vive en un archivo**: vive en los [Issues](../../../issues) y [Milestones](../../../milestones) de este repositorio, importado 1:1 desde la sección 15 de la especificación. Los IDs `WO-001`…`WO-084` (y sus `WO-XXX.N`) se conservan en los títulos de los issues aunque GitHub les asigne otro número.
