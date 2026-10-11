# Procedencia: assets portados desde Agent Viewer (2026-10-11)

**Autorización:** Juan Manuel Castillo Pinto, propietario de producto de ambos repositorios (Warlock Oráculo y Agent Viewer), autorizó explícitamente este port el 2026-10-11 tras revisar el origen y las limitaciones descritas abajo. Esta autorización resuelve la condición que ADR-014 dejaba pendiente ("alguien con autoridad sobre ambos proyectos debe confirmar autoría original").

**Repositorio origen:** [`jmmana/Agent-Viewer`](https://github.com/jmmana/Agent-Viewer), mismo propietario que este repositorio.
**Licencia en origen:** MIT (ver `LICENSE` de ese repositorio).
**Destino en este repositorio:** `assets/source/` (material fuente/semilla, no asset final empaquetado). Todo lo listado aquí conserva su estado real de origen; nada se promueve a `approved` por el solo hecho de copiarse.

## Qué se copió

| Destino aquí | Origen en Agent Viewer | Commit de origen | Estado heredado |
| --- | --- | --- | --- |
| `assets/source/characters/ceo/approved-concept.png` | `assets/crew/references/ceo-approved-concept.png` | `a878c293d671898089b614091e21c8cf9c587f78` | `reference` — ficha de estilo aprobada, 1536×1024, nunca editada en el sitio. **Única referencia de estilo "Office Beans" aprobada que existe hoy en cualquiera de los dos proyectos.** |
| `assets/source/characters/ceo/idle-{front,back,left,right}.png`, `phone-front.png`, `work-front.png` | `assets/crew/bank/characters/ceo/*.png` | rama `main` de Agent Viewer | `prototype` — poses estáticas, 256×352, ancla `(0.5, 0.9375)`. **No usar el canvas/ancla tal cual**: la sección 5.3 de nuestra especificación exige 512×512 y ancla `(0.5, 0.88)`; sirven como referencia visual, no como asset final a empaquetar sin reprocesar. |
| `assets/source/characters/other-archetypes/{analyst,developer,finance,planner,reviewer}-idle-front.png` | `assets/crew/bank/characters/<rol>/idle-front.png` | rama `main` de Agent Viewer | `prototype` — un único frame estático por rol, sin otras direcciones ni poses. Mismo punto de partida que tenía Agent Viewer; no hay trabajo de animación que heredar para estos 5. |
| `assets/source/characters/ceo/walk-atlas-prototype/*.png` + `manifest.v1.json` | `assets/crew/clips/ceo-walk-*.png`, `assets/crew/clips/manifest.v1.json` | `790ee664a5b0e476828202e13e4783b90c6b44bf` | `prototype` — único ciclo de caminar real que existe en cualquiera de los dos proyectos (4 direcciones × 8 frames, generado con `image_gen`, medido a pixel, documentado en `docs/crew/FRAME-PIPELINE.md` de Agent Viewer). Limitaciones conocidas documentadas en origen: pequeñas inconsistencias de escala/alpha entre frames generados; nunca presentado como arte final allá tampoco. |
| `assets/source/furniture/*.svg` (19 archivos + `catalog.json`) | `assets/furniture/*.svg` | `355e9194bd83d8b741bc65d5b58a897d320ee584` (rama `feat/visual-assets-v1`, **PR #43 de Agent Viewer, nunca integrado a su `main`**) | `prototype`, procedencia marcada en origen como no verificada de forma independiente antes de esta autorización. Ver advertencia abajo. |
| `assets/source/electronics/*.svg` (13 archivos) | `assets/electronics/*.svg` | igual que arriba | igual que arriba |
| `assets/source/effects/*.svg` (9 archivos) | `assets/effects/*.svg` | igual que arriba | igual que arriba |

## Advertencia heredada sobre el mobiliario/electrónica/efectos

La documentación de migración de Agent Viewer (`docs/crew/ASSET-MIGRATION-PR43.md` en ese repositorio) marca estos 41 SVG como procedentes de una rama draft nunca mergeada, con la nota explícita: *"las alegaciones de código local en otra sesión siguen no verificadas sin commits o material original"*. Es decir: en el momento de esa migración, no había una confirmación independiente de que estos SVG fueran dibujo original (vs. derivados de alguna librería externa). La autorización del propietario de producto el 2026-10-11 para portarlos a este repositorio se basa en su propio conocimiento de cómo se produjeron originalmente, no en una verificación automática de este proceso. Si en el futuro aparece evidencia de que alguno de estos SVG no es original, debe retirarse de este repositorio y de cualquier build publicado.

## Qué significa "fuente", no "final"

Nada en `assets/source/` se usa directamente en el runtime del producto ni se empaqueta en un atlas sin pasar por el ciclo de vida de `ADR-014` (`reference → vector-study → prototype → approved`) y, para los personajes, sin alcanzar las 8 direcciones reales que exige la sección 5.3 (aquí solo hay 4, heredadas de la cámara de 4 vistas de Crew). `tools/asset-forge/` trata estos archivos como material de partida para generar las fichas y atlas definitivos de Warlock Oráculo, no como el resultado final.
