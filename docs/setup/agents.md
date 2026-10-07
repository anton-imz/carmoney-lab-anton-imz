# Роли агентов

В проекте две специализированные роли. Описания их системных промптов лежат в
`.kilo/agents/planner.md` и `.kilo/agents/scout.md`.

## planner

- **Что делает:** читает код, `AGENTS.md`, `docs/setup/code_map.md` и пишет план
  в `docs/plan/`.
- **Что может:** читать файлы, искать по коду, создавать и редактировать только
  файлы в `docs/plan/`.
- **Что не может:** менять исходный код, конфиги или любые другие файлы вне
  `docs/plan/`. Не запускает изменяющих команд.
- **Контракт плана:** разделы **Файлы**, **Шаги**, **Тесты**, **Риски**. Если
  данных не хватает — перечисляет, чего не хватает, а не додумывает.

## scout

- **Что делает:** находит в коде то, что просят, и возвращает список: файл,
  строка, одна фраза — что там.
- **Что может:** только читать и искать по коду.
- **Что не может:** менять файлы, предлагать исправления, писать планы.

## Что вернул scout по запросу «mileage»

Места, где читается `mileage`:

- `backend/src/Domain/ApplicationValidator.php:43` — читает `mileage` из payload:
  `(int)($payload['mileage'] ?? -1)`.
- `backend/src/Domain/ApplicationValidator.php:44-45` — проверяет диапазон по
  `rules['vehicle']['max_mileage_km']`.
- `backend/src/Domain/ApplicationValidator.php:78` — возвращает нормализованный
  `mileage` в `$input`.
- `backend/src/Repository/ApplicationRepository.php:38-46` — INSERT в
  `vehicles.mileage_km` из `$input['mileage']`.
- `backend/src/Repository/ApplicationRepository.php:68` — SELECT `v.mileage_km`
  для list/show.
- `backend/config/rules.php:23` — порог `vehicle.max_mileage_km => 500000`.
- `db/schema.sql:22` — колонка `mileage_km INT UNSIGNED NOT NULL`.
- `db/seed.sql:31` — INSERT с `mileage_km`.
- `frontend/index.html:30-31` — `<label for="mileage">` + `<input name="mileage"
  type="number">`.
- `frontend/app.js:8` — `'mileage'` в `NUMERIC_FIELDS` (парсится как число).
- `tests/Unit/AssessmentServiceTest.php:38` — фикстура `'mileage' => 96000`.
- `tests/Unit/ApplicationValidatorTest.php:34` — фикстура `'mileage' => 84000`.
- `README.md:47` — пример curl с `"mileage":84000`.
- `docs/setup/code_map.md:28,33,67,76,81,85,92,100,102` — описания поведения и
  пробелов.
- `docs/plan/README.md:14` — упоминание существующего `max_mileage_km`.

Замечание scout: `mileage` нигде не участвует в расчёте LTV, решении или лимите
— только валидация диапазона и сохранение.
