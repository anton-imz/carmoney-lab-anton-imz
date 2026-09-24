1. Сервис: учебный сервис предварительной оценки заявки на заём под ПТС; считает LTV и возвращает решение approve / review / reject.
2. Makefile: `make up` — поднять сервис и базу (docker compose up -d --build), `make down` — остановить, `make test` — PHPUnit, `make lint` — `php -l` по backend/ и tests/, `make seed` — залить db/seed.sql, `make logs`, `make ps`, `make install`, `make help`. docker-compose.yml: сервис `backend` (php -S на 8080), сервис `db` (mysql:8.0, порт 3307→3306), том db-data, initdb через db/schema.sql и db/seed.sql.
3. Решение approve / review / reject считается в `backend/src/Domain/`: класс `DecisionEngine.php` (правила в `backend/config/rules.php`), участвуют также `LtvCalculator.php`, `AssessmentService.php`.

модель: training-2026-09-minimax-m3
