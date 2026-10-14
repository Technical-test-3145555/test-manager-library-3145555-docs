# Deuda técnica y hallazgos del spec — Manager Library (Laravel + React)

Registro de lo encontrado al traducir el spec a PHP (Laravel) y React. Regla de trabajo: **el alcance del producto no se toca**.
No se renombran atributos, entidades ni endpoints; no se agregan ni quitan historias. Lo que no se puede cambiar se **registra aquí**
con la decisión provisional que se aplica en la construcción, para que el propietario del spec la confirme.

Estados: **decidido** (se aplica y queda documentado), **adoptada** (decisión provisional que se usa en la construcción; si el propietario del spec la cambia, se ajusta en un PR y no antes).

## 1. Contradicciones entre documentos

| ID | Hallazgo | Dónde | Decisión | Estado |
|---|---|---|---|---|
| DT-01 | ADR-002 exige un «testigo de versión» por libro, pero `data-model.md` no tiene ninguna columna de versión en `book`. Con la regla «gana el motor» (Artículo X) el modelo manda. | ADR-002, `data-model.md` §3 | No se agrega columna. La concurrencia se resuelve con un `UPDATE` condicional atómico (`available_copies = available_copies - 1 WHERE id = ? AND available_copies > 0`) más el `CHECK` `ck_book_available_range`. Se reescribe ADR-002 en `spec-php`. | decidido |
| DT-02 | `GET /books/{id}/cover` se menciona en el texto de §3.4 (devuelve el binario o 404) pero **no aparece** en la tabla de endpoints ni tiene rol. | `api-contract.md` §3.4 | Se implementa como endpoint de rol «autenticado», igual que el resto de lecturas de libros. No se cambia la tabla original. | adoptada |
| DT-03 | `ck_book_total_positive` y `ck_user_role` están marcadas «pendiente (T-18)», pero el Artículo VI dice que toda invariante expresable en el motor se declara en el motor. T-18 es opcional, por lo que el motor quedaría sin esas barreras. | `data-model.md` §4, `tasks.md`, Artículo VI | Se crean en la migración inicial (T-02). T-18 queda como verificación. | decidido |
| DT-04 | `tasks.md` pone T-18 después de T-17 aunque las restricciones pertenecen al esquema (T-02, ADR-001: todo el DDL vive en las migraciones). | `tasks.md` | Se fusiona con T-02 en `spec-php/tasks.md`. | decidido |
| DT-05 | `arquitectura-panoramica.md` está en español mientras todos los demás archivos de documentación están en inglés (Artículo XI: nombres técnicos en inglés). | nombre de archivo | En `spec-php` se llama `architecture-overview.md`. El archivo original no se toca. | decidido |

## 2. Reglas ambiguas o no definidas en el contrato

| ID | Hallazgo | Decisión provisional | Estado |
|---|---|---|---|
| DT-06 | No hay `code` de error para un token ausente, inválido o vencido (401). `INVALID_CREDENTIALS` está definido solo para el inicio de sesión (CA-01.2). | Se reutiliza `INVALID_CREDENTIALS` con mensaje genérico, para no inventar códigos nuevos. | adoptada |
| DT-07 | El orden de validación de `POST /loans` no está definido y las reglas se pisan (vencidos 409, límite 422, sin ejemplares 409, repetido 422). | Orden: validación de entrada (`VALIDATION_ERROR`, `DUPLICATE_BOOK_IN_LOAN`) → socio existe (`NOT_FOUND`) → `MEMBER_HAS_OVERDUE` → `MEMBER_LIMIT_EXCEEDED` → libros y ejemplares (`NOT_FOUND`, `BOOK_NOT_AVAILABLE`). | adoptada |
| DT-08 | CA-06.4 («no más de 3 sin devolver en total») no dice si cuentan las líneas del préstamo nuevo. | Cuentan: `sin_devolver_actuales + líneas_nuevas <= 3`. El 422 informa el conteo actual. | adoptada |
| DT-09 | `PUT /books/{id}` «reemplaza título, autor, categoría y total»: el ISBN no está. | El ISBN es inmutable tras la creación. | adoptada |
| DT-10 | `size` mayor a 50 o menor a 1, y `page` menor a 1: el contrato no dice si se recorta o se rechaza. | Se rechaza con 422 `VALIDATION_ERROR`. | adoptada |
| DT-11 | Portada inválida (tipo distinto de JPG/PNG o mayor a 2 MB): el contrato no da `code`. | 422 `VALIDATION_ERROR` (coincide con T-15: 3 MB da 422). | decidido |
| DT-12 | `from` y `to` del reporte son fechas «ambos inclusive», pero `loaned_at` es un instante. | Rango en UTC: `loaned_at >= from 00:00:00Z` y `loaned_at < (to + 1 día) 00:00:00Z`. | adoptada |
| DT-13 | `days_late` no define el redondeo. Un ítem vencido hace 3 horas tendría 0 días. | Días completos transcurridos desde `due_at`, con mínimo 1 mientras esté vencido. | adoptada |
| DT-14 | `GET /loans/overdue` choca con `GET /loans/{id}` si la ruta variable se registra primero. | La ruta fija se registra antes y `{id}` se restringe a UUID. | decidido |
| DT-15 | El `sub` del JWT no dice qué identificador lleva. | `sub` = `user.id` (UUID). Sin datos personales dentro (§4 del contrato). | decidido |

## 3. Nombres y convenciones (sin cambiar alcance)

| ID | Hallazgo | Decisión | Estado |
|---|---|---|---|
| DT-16 | Tablas en singular (`book`, `loan_item`, `user`) pero rutas en plural (`/books`, `/loans`, `/users`). | Es la convención REST y la del modelo (§0). No se cambia ninguna. Se documenta para que nadie la «corrija». | decidido |
| DT-17 | Laravel asume tablas en plural y marcas `created_at`/`updated_at`. El modelo exige singular y **sin** columnas de auditoría. | Cada modelo declara `$table` explícito y `$timestamps = false`. UUID como clave, sin `DEFAULT` en la base. | decidido |
| DT-18 | Laravel crea por defecto `users`, `password_reset_tokens`, `sessions`, `cache`, `jobs`. El modelo son seis tablas. | Se eliminan esas migraciones. La única tabla extra es `migrations` del framework, que no es del dominio. | decidido |
| DT-19 | Los nombres de las cinco categorías de la semilla están en español (Novela, Ciencia, Tecnología, Historia, Infantil). | Son **datos de producto** con identificadores fijos, no nombres de código. No se traducen. | decidido |
| DT-20 | La carpeta se llama `spec-.net` (con punto). | No se renombra: es del propietario. | decidido |

## 4. Traducción de tecnología (Python o .NET a Laravel y React)

| Spec original | En el reto |
|---|---|
| FastAPI + SQLAlchemy + Alembic | Laravel + Eloquent + migraciones de Laravel |
| `import-linter` / NetArchTest | Deptrac (reglas de capas) |
| pytest | Pest o PHPUnit (dominio puro sin framework) |
| Argon2id vía `PasswordHasher` | `Hash` de Laravel con driver `argon2id` detrás del puerto |
| JWT HS256 de 8 horas | `firebase/php-jwt` (Sanctum no emite JWT con `sub` y `role`) |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | Comando de arranque `app:bootstrap-admin` que lee el entorno |
| Frontend | React con Vite, consumiendo `/api/v1` |
