# Instrucciones para agentes de código — Warlock Oráculo

Warlock Oráculo es un producto SaaS final: un centro de deliberación multiagente con cinco salas, personajes originales de oficina y ejecución durable. Constrúyelo siempre como producto de producción, nunca como demo desechable. Una pantalla, un placeholder o una simulación no equivalen a una funcionalidad terminada.

## Fuente canónica

1. **Especificación maestra:** [`docs/product/Warlock-Oraculo-Especificacion-Maestra.md`](docs/product/Warlock-Oraculo-Especificacion-Maestra.md). Contiene visión, las cinco salas, arquitectura, contratos, modelo de datos, API, dirección visual y el backlog completo (14 epics, 84 issues padre `WO-001`–`WO-084`, 420 subissues). Léela antes de tocar código.
2. **Backlog ejecutable:** vive en [GitHub Issues](../../issues) de este repositorio, no en un archivo aparte. Cada epic es un [milestone](../../milestones); cada issue padre conserva su ID `WO-XXX` en el título; cada subissue (`WO-XXX.N`) es un sub-issue nativo de GitHub enlazado a su padre. Los IDs `WO-*` son estables aunque GitHub asigne otro número de issue.
3. **ADR:** `docs/adr/ADR-001` a `ADR-012` (sección 6.2 de la especificación). Si tu issue afecta una decisión registrada, actualiza el ADR correspondiente o documenta la excepción en el PR.

## Antes de tocar código

1. Lee el issue asignado completo: historia de usuario, dependencias, entregables, sus subissues, criterios de aceptación y verificación requerida.
2. Confirma que todas sus dependencias (`Depende de: WO-XXX`) están cerradas o acepta y documenta una excepción acotada.
3. Trabaja en una rama propia (`feat/wo-0xx-tema`) y abre un PR que referencie el issue con `Refs #N` o `Closes #N` (usa `Closes` solo si el PR deja el issue realmente completo).

## Selección autónoma del siguiente issue

No esperes a que alguien te asigne tarea por tarea. Un issue está disponible cuando todas sus dependencias están cerradas. Ordena los disponibles por prioridad (P0 antes que P1), riesgo de integración y luego por ID. Si no hay ninguno disponible, documenta el bloqueo concreto en el issue y continúa con otro trabajo independiente; no le pidas al propietario que elija por ti salvo que falte una decisión de producto material o una autorización externa.

## Stack por paquete

| Paquete | Stack previsto | Comandos (una vez bootstrapeado en WO-007) |
| --- | --- | --- |
| `apps/web/` | React 19, TypeScript estricto, Vite, TailwindCSS, PixiJS, i18next (es/en) | `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, `pnpm e2e` |
| `services/api/` | Python 3.13, FastAPI, Pydantic v2 | `uv run ruff check`, `uv run mypy src`, `uv run pytest` |
| `services/worker/` | Python 3.13, LangGraph, PostgreSQL (leases/fencing) | `uv run pytest -m integration` |
| `packages/domain-python/` | Entidades y estados de dominio, sin dependencias de FastAPI ni PixiJS | `uv run pytest` |
| `packages/contracts/` | OpenAPI + JSON Schema → `generated-ts/` | `make contracts` (a definir en WO-009) |
| `packages/scene-runtime/` | Reducers de estado visual, pathfinding, cámara (independiente del motor deliberativo) | `pnpm test` |
| `assets/` | Personajes Office Beans, salones, manifests, licencias (WO-055 a WO-060) | validador de manifest (a definir en WO-060) |
| `supabase/` | Migraciones SQL, RLS, seeds | `supabase db reset` |

Hasta que `WO-007` (bootstrap del monorepo) esté cerrado, estas carpetas son solo el esqueleto de paquetes: cada una trae un `README.md` con su propósito y el/los issue(s) que la llenan de código real.

## Reglas no negociables

- **Dominio puro:** el dominio (`packages/domain-python/`) no importa FastAPI, SDKs de proveedores ni PixiJS (ADR-001).
- **Un solo coordinador:** LangGraph es el único orquestador de turnos por run; no introducir un segundo scheduler que compita por el mismo turno.
- **Persistir antes de emitir:** ningún mensaje se muestra como confirmado sin estar persistido; ninguna llamada a un proveedor de IA ocurre sin su intención registrada primero (operation ledger, sección 7.3–7.4).
- **Secretos:** cifrado AEAD en aplicación (ADR-006). Ninguna llave en claro llega al frontend, a logs, a checkpoints ni a exportaciones. No hay excepciones de "solo para pruebas".
- **RLS real:** toda tabla de contenido compartido se prueba con al menos dos workspaces y los cuatro roles de membresía (Owner/Admin/Member/Viewer).
- **Multilenguaje (es/en):** todo texto visible en `apps/web/` sale de diccionarios i18next; nunca texto literal en un componente.
- **Demos etiquetadas:** cualquier evento, sesión o dato simulado lleva `origin: "demo"` visible en la UI y en exportaciones. Nunca se presenta una simulación como ejecución real.
- **Sin certeza falsa:** una afirmación sin fuente localizable no se marca como verificada; un voto o puntaje de agente es una evaluación ordinal, nunca una probabilidad de verdad.
- **Alcance:** no modifiques archivos fuera de lo que pide tu issue. Si necesitas tocar un contrato o archivo compartido, explícalo en el PR.
- **Pruebas:** cada cambio trae sus pruebas (pytest/hypothesis en motor y API, Vitest/Playwright en el portal). No declares probado lo que no ejecutaste; un test no ejecutado se reporta como no ejecutado, no como "implementado".
- **Estilo:** comentarios y textos en español natural. Nunca uses el carácter em dash (U+2014).
- **Seguridad:** sin secretos reales en el repositorio y sin datos reales de clientes; los fixtures de ejemplo son ficticios y lo dicen.

## Formato de reporte por issue

Al cerrar trabajo sobre un `WO-XXX`, deja en el PR o en un comentario del issue:

```markdown
## WO-XXX — Título
Estado: in_progress | blocked | review | done
Rama/PR: referencia real
Dependencias verificadas: IDs
Subissues terminados: IDs y evidencia
Comportamiento entregado: resultado observable
Validación: comandos ejecutados y resultado real
Contrato/migración/asset cambiado: versión y compatibilidad
Limitaciones: concretas; vacío si no hay conocidas
Siguiente issue disponible: ID y razón de prioridad
```

No hay atajo para esto: "todo listo" sin evidencia no cierra un padre. Un padre cierra cuando sus cinco subissues y su integración están verificados (sección 15.2 de la especificación).
