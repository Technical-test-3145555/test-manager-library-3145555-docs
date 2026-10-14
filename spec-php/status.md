# Estado de la entrega — Manager Library (Laravel + React)

Corte: miércoles 14 de octubre de 2026. Cada fila dice qué tarea de [`tasks.md`](tasks.md) tiene evidencia y cuál no.
Una tarea sin evidencia no se marca como hecha (Artículo XII).

## 1. Tareas

| Tarea | Repo | Estado | Evidencia |
|---|---|---|---|
| T-01 | api | hecha | `GET /api/v1/health` responde `{"status":"ok"}`; `HealthTest` en verde |
| T-02 | api | hecha | Seis tablas en el esquema `library`, ninguna columna con `DEFAULT`; los `CHECK` y el índice único parcial rechazan datos inválidos en PostgreSQL 16 |
| T-03 | api | hecha | Pruebas de `Isbn`, `Quantity` y `DateRange` |
| T-04 | api | hecha | Pruebas: disponibles nunca bajan de 0 ni superan el total |
| T-05 | api | hecha | Pruebas con reloj fijo: `open`, `overdue`, `closed`; 14 días; máximo 3 líneas; una devolución no suma dos veces |
| T-06 | api | parcial | La normalización del nombre de usuario y el rechazo del rol `admin` están en los casos de uso y probados. No existen las clases de dominio `Member` y `User`: los casos de uso usan los repositorios directamente |
| T-07 | api | hecha | Login, usuarios y arranque del admin; pruebas HTTP contra PostgreSQL (mismo mensaje con usuario o clave erróneos, 401, 403, 409, 422) |
| T-08 | api | hecha | Libros, búsqueda, baja lógica, portadas y socios; pruebas HTTP contra PostgreSQL |
| T-09 | api | hecha | Préstamo con límite de 3, vencidos, sin ejemplares y todo-o-nada |
| T-10 | api | hecha | Devolución que no suma dos veces (`ITEM_ALREADY_RETURNED`) y lista de vencidos |
| T-11 | api | hecha | Reporte de más prestados; el título viejo se conserva tras renombrar el libro |
| T-12 | api | hecha | Repositorios sobre PostgreSQL real |
| T-13 | api | hecha | Dos préstamos simultáneos del último ejemplar con el servidor levantado: uno recibe 201, el otro 409 y quedan 0 ejemplares (ADR-002) |
| T-14 | api | hecha | Argon2id y JWT HS256 de 8 horas; la clave no aparece en las respuestas |
| T-15 | api | hecha | PNG válido se guarda y se devuelve idéntico; un archivo de texto y uno de más de 2 MB dan 422; comprobado también con `curl` y `PUT multipart` |
| T-16 | api | hecha | Rutas del contrato con cuerpo de error único y autorización por rol |
| T-17 | api | hecha | Con la base vacía, la imagen migra, crea las cinco categorías y el admin inicial; reiniciarla no repite nada |
| T-18 | api | hecha | Los dos `CHECK` están en la migración inicial y rechazan datos inválidos |
| T-19 | api | pendiente | No hay Deptrac ni pruebas de arquitectura |
| T-20 | infra | hecha | Con un solo `docker compose up`, base vacía, API y frontend levantan y responden |
| T-21 | tool | hecha | Tras correrlo: 40 libros, 13 socios, 15 préstamos, 3 líneas vencidas en `/loans/overdue` y el reporte con filas. Se puede repetir sin duplicar |
| T-22 | app | hecha | Login, catálogo, socios, prestar, devolver, vencidos, reporte y equipo; 11 pruebas; recorrido comprobado en el navegador en tema claro y oscuro |
| T-23 | page | hecha | Sin llamadas externas ni a la API |

## 2. Desviaciones respecto al spec

| ID | Desviación | Motivo |
|---|---|---|
| DT-24 | Los casos de uso de `api` se prueban por HTTP contra PostgreSQL real y no con fakes de los puertos | Los puertos existen y están enlazados en un único punto, pero la prueba más útil con el tiempo disponible fue la de contrato. Quedan pendientes las pruebas de aplicación con fakes |
| DT-25 | La tabla `migrations` del framework vive en el esquema `public` | Así `library` queda con exactamente las seis tablas del dominio y el contenedor se reinicia sin repetir migraciones |
| DT-26 | `GET /categories` devuelve una lista simple y no una página | El contrato no define su forma; no hay paginación que aplicar a cinco elementos fijos |
| DT-27 | Una ruta inexistente o un error inesperado también usan el cuerpo único, con `NOT_FOUND` e `INTERNAL_ERROR` | `INTERNAL_ERROR` no está en la tabla de códigos del contrato; se añade solo para no filtrar trazas |

## 3. Problemas resueltos

| ID | Problema | Resolución |
|---|---|---|
| DT-22 | El `Dockerfile` de `api` no compilaba: el `composer.lock` exigía PHP 8.4 | Se fijó la plataforma de Composer en PHP 8.3 y se regeneró el lock. La imagen compila y arranca |
| DT-23 | El plan decía Laravel 11, bloqueado por avisos de seguridad | Se usa Laravel 13; `plan.md` actualizado |
| DT-28 | Al reiniciar el contenedor de `api` fallaba: la tabla `migrations` cambiaba de esquema y las migraciones se repetían | La tabla se declara como `public.migrations`. Comprobado: segundo arranque con "Nothing to migrate" |
| DT-29 | El `compose` no pasaba `DB_CONNECTION` ni `APP_KEY` a la API, que habría usado SQLite | Se pasan los valores y se documentan en `.env.example` |

## 4. Qué se respetó

- El alcance del producto no cambió: mismos atributos, entidades y endpoints que el spec original.
- La documentación se cerró antes de escribir código (`docs`, PR de `hu-01` y `hu-02`).
- Ramas padre sin commits directos; todo entró por ramas hijas y PR.
