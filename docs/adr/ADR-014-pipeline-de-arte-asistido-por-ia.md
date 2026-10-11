# ADR-014: Pipeline asistido por IA para el arte de los Office Beans

**Estado:** aceptado (revisión 2026-10-11; **revisión 2026-10-11b** incorpora el pipeline ya probado en Agent Viewer; **revisión 2026-10-11c** registra el port de assets ya realizado)
**Issues:** WO-055, WO-056, WO-057, WO-058

## Contexto

La sección 5.4 de la especificación describía un proceso de producción de arte (fichas, poses maestras, animaciones) asumiendo un equipo de diseño ejecutándolo manualmente, con un "prompt maestro sugerido" como referencia textual. No existía ninguna herramienta real del repositorio para ejecutar ese proceso, ni un contrato de procedencia específico para frames generados por un modelo de imagen.

**Precedente directo:** Agent Viewer (otro producto de WarlockCode) ya construyó y validó en producción un modo "Crew" con un estilo "Office Beans" chibi, los mismos 6 arquetipos cosméticos (CEO, Planner, Developer, Analyst, Reviewer, Finance), y un pipeline real de generación asistida por IA para el arquetipo CEO (`docs/crew/FRAME-PIPELINE.md` de ese repositorio): referencia aprobada → prompt exacto versionado → salida de un modelo de generación de imágenes → medición manual de rectángulos reales por frame (nunca una grilla asumida) → manifest versionado con hash/licencia/commit/prompt → aprobación humana explícita antes de promover de `prototype` a `approved`. Esta ADR adopta ese mismo pipeline en vez de inventar uno nuevo, porque ya está probado: produjo un ciclo de caminar real (4 direcciones × 8 frames) para un arquetipo, documentado con limitaciones honestas (calidad `prototype`, pequeñas inconsistencias de escala entre frames generados).

## Decisión

Se construye `tools/asset-forge/`: una herramienta de línea de comandos, fuera del runtime del producto, con el mismo ciclo de vida que ya usa Agent Viewer para Crew:

**Estados del asset:** `reference` (fuente de estilo/identidad, exactamente una por personaje salvo una versión nueva explícita, ej. `reference.ceo.v2`, nunca una edición en el sitio) → `vector-study` (estudios de proporción/línea, excluidos del runtime, solo material de apoyo para quien dibuja/ajusta prompts) → `prototype` (seguro de integrar en el motor de escena para ingeniería; nunca presentado como arte final) → `approved` (pasó revisión visual contra la biblia de estilo de `WO-055`, con las ocho direcciones reales requeridas por la sección 5.3 presentes y pasando el chequeo de identidad).

**Proceso por frame:**

1. Toma el prompt maestro parametrizado (`[ROL]`, `[ACCESORIO]`, `[DIRECCIÓN]`, `[ESTADO]`, `[ÍNDICE]`/`[TOTAL]`) y la ficha `reference` aprobada del arquetipo como imagen de condicionamiento.
2. Llama a un proveedor de generación de imágenes configurado por BYOK, reutilizando el mismo `ProviderAdapter`/`capability` que el resto del producto (sin credenciales embebidas).
3. Condiciona cada frame con la ficha de referencia y, cuando aplica, el frame anterior de la secuencia, para sostener identidad y continuidad. El prompt exacto usado y el intento rechazado (si lo hay) se preservan en `assets/source/origins/<secuencia>/`, igual que Agent Viewer conserva sus intentos descartados: nunca se borra el historial de qué no funcionó.
4. **Mide, no asume:** las coordenadas de cada frame dentro de un atlas se miden sobre los píxeles reales decodificados (alpha real, no estimado), igual que hace `FRAME-PIPELINE.md` de Agent Viewer. Un generador de imágenes no entrega una grilla perfecta; tratar su salida como si lo fuera produce sprites con las manos o pies cortados.
5. Registra en un manifest versionado (`assets/manifest.v1.json`, un documento por lote): `schemaVersion`, `id` (`<rol>.<variante>.<dirección>.v<versión>`), dimensiones de raster reales, resolución lógica, FPS, ancla normalizada, si hace loop, y por cada frame: rectángulo medido, ancla, duración, orden y hash; además licencia, commit de origen, referencia usada, archivo/hash del prompt exacto y notas de revisión visual.
6. Un CLI de importación (`asset-forge check` / `asset-forge import <lote>.json`) coacciona toda entrada nueva a `prototype`, valida antes de escribir, rechaza reemplazar un catálogo existente que difiera, y escribe de forma atómica. Reimportar lo mismo es idempotente. La aprobación (`prototype` → `approved`) es siempre un cambio humano explícito de estado en el manifest con quien aprobó registrado; el importador no puede otorgarla.
7. **Ninguna vista faltante se finge.** Si falta una de las 8 direcciones (N/NE/E/SE/S/SW/W/NW) requeridas por la sección 5.3, el manifest la marca `artAvailability: 'missing'` y la escena muestra el fallback explícito de la sección 5.3; nunca se espeja, rota o estira una dirección existente para simular otra. Esta regla es más estricta que la de Agent Viewer: Crew solo sostiene 4 cámaras (frente/derecha/atrás/izquierda) y nunca llegó a producir las 8 direcciones completas ni para su arquetipo más avanzado (CEO); Warlock Oráculo sí las necesita, así que el trabajo real por arquetipo es mayor al ya hecho ahí, no un copiar-pegar.

La herramienta acelera la producción de los 1.632 frames objetivo (sección 5.3); no sustituye la aprobación de fichas, la guía común de proporción/luz/cámara, ni la revisión de continuidad y alpha a tamaño real de uso.

## Reutilización de Agent Viewer: qué se portó y qué sigue pendiente

**Ya portado (2026-10-11c), con autorización explícita del propietario de producto de ambos proyectos.** Procedencia completa, commits de origen y advertencias heredadas en [`assets/licenses/source-agent-viewer-2026-10-11.md`](../../assets/licenses/source-agent-viewer-2026-10-11.md):

- **Semilla de estilo del CEO:** `assets/source/characters/ceo/` contiene la ficha de referencia aprobada, las 6 poses estáticas y el único ciclo de caminar real (prototipo) que existe en cualquiera de los dos proyectos. Es punto de partida, no asset final: su lienzo/ancla (`256×352`, `(0.5, 0.9375)`) y sus 4 direcciones no cumplen todavía la sección 5.3 (`512×512`, `(0.5, 0.88)`, 8 direcciones reales). Adaptarla y extenderla es trabajo de `WO-056`/`WO-057`, no algo ya terminado por copiarse.
- **Punto de partida de los otros 5 arquetipos:** `assets/source/characters/other-archetypes/` tiene un frame estático por rol, igual que tenía Agent Viewer. No hay animación que heredar ahí; el trabajo es el mismo que empezar de cero.
- **Mobiliario/electrónica/efectos:** `assets/source/{furniture,electronics,effects}/` tiene los ~41 SVG explorados en la rama no mergeada de Agent Viewer (PR #43). Su procedencia quedaba marcada como no verificada de forma independiente en el repositorio de origen; el port a este repositorio se autorizó explícitamente conociendo esa advertencia (ver el documento de procedencia), no porque la verificación automática la haya resuelto.
- **El pipeline de medición-y-manifest** descrito arriba se adopta casi literal de `scripts/crew-clip-assets.mjs` de Agent Viewer.

**Sigue pendiente, sin importar lo anterior:** nada en `assets/source/` es `approved`. Para cada arquetipo faltan las 8 direcciones reales (Crew solo cubre 4), la aprobación visual contra la biblia de estilo de `WO-055`, y el registro en el manifest versionado de `tools/asset-forge/` antes de usarse en el motor de escena.

## Alternativas consideradas

- **Solo proceso manual (ilustradores humanos) sin herramienta:** más control de calidad desde el día uno, pero no escala a 1.632 frames en un proyecto de una persona; se descarta como único camino.
- **Generación sin cola de revisión ni medición manual:** más rápido, pero es exactamente lo que Agent Viewer identificó como riesgo real (inconsistencias de escala/alpha entre frames) y contradice la regla de la sección 5.4 ("una hoja bonita no garantiza un sprite legible").
- **Copiar el catálogo de muebles de la rama draft de Agent Viewer sin que el propietario de producto autorizara conociendo la advertencia de procedencia:** descartado; se hizo con autorización explícita (ver sección anterior y el documento de procedencia).

## Consecuencias

- `tools/asset-forge/` es un paquete nuevo del monorepo (fuera de `apps/` y `services/`, no se despliega con el producto).
- El costo de generación de imágenes se registra con el mismo `usage_ledger`/BYOK que cualquier otra llamada a un proveedor de IA (ADR-005, ADR-009); generar assets no es gratis ni queda fuera del presupuesto del proyecto.
- Un frame sin su entrada de procedencia en el manifest bloquea el empaquetado del atlas correspondiente; un frame `prototype` nunca se presenta como `approved` sin el cambio de estado explícito.
- El arquetipo CEO parte con ventaja real (ficha de referencia ya aprobada en otro producto); los otros cinco arquetipos empiezan desde cero, igual que en Agent Viewer.
