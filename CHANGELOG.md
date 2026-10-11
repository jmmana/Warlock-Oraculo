# Changelog

Formato: [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/). Este proyecto sigue versionado semántico una vez publique su primera versión instalable (`WO-084`).

## [Unreleased]

### Added

- Especificación maestra del producto (`docs/product/Warlock-Oraculo-Especificacion-Maestra.md`).
- Backlog completo importado a GitHub Issues: 14 epics, 84 issues padre, 420 subissues.
- Esqueleto del monorepo, `AGENTS.md`, ADR pendientes (`docs/adr/`) y plantillas de PR/issue.
- `ADR-013`: despliegue en una sola imagen Docker (web + API + worker + PostgreSQL + GoTrue embebidos), con Supabase Cloud y servicios separados como rutas alternativas.
- `ADR-014`: pipeline `tools/asset-forge/` de generación de arte asistido por IA, con cola de revisión humana y procedencia obligatoria.

### Changed

- Sección 6.1, 6.5 y 13.4 de la especificación maestra: la ruta de despliegue primaria pasa de "frontend en Cloudflare Pages + API/worker en Coolify + Supabase administrado" a "una sola imagen Docker autocontenida"; las rutas anteriores quedan como alternativa documentada.
