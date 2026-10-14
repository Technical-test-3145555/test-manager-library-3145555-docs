# Plan de ramas y entrega — Manager Library

Cómo se entrega el trabajo: ramas hijas, commits pequeños y un PR por avance. Complementa [`tasks.md`](tasks.md) y [`repositories.md`](repositories.md).

## 1. Reglas

| Regla | Detalle |
|---|---|
| Ambientes | `develop`, `qa` y `main` en cada repositorio |
| Ramas padre | Nunca reciben commits directos ni PR desde una rama de trabajo |
| Rama de trabajo | `hu-<n>-dev`, cortada desde `develop`, con PR hacia `develop` |
| Promoción | Se hace con ramas hijas: `hu-<n>-qa` cortada desde `qa` y `hu-<n>-main` desde `main`, cada una con su PR |
| Repositorio base del PR | Siempre el fork de la organización `Technical-test-3145555`, nunca el del profesor |
| Commits | Pequeños y frecuentes, en inglés, con el prefijo `tipo(hu-<n>):`. Ningún commit gigante |
| Código antes de documentación | No se escribe código de un repositorio hasta que su guía en [`repositories.md`](repositories.md) esté cerrada |

## 2. Orden de entrega

| Paso | Repo | Rama | Qué entra |
|---|---|---|---|
| 1 | `docs` | `hu-01-dev` | `spec-php/`: traducción y deuda técnica (fusionado) |
| 2 | `infra` | `hu-01-dev` | Compose con el motor vacío, `api` y `app` (T-20) |
| 3 | `docs` | `hu-02-dev` | Decisiones adoptadas, guía por repositorio y este plan |
| 4 | `api` | `hu-01-dev` | T-01 y T-02: esqueleto, `/health` y migración inicial |
| 5 | `api` | `hu-02-dev` | T-03 a T-06: dominio |
| 6 | `api` | `hu-03-dev` | T-07 a T-11: casos de uso con fakes |
| 7 | `api` | `hu-04-dev` | T-12 a T-17: persistencia, seguridad, HTTP y arranque |
| 8 | `app` | `hu-01-dev` | T-22: pantallas |
| 9 | `tool` | `hu-01-dev` | T-21: sembrador |
| 10 | `page` | `hu-01-dev` | T-23: sitio público |

El orden prioriza el **núcleo** (T-01 a T-17) porque quien llega ahí con evidencia tiene lo esencial hecho.

## 3. Evidencia que acompaña cada PR

Cada PR dice qué tarea de `tasks.md` cierra y cómo se comprobó (comando y resultado). Una tarea sin evidencia no se marca como terminada (Artículo XII).
