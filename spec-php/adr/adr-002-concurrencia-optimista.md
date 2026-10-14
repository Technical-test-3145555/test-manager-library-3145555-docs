# ADR-002 · Concurrencia sobre los ejemplares sin columna de versión

- **Estado:** aceptada (reemplaza la versión Python y .NET; ver DT-01 en [`technical-debt.md`](../technical-debt.md))
- **Contexto:** dos bibliotecarios pueden prestar el **último ejemplar** del mismo libro al mismo tiempo (RNF-2). Si ambos leen
  `available_copies = 1` y ambos escriben, el sistema presta un libro que no tiene. La versión original pedía un «testigo de versión»
  por libro, pero [`data-model.md`](../data-model.md) no tiene esa columna, y por el Artículo X gana el modelo.
- **Decisión:** el descuento es una sola sentencia atómica y condicional dentro de la misma transacción que crea la línea:
  `UPDATE library.book SET available_copies = available_copies - 1 WHERE id = ? AND available_copies > 0 AND deleted_at IS NULL`.
  Si afecta **0 filas**, el servidor responde **409 `BOOK_NOT_AVAILABLE`** y la transacción se revierte completa (CA-06.3: no se registra nada).
  La **última barrera** sigue siendo el `CHECK` `ck_book_available_range` (`0 <= available_copies <= total_copies`).
- **Alternativas descartadas:** agregar una columna `version` (cambia el modelo de datos sin un requisito que lo pida); bloqueo pesimista
  de la fila (`SELECT … FOR UPDATE`) en cada préstamo: serializa toda la biblioteca por un caso raro; contador en memoria: no sobrevive a más de un proceso.
- **Consecuencias:** el resultado es el mismo que con versión (uno gana, el otro recibe 409) con menos piezas. Si el `CHECK` salta, **algo escribió
  fuera del adaptador**: es un defecto, no un caso de negocio. Devolver dos veces la misma línea no suma dos veces porque `returned_at` ya está
  lleno (CA-07.2): la devolución marca la línea con `UPDATE … WHERE returned_at IS NULL` y solo suma si afectó una fila.
- **Cómo se verifica:** prueba concurrente con dos peticiones simultáneas sobre un libro con un ejemplar: exactamente una recibe 201.
