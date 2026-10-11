# services/worker

Workers Python separados de la API: reclaman jobs con FOR UPDATE SKIP LOCKED, ejecutan el grafo LangGraph por run y escriben eventos/efectos de forma idempotente (ADR-002, ADR-003).

Sin codigo todavia: WO-031 a WO-038 implementan cola, leases, fencing y el coordinador deliberativo.
