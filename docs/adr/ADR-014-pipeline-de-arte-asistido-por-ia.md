# ADR-014: Pipeline asistido por IA para el arte de los Office Beans

**Estado:** aceptado (revisión 2026-10-11)
**Issues:** WO-055, WO-056, WO-057

## Contexto

La sección 5.4 de la especificación describía un proceso de producción de arte (fichas, poses maestras, animaciones) asumiendo un equipo de diseño ejecutándolo manualmente, con un "prompt maestro sugerido" como referencia textual. No existía ninguna herramienta real del repositorio para ejecutar ese proceso, ni un contrato de procedencia específico para frames generados por un modelo de imagen.

## Decisión

Se construye `tools/asset-forge/`: una herramienta de línea de comandos, fuera del runtime del producto, que:

1. Toma el prompt maestro parametrizado y la ficha aprobada de cada arquetipo como imagen de referencia.
2. Llama a un proveedor de generación de imágenes configurado por BYOK, reutilizando el mismo `ProviderAdapter`/`capability` que el resto del producto (sin credenciales embebidas).
3. Condiciona cada frame con la ficha maestra y, cuando aplica, el frame anterior de la secuencia, para sostener identidad y continuidad.
4. Encola cada frame generado como `pendiente_revisión`; ningún frame entra a un atlas sin aprobación humana explícita.
5. Registra procedencia (proveedor, modelo, prompt final, fecha, persona que aprobó) en `assets/licenses/` para cada frame aprobado, con el mismo rigor que cualquier otro asset distribuido.

La herramienta acelera la producción de los 1.632 frames objetivo (sección 5.3); no sustituye la aprobación de fichas, la guía común de proporción/luz/cámara, ni la revisión de continuidad y alpha a tamaño real de uso.

## Alternativas consideradas

- **Solo proceso manual (ilustradores humanos) sin herramienta:** más control de calidad desde el día uno, pero no escala a 1.632 frames en un proyecto de una persona; se descarta como único camino.
- **Generación sin cola de revisión:** más rápido, pero contradice la regla de la sección 5.4 ("una hoja bonita no garantiza un sprite legible") y el principio de no declarar terminado lo no verificado.

## Consecuencias

- `tools/asset-forge/` es un paquete nuevo del monorepo (fuera de `apps/` y `services/`, no se despliega con el producto).
- El costo de generación de imágenes se registra con el mismo `usage_ledger`/BYOK que cualquier otra llamada a un proveedor de IA (ADR-005, ADR-009); generar assets no es gratis ni queda fuera del presupuesto del proyecto.
- Un frame sin su entrada de procedencia en `assets/licenses/` bloquea el empaquetado del atlas correspondiente.
