# Estado de implementación e integración

Este registro distingue el contrato original de decisiones confirmadas por el
usuario del fork y evidencia ejecutada. No modifica los ADR ni declara completo
el proyecto.

## Los seis repositorios

| Repositorio | Responsabilidad y avance |
|---|---|
| docs | Especificaciones y cuatro ADR conservados; este registro documenta decisiones y pendientes. |
| api | Laravel 13/PHP 8.4; seis tablas, categorías, autenticación, catálogo, socios, préstamos, devoluciones y reporte histórico. |
| app | React con API real, mensajes españoles, token en memoria; lint, pruebas y build disponibles. |
| page | Presentación estática sin conexión a la API; responsive y navegación por teclado. |
| infra | Compose: PostgreSQL 16 vacío, API que ejecuta migraciones, frontend por proxy y sitio estático. |
| tool | Sembrador exclusivo por API para 40 libros, 10 socios y 15 préstamos; vencidos inmediatos pendientes. |

## Decisiones confirmadas durante la implementación

Estas confirmaciones proceden del usuario del fork; no implican una aprobación
independiente del responsable original del proyecto:

- `xmin` de PostgreSQL como testigo optimista, sin nueva columna ni modificación
  del ADR-002; no se expone en la API.
- Paginación con valores predeterminados 1/20 y rechazo 422 de valores no enteros,
  página menor de 1 o tamaño fuera de 1–50.
- Códigos `UNAUTHENTICATED` (401) e `INTERNAL_ERROR` (500), manteniendo el envelope.
- Respuestas de categorías como lista de `{id,name}` y socios creados/listados
  con `{id,full_name,document_number}`; detalle con `active_loans`.
- Préstamos listados/detallados con los campos del ejemplo del contrato; retiro
  de libro y futura eliminación de portada con 204.

La aplicación usa un advisory lock transaccional por socio para preservar el
límite global y coordinar devoluciones. Los libros siguen utilizando control
optimista, no `SELECT FOR UPDATE`. Esta es una decisión de implementación,
no un cambio del límite de tres ni una nueva columna.

## Evidencia y comandos

En `api`, después de configurar PHP portable y Composer:

```powershell
php artisan migrate
php artisan library:provision-admin
php artisan serve --host=127.0.0.1 --port=8000
php vendor\phpunit\phpunit\phpunit
php vendor\bin\pint --test
```

La integración PostgreSQL requiere variables de proceso, una base exclusiva
`library_test` y `LIBRARY_TEST_DESTRUCTIVE=yes`:

```powershell
php vendor\phpunit\phpunit\phpunit -c phpunit.pgsql.xml
```

Las migraciones se ejecutaron sobre PostgreSQL 16.15 real: esquema exacto de 30
columnas, FK, índices parciales, unicidad, rollback y segunda ejecución. Las
pruebas de API verifican permisos, búsquedas, límites, atomicidad, devoluciones y
reporte histórico. Las pruebas de concurrencia usan dos procesos independientes
con el kernel HTTP real y PostgreSQL, no una base simulada.

En `app`: `npm ci`, `npm run lint`, `npm test`, `npm run build`.
En `tool`: `npm test` y `node --env-file=.env seed.mjs`.
En `infra`: completar `.env` desde `.env.example`, `docker compose config --quiet`
y `docker compose up --build -d`. Aplicación en 8080, sitio en 8081, API en 8000.
No utilizar `down -v` ni migraciones destructivas con datos compartidos.

## Pendientes conservados

No se han cerrado por suposición: rol de descarga de portadas, actualización de
ISBN, redondeo de días de retraso, formato de filtros de fecha de préstamos,
metodología de rendimiento y procedimiento para generar vencidos inmediatos por
API. Portadas y lista operativa de vencidos no están publicadas.

`ck_book_total_positive` y `ck_user_role` siguen pendientes T-18 conforme al modelo.
No se promete que esos CHECK existan. La base protege el rango de disponibilidad.
La interfaz y el sitio no deben presentarse como un sistema de producción ni como
evidencia del cumplimiento de la meta de rendimiento.

La documentación nueva de este bloque está en español, conforme al artículo XI.
Se conservan las traducciones anteriores y sus notas; no se reescribe contenido
válido para ocultar la desviación previa.
