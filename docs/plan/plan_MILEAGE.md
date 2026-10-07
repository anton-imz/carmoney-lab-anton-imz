# plan_MILEAGE — пробег больше 400 000 км → решение review

Задача 1.4.3 (день 1). Формулировка: «пробег авто не больше 400 000 км, иначе
решение review». Граница закреплена формулировкой: 400 000 — ещё approve,
400 001 — review (сравнение строгое `>`).

Основание: `docs/setup/code_map.md` (п. 2–3), scout-разведка по `mileage`,
`AGENTS.md`. Спеки `spec_MILEAGE.md` ещё нет — при появлении сверить план с ней.

Правило: пробег больше `vehicle.review_mileage_km` (400 000 км) понижает решение
`approve` до `review`. Решения `review` и `reject` пробег не меняет: пробег не
«повышает» решение, reject не смягчается (рекомендация — см. вопрос 1).

## Файлы

- `backend/config/rules.php` — новый ключ `vehicle.review_mileage_km => 400000` в секции `vehicle`, рядом с `max_mileage_km`, с комментарием о смысле (порог понижения решения, не валидация).
- `backend/src/Domain/AssessmentService.php` — порог пробега в конструктор, понижение approve→review в `assess()`, обновление докблока класса.
- `backend/src/AppFactory.php` — сборка в `create()`: передать `$rules['vehicle']['review_mileage_km']` в `AssessmentService`.
- `tests/Unit/AssessmentServiceTest.php` — новый аргумент в `setUp()`, параметр пробега в хелпере `payload()`, граничные тесты правила.
- `tests/Unit/ApplicationValidatorTest.php` — тест на пустой пробег.

Всё, чего нет в списке, при реализации не трогается: `DecisionEngine`,
`ApplicationValidator`, `LtvCalculator`, БД (`schema.sql`/`seed.sql`), фронтенд,
`README.md` — без изменений.

## Шаги

1. `rules.php`: добавить ключ `vehicle.review_mileage_km => 400000` с комментарием (число учебное; по конвенции бизнес-числа только здесь).
2. `AssessmentService`: добавить последним параметром конструктора порог — сигнатура `__construct(ApplicationValidator $validator, LtvCalculator $ltvCalculator, DecisionEngine $decisionEngine, VehicleAge $vehicleAge, int $reviewMileageKm)`.
3. `AssessmentService::assess()`, после строки с `$decision = $this->decisionEngine->decide($ltv);`: если решение `APPROVE` и `$input['mileage'] > $this->reviewMileageKm` — решение становится `DecisionEngine::REVIEW`.
4. Обновить докблок `AssessmentService`: упомянуть правило понижения approve→review по пробегу.
5. `AppFactory::create()`: передать `$rules['vehicle']['review_mileage_km']` последним аргументом в `AssessmentService`.
6. Тесты: обновить `AssessmentServiceTest::setUp()` (порог из того же `$rules`), расширить хелпер `payload()` параметром `int $mileage = 96000`, добавить тесты из раздела «Тесты».
7. Прогнать `make test` и `make lint`.

## Тесты

Новые в `tests/Unit/AssessmentServiceTest.php` (база: LTV 50% → approve по LTV, LTV 95% → reject по LTV):

- 399999 → `approve`, approved_limit = запрошенной сумме (порог не превышен);
- 400000 → `approve` (граница «не больше 400 000 км»: сравнение строгое `>`);
- 400001 → `review`, approved_limit = 0;
- 400001 при LTV 95% → `reject` (пробег не смягчает reject — при подтверждении вопроса 1);
- пустой пробег (нет ключа `mileage` или null) → `ValidationException` с ключом ошибки `mileage` → HTTP 422 — тест в `tests/Unit/ApplicationValidatorTest.php`, сейчас такого теста нет.

Существующий `testApprovesLowLtvAndSetsLimitToRequestedAmount` (пробег 96000) остаётся зелёным — регресс на «обычных» заявках.

## Риски

- Смена сигнатуры конструктора `AssessmentService`: обе точки сборки (`AppFactory::create()`, `AssessmentServiceTest::setUp()`) правятся в одном шаге, иначе падает сборка и все юнит-тесты.
- Существующая валидация `max_mileage_km` (0–500 000) не меняется: правило реально срабатывает только для 400 001–500 000 км; пробег выше 500 000 — это HTTP 422 (заявка не принимается к оценке), а не review/reject.
- Пороги связаны: `review_mileage_km` должен оставаться меньше `max_mileage_km`, иначе часть зоны правила отсекается валидацией до вычисления решения.
- Пониженная до review заявка получает `approved_limit = 0` (существующая строка 39 `AssessmentService`) — ожидаемо, но видно в ответе API.
- Пустая строка в `mileage` (`''`) сегодня нормализуется в 0 и проходит валидацию — существующее поведение валидатора для всех числовых полей; тест «пустой пробег» покрывает отсутствие ключа/null, а не `''`.
- Известное расхождение докблока `DecisionEngine` с кодом (строгое `<` при LTV ровно 60.0) — вне задачи, не трогаем.
- Не входит: расчёт лимита по `ltv_by_age` (LOAN-12), изменение валидации пробега, смена контракта `DecisionEngine::decide()`, фронтенд, БД, написание `spec_MILEAGE.md`, документация.

## Вопросы заказчику (без спеки не решить)

1. Комбинирование с LTV-решением: при пробеге 400 001 и LTV-`reject` — оставить `reject` (рекомендация: пробег не «повышает» решение) или переводить в `review`? В плане принят первый вариант; тест «400001 при LTV 95%» при другом ответе меняется.
2. Имя ключа конфига `vehicle.review_mileage_km` — предложено планом; если спека закрепит другое имя, заменить (код читает ключ из `rules.php`, имя свободное).
3. Пустая строка `mileage: ''` нормализуется в 0 и проходит валидацию (существующее поведение валидатора, одинаковое для всех числовых полей) — оставить как есть (вне задачи) или чинить отдельной задачей?
