# Guía por repositorio — Manager Library (Laravel + React)

Qué se construye en cada uno de los seis repositorios, con qué tareas de [`tasks.md`](tasks.md) y con qué evidencia. Nada aquí cambia el alcance del
producto: solo reparte el trabajo ya especificado.

| Repo | Tecnología | Tareas | Estado |
|---|---|---|---|
| `docs` | Markdown | spec completo | cerrado en `spec-php/` |
| `infra` | Docker Compose, PostgreSQL 16 | T-20 | PR abierto en `hu-01-dev` |
| `api` | PHP 8.3, Laravel 11 | T-01 a T-19 | pendiente |
| `tool` | Script que usa la API | T-21 | pendiente |
| `app` | React 18 + Vite | T-22 | pendiente |
| `page` | HTML y CSS estáticos | T-23 | pendiente |

## `api` — backend

**Qué es:** reglas de negocio y contrato HTTP de [`api-contract.md`](api-contract.md) bajo `/api/v1`.

**Estructura:** la de [`architecture.md`](architecture.md) §2 (`Domain`, `Application`, `Adapters`, `Bootstrap`, `database/migrations`, `tests`).

**Se construye en este orden:** T-01 esqueleto y `/health`; T-02 migración inicial con las seis tablas, los `CHECK` y las cinco categorías; T-03 a T-06 dominio;
T-07 a T-11 casos de uso con fakes; T-12 a T-15 adaptadores; T-16 rutas y error único; T-17 composición y admin inicial; T-19 Deptrac.

**Evidencia mínima por entrega:** pruebas de dominio y aplicación en verde, la migración corre sobre la base vacía de `infra`, las consultas de
[`data-model.md`](data-model.md) §10 coinciden, y dos préstamos del último ejemplar dan un 201 y un 409.

**Decisiones que ya se aplican:** DT-01 (sin columna de versión), DT-03 (los dos `CHECK` en la migración), DT-07 (orden de validación del préstamo),
DT-17 y DT-18 (modelos con `$table` y sin timestamps; sin migraciones por defecto de Laravel).

## `infra` — contenedores y motor vacío

**Qué es:** `docker compose up` con `db` (PostgreSQL 16 vacío, UTC, volumen), `api` y `app`. No contiene `CREATE TABLE` (ADR-001).
**Evidencia:** la base arranca sana y con 0 tablas; `scripts/check-no-ddl.sh` imprime OK.

## `tool` — sembrador de demostración

**Qué es:** un script que crea datos **solo por la API**, nunca por SQL (`data-model.md` §9): al menos 40 libros, 10 socios, 15 préstamos y 3 vencidos.
**Dependencia:** necesita la API funcionando (T-17). Para dejar 3 vencidos hace falta poder fijar la fecha del préstamo; como la API no lo permite, se
usa el desfase del reloj `CLOCK_OFFSET_DAYS` (DT-21) y no una columna ni un endpoint nuevo.
**Evidencia:** tras correrlo, `/loans/overdue` devuelve 3 líneas y el reporte tiene filas.

## `app` — frontend

**Qué es:** pantallas de operación sobre el contrato: login, catálogo, socios, prestar, devolver, vencidos y reporte.
**Reglas de interfaz:** el token vive en memoria; cada pantalla muestra el mensaje en español que trae el cuerpo de error único; las rutas se protegen por
rol (el reporte solo para `admin`).
**Evidencia:** recorrido de punta a punta login, prestar, devolver, reporte.

## `page` — sitio público

**Qué es:** presentación estática de Manager Library, **sin llamar a la API**.
**Evidencia:** se abre sin la API y se ve bien en móvil y en escritorio.
