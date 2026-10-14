# Plan técnico — Manager Library (Laravel + React)

Cómo se construye lo que dicen [`spec.md`](spec.md) y [`constitution.md`](constitution.md). El esquema **no** se describe aquí: vive en
[`data-model.md`](data-model.md). La forma del sistema, en [`architecture.md`](architecture.md).

## 1. Decisiones técnicas

| # | Decisión | Elección (Laravel + React) | Por qué |
|---|---|---|---|
| D-01 | Lenguaje y runtime | PHP 8.3 | Stack del reto |
| D-02 | Framework HTTP | Laravel 11 (solo API, sin sesiones ni vistas) | Contrato en [`api-contract.md`](api-contract.md) |
| D-03 | Persistencia | Eloquent + migraciones de Laravel sobre PostgreSQL 16, esquema `library` | El esquema es de las migraciones (ADR-001) |
| D-04 | Concurrencia | `UPDATE` condicional atómico + `CHECK` de rango ([ADR-002](adr/adr-002-concurrencia-optimista.md)) | Último ejemplar (RNF-2); sin columna de versión (DT-01) |
| D-05 | Fechas | Instantes UTC (`timestamptz`); el reloj es un puerto (`Clock`) | Probar vencimientos sin esperar 14 días |
| D-06 | Reporte | Una consulta agregada por un puerto de lectura ([ADR-004](adr/adr-004-reporte-agregado-y-congelado.md)) | Estable y barato |
| D-07 | Objetos de valor | `Isbn`, `Quantity`, `DocumentNumber`, `DateRange`: viven dentro de la fila de su dueño, sin tabla | No tienen identidad |
| D-08 | Portadas | Binario en almacenamiento externo (disco en local, S3 en producción) con `Storage` de Laravel detrás de `CoverStorage`; la base guarda solo `cover_key` | Clave opaca, nunca ruta |
| D-09 | Contraseñas | Argon2id (`Hash` de Laravel, driver `argon2id`) detrás del puerto `PasswordHasher` | Artículo IX |
| D-10 | Arranque | Comando `app:bootstrap-admin` crea el admin inicial desde `ADMIN_USERNAME` / `ADMIN_PASSWORD` | La base no puede producir el hash |
| D-11 | Autenticación | JWT HS256, 8 horas, clave en `JWT_SECRET`, con `firebase/php-jwt` | Simple y sin estado; Sanctum no emite JWT |
| D-12 | Pruebas | Pest/PHPUnit: dominio y aplicación con fakes; base real solo en integración | Pirámide de pruebas |
| D-13 | Arquitectura | Deptrac para las reglas de capas | Equivalente a `import-linter` |
| D-14 | Frontend | React 18 + Vite + React Router; cliente HTTP con `fetch`; token en memoria | Stack del reto |
| D-15 | Contenedores | Docker Compose: `db` (PostgreSQL 16 vacío), `api` (PHP-FPM + Nginx), `app` | RNF-5: un solo comando |

## 2. Configuración por entorno

| Variable | Obligatoria | Ejemplo |
|---|---|---|
| `DB_URL` | sí | `pgsql://library:library@db:5432/library` |
| `DB_SCHEMA` | sí | `library` |
| `JWT_SECRET` | sí | cadena larga aleatoria |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | sí | definidos en `.env`, nunca en el repo |
| `COVERS_DIR` | no | `storage/app/covers` |
| `CORS_ORIGIN` | sí | `http://localhost:5173` |
| `VITE_API_URL` | sí (app) | `http://localhost:8080/api/v1` |

## 3. Estrategia de pruebas

| Nivel | Qué prueba | Contra qué |
|---|---|---|
| Dominio | Invariantes: máximo 3, vencimiento a 14 días, disponibles en rango | Nada: objetos puros |
| Aplicación | Casos de uso: prestar, devolver, reporte | Fakes de los puertos |
| Integración | Repositorios y consulta del reporte | PostgreSQL real |
| Contrato | Cuerpo de error único y códigos de estado | La API levantada |
| Arquitectura | Artículos I, II y III | Deptrac y `tests/Architecture/` |
| Concurrencia | Dos préstamos del último ejemplar | PostgreSQL real |
| Frontend | Recorrido login, prestar, devolver, reporte | Vitest y Testing Library |

## 4. Orden de construcción

Esqueleto y salud → migración inicial con semilla → dominio → casos de uso con fakes → adaptadores → API → pruebas de contrato y concurrencia → frontend.
El detalle, con evidencia, está en [`tasks.md`](tasks.md).
