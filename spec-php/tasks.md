# Tareas — Manager Library (Laravel + React)

Cada tarea termina cuando su **evidencia** existe y se puede ejecutar (artículo XII). Estado inicial de todas: **pendiente**.
Los números T-01 a T-23 coinciden con las versiones Python y .NET; las diferencias están marcadas con (*) y explicadas en [`technical-debt.md`](technical-debt.md).

| # | Tarea | Repo | Depende de | Evidencia |
|---|---|---|---|---|
| T-01 | Esqueleto Laravel (solo API) y `GET /api/v1/health` | api | — | `GET /health` responde `{"status":"ok"}` |
| T-02 (*) | Migración inicial: seis tablas, FK, índices únicos, `ck_book_total_positive`, `ck_user_role` y las cinco categorías sembradas (absorbe T-18, DT-03 y DT-04) | api | T-01 | Las consultas de `data-model.md` §10 devuelven lo documentado; ninguna columna con `DEFAULT`; un `INSERT` por `psql` que viole los `CHECK` falla |
| T-03 | Dominio: `Isbn`, `Quantity`, `DateRange` | api | T-01 | Pruebas unitarias: ISBN de 13 dígitos, cantidad > 0, fin ≥ inicio |
| T-04 | Dominio: `Book` (`withdraw`, `restock`, baja lógica) | api | T-03 | Pruebas: nunca < 0, nunca > total |
| T-05 | Dominio: `Loan` (`addItem`, `returnItem`, estado calculado, 14 días, máx. 3) | api | T-04 | Pruebas con un `Clock` falso: `open`, `overdue`, `closed` |
| T-06 | Dominio: `Member` y `User` (normalización de nombre, rol válido) | api | T-03 | `"Ana "` queda `ana`; rol `admin` por API rechazado |
| T-07 | Puertos y casos de uso de autenticación (`LoginService`, `CreateLibrarianService`) con fakes | api | T-06 | HU-01 y HU-02 pasan contra fakes |
| T-08 | Puertos y casos de uso de libros (crear, buscar, editar, baja, portada) con fakes | api | T-04 | HU-03 y HU-04 pasan contra fakes |
| T-09 | Caso de uso prestar (`LendBooksService`) con fakes, en el orden de DT-07 | api | T-05, T-08 | HU-06 completa: límites, vencidos, sin ejemplares |
| T-10 | Casos de uso devolver y vencidos | api | T-09 | HU-07 y HU-09; devolver dos veces da `ITEM_ALREADY_RETURNED` |
| T-11 | Caso de uso reporte de más prestados y su puerto de lectura | api | T-09 | HU-10; el título viejo se conserva tras renombrar (ADR-004) |
| T-12 | Adaptador de persistencia: repositorios de libro, socio, usuario (Eloquent con `$table` explícito y sin timestamps) | api | T-02, T-08 | Pruebas de integración contra PostgreSQL real |
| T-13 (*) | Adaptador de persistencia: repositorio de préstamos con `UPDATE` condicional atómico (ADR-002 de esta versión) | api | T-12, T-09 | Dos préstamos del último ejemplar: uno 201, otro 409 (RNF-2) |
| T-14 | Adaptador de seguridad: Argon2id y JWT HS256 (`firebase/php-jwt`) | api | T-07 | La clave nunca aparece en logs ni respuestas (RNF-3) |
| T-15 | Adaptador de portadas (disco) | api | T-08 | Subir PNG de 2 MB ok; 3 MB da 422; sin portada `cover_key` nulo |
| T-16 | Adaptador HTTP: rutas de [`api-contract.md`](api-contract.md), cuerpo de error único, autorización por rol, `GET /books/{id}/cover` (DT-02) | api | T-10, T-11, T-14, T-15 | Prueba de contrato recorre todas las rutas y todos los códigos de §2 |
| T-17 | Composición y arranque: `PortBindingsServiceProvider` y `app:bootstrap-admin` | api | T-16 | El sistema arranca con la base vacía y se puede iniciar sesión como admin |
| T-18 (*) | Verificación de que las invariantes quedaron en el motor (la creación se hizo en T-02) | api | T-12 | `data-model.md` §4 coincide con las consultas de §10 |
| T-19 (*) | Pruebas de arquitectura (artículos I, II y III) con Deptrac | api | T-17 | Deptrac y `tests/Architecture/` pasan y fallan si se introduce una dependencia prohibida |
| T-20 | `infra`: `docker compose up` levanta motor vacío, API y frontend | infra | T-17 | Un comando; la base arranca sin tablas hasta correr las migraciones |
| T-21 | `tool`: sembrador de demostración **por la API** (≥ 40 libros, 10 socios, 15 préstamos, 3 vencidos) | tool | T-17 | Tras correrlo, `/loans/overdue` devuelve 3 líneas y el reporte tiene filas |
| T-22 | `app`: login, catálogo, socios, prestar, devolver, vencidos, reporte | app | T-16 | Recorrido manual de punta a punta |
| T-23 | `page`: sitio público estático de presentación | page | — | Se abre sin la API; correcto en móvil y escritorio |

## Orden de entrega por repositorio

Cada repositorio avanza por ramas hijas `hu-<n>-dev` con commits pequeños y un PR hacia `develop`. Las ramas `develop`, `qa` y `main` nunca reciben
commits directos. Primero se cierra esta documentación (`docs`), después `infra`, `api`, `tool`, `app` y `page`.

## Definición de terminado de la prueba

Quien llegue a T-17 con evidencia tiene el **núcleo** hecho. Se evalúa el desempeño con SDD, no la cantidad de tareas: una tarea con evidencia
vale más que tres a medias.
