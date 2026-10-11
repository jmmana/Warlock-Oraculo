# ADR-013: Despliegue en un solo contenedor Docker con PostgreSQL y Auth embebidos

**Estado:** aceptado (revisión 2026-10-11, reemplaza la intención inicial de ADR-001 sobre despliegue separado)
**Issues:** WO-002, WO-079

## Contexto

La primera versión de la especificación asumía frontend en Cloudflare Pages y API/worker en contenedores separados sobre Coolify, con PostgreSQL/Auth/Storage en Supabase administrado o una instalación propia no detallada. Esto exige varios servicios para levantar el producto, incluso en desarrollo o en una demo self-hosted.

## Decisión

La ruta de despliegue **primaria** es una única imagen Docker que contiene:

- PostgreSQL (datos de la aplicación).
- GoTrue (el componente open source de Supabase Auth) sobre ese mismo Postgres.
- La API (FastAPI).
- El worker (cola + coordinador LangGraph).
- El frontend ya compilado, servido como archivos estáticos por la API.

Un supervisor de procesos (`supervisord` o `s6-overlay`) mantiene los cuatro procesos dentro del contenedor y los reinicia de forma independiente si fallan. El storage privado se implementa de forma nativa en la API sobre el volumen persistente del contenedor, en vez de sumar el microservicio Storage de Supabase como quinto proceso. `docker run -p 8080:8080 -v warlock-data:/data warlock-oraculo` deja el producto completo operativo sin infraestructura externa.

Supabase Cloud (gestionado) sigue siendo una **alternativa documentada**: el mismo código de aplicación puede apuntar a un Postgres/Auth/Storage externos en vez de arrancar sus copias locales, y la ruta de servicios separados (Cloudflare Pages + Coolify) sigue disponible para quien necesite escalar API y worker de forma independiente.

## Alternativas consideradas

- **Mantener servicios separados como única ruta:** más flexible para escalar, pero exige varios contenedores/servicios incluso para una instalación de un solo usuario; se descarta como ruta *primaria*, se conserva como alternativa.
- **Reemplazar Supabase por completo (Auth/Storage/RLS propios):** se descartó por ahora; se prefiere reutilizar GoTrue (componente real de Supabase, no una reimplementación) embebido, conservando RLS de Postgres tal como lo usa Supabase.
- **Contenedor único sin Postgres (SQLite):** el motor depende de `FOR UPDATE SKIP LOCKED`, `LISTEN/NOTIFY` y RLS de Postgres (ADR-003, ADR-007); SQLite no sostiene esas garantías, se descarta.

## Consecuencias

- Instalar y probar el producto completo requiere un solo `docker run`, sin Supabase CLI ni Compose multi-servicio como paso obligatorio.
- Postgres y la aplicación comparten ciclo de vida: actualizar la imagen reinicia también la base de datos; se documenta como ventana de mantenimiento esperada, no como una limitación oculta.
- Backups cubren un único volumen (`/data`) en vez de coordinar base de datos y archivos por separado; sigue sin ser snapshot instantáneo de "borrado" (ver sección 8.4 de la especificación).
- Escalar API y worker de forma independiente exige moverse a la ruta alternativa de servicios separados; no es el camino por defecto.
