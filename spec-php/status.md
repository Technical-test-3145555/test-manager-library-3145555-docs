# Estado de la entrega — Manager Library (Laravel + React)

Corte: miércoles 14 de octubre de 2026, antes de las 11:00 a. m. Cada fila dice qué tarea de [`tasks.md`](tasks.md) tiene evidencia y cuál no.
Una tarea sin evidencia no se marca como hecha (Artículo XII).

## 1. Tareas

| Tarea | Repo | Estado | Evidencia |
|---|---|---|---|
| T-01 | api | hecha | `GET /api/v1/health` responde `{"status":"ok"}`; `HealthTest` en verde |
| T-02 | api | hecha | Seis tablas, ninguna columna con `DEFAULT`; los `CHECK` y el índice único parcial rechazan datos inválidos en PostgreSQL 16 |
| T-03 | api | hecha | Pruebas de `Isbn`, `Quantity` y `DateRange` |
| T-04 | api | hecha | Pruebas: disponibles nunca bajan de 0 ni superan el total |
| T-05 | api | hecha | Pruebas con reloj fijo: `open`, `overdue`, `closed`; 14 días; máximo 3 líneas; una devolución no suma dos veces |
| T-06 a T-19 | api | pendiente | Usuarios, casos de uso, persistencia, JWT, portadas, rutas HTTP, composición y Deptrac |
| T-20 | infra | hecha | `docker compose config` válido; el motor arranca sano con 0 tablas; `scripts/check-no-ddl.sh` imprime OK. Los servicios `api` y `app` no se probaron de punta a punta |
| T-21 | tool | pendiente | Sin código |
| T-22 | app | pendiente | Sin código |
| T-23 | page | hecha | Sin llamadas externas ni a la API |

Total de pruebas automáticas del dominio: 18 pruebas, 32 aserciones, más la prueba de salud.

## 2. Problemas conocidos

| ID | Problema | Efecto | Siguiente paso |
|---|---|---|---|
| DT-22 | El `Dockerfile` de `api` no compila: `composer install` falla con el código 2 dentro de la imagen `php:8.3-cli`. Se subió sin haberse probado. | `docker compose up` no puede construir `api`. No afecta al código ni a las pruebas, que corren fuera de Docker. | Reproducir el error de `composer install`, corregirlo o quitar el archivo en su propia rama. |
| DT-23 | El plan decía Laravel 11, pero todas sus versiones 11.x están bloqueadas por avisos de seguridad de Composer. | Se usó Laravel 13 con PHP ^8.3. | Resuelto en `plan.md` (D-02). |

## 3. Qué se respetó

- El alcance del producto no cambió: mismos atributos, entidades y endpoints que el spec original.
- La documentación se cerró antes de escribir código (`docs`, PR de `hu-01` y `hu-02`).
- Ramas padre sin commits directos; todo entró por ramas hijas y PR.
