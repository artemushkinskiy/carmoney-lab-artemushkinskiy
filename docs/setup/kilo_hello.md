# kilo hello

1) По README — это учебный сервис **предварительной оценки заявки на заём под ПТС**: принимает заявку (VIN, год, пробег, оценочная стоимость, сумма, срок), считает LTV и возвращает решение `approve` / `review` / `reject`. Все данные синтетические.

2) В Makefile цели: `help`, `up` (поднять сервис и базу на 8080), `down`, `ps`, `logs`, `install` (composer), `test` (PHPUnit), `lint` (`php -l` по `backend/` и `tests/`), `seed` (залить `db/seed.sql` в базу). В `docker-compose.yml` два сервиса: `backend` (php -S на 8080, билдится из `backend/Dockerfile`, переменные `DB_DSN/DB_USER/DB_PASSWORD/APP_DEBUG`, порт через `APP_PORT`) и `db` (mysql:8.0, initdb-скрипты `db/schema.sql` и `db/seed.sql`, том `db-data`, healthcheck через `mysqladmin ping`). Команд docker-compose напрямую в репозитории не нашёл — только через `make`.

3) Решение считается в `backend/src/Domain/DecisionEngine.php` (константы `APPROVE/REVIEW/REJECT`, пороги `approve_max` и `review_max`); LTV считает `backend/src/Domain/LtvCalculator.php`, склейка — `backend/src/Domain/AssessmentService.php`, HTTP-вход — `backend/src/Http/ApplicationController.php` (маршруты в `backend/public/router.php`).

модель: training-2026-09-minimax-m3