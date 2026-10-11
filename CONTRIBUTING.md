# Contribuir a Warlock Oráculo

1. Lee [`AGENTS.md`](AGENTS.md) y la [especificación maestra](docs/product/Warlock-Oraculo-Especificacion-Maestra.md) completa antes de abrir un PR.
2. Elige un issue `WO-XXX` disponible (sus dependencias ya cerradas) desde los [Issues](../../issues), ordenado por prioridad.
3. Trabaja en una rama `feat/wo-0xx-tema`, implementa sus cinco subissues y deja evidencia real (comandos ejecutados, no solo "funciona").
4. Abre el PR con la plantilla del repositorio, referenciando el issue (`Refs #N` o `Closes #N`).
5. `make check` (o el comando equivalente de tu paquete) debe pasar antes de pedir revisión.
6. No se aceptan secretos reales, datos de clientes reales, ni demos sin etiquetar como `origin: "demo"`.

Este es software original: no copies assets, prompts protegidos de terceros, ni código con licencia incompatible. Toda dependencia nueva se declara en el lockfile correspondiente.
