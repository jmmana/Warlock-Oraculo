# tools/asset-forge

Herramienta de línea de comandos para producir el arte de los Office Beans con ayuda de un proveedor de generación de imágenes (BYOK), descrita en `ADR-014` y en la sección 5.4 de la especificación maestra.

No es parte del runtime del producto: no se empaqueta en la imagen Docker de `ADR-013`. Toma el prompt maestro parametrizado y la ficha aprobada de cada arquetipo, genera frames candidatos condicionados por continuidad, los encola para revisión humana y registra procedencia en `assets/licenses/` una vez aprobados.

Sin código todavía: se implementa junto con el epic E10 (`WO-055` a `WO-057`).
