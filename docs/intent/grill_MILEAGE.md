# grill_MILEAGE — лог интервью по intent правила пробега

- Дата: 2026-10-08.
- Режим: Grill Me — вопросы по одному, не больше шести, только те, от ответа на
  которые зависит реализация; перед вопросами сверка с документами и кодом.
- Тема: intent задачи 1.4.3 «пробег авто не больше 400 000 км, иначе решение
  review» (день 1, учебный сервис, данные синтетические).
- Итог: задано 2 из 6 допустимых вопросов; оба ответа подтвердили позиции,
  уже принятые в коде и плане, — изменений реализации не требуется.

## Сверка до вопросов

Сверился с `docs/setup/code_map.md`, `docs/plan/plan_MILEAGE.md` и кодом:
`backend/config/rules.php`, `backend/src/Domain/AssessmentService.php`,
`ApplicationValidator.php`, `DecisionEngine.php`, `backend/src/AppFactory.php`,
`tests/Unit/AssessmentServiceTest.php`, `tests/Unit/ApplicationValidatorTest.php`.

Открытие: правило уже реализовано — code_map (п. 2 «Чего не хватает») и план
описывают состояние до реализации:

- `backend/config/rules.php:25` — `vehicle.review_mileage_km => 400000`;
- `backend/src/Domain/AssessmentService.php:39-41` — понижение approve → review
  при пробеге выше порога;
- `backend/src/AppFactory.php:39` — порог прокинут в конструктор `AssessmentService`;
- тесты: `AssessmentServiceTest.php:74-108` (границы 399999/400000/400001,
  «reject не смягчается»), `ApplicationValidatorTest.php:56-76`
  (отсутствие/null пробега → 422).

## Темы, не заданные как вопросы (закрыты сверкой)

- **Граница 400 000**: 400 000 = approve, 400 001 = review, сравнение строгое
  `>` — закреплено формулировкой «не больше 400 000 км», планом
  (plan_MILEAGE.md:4) и тестами.
- **Пробег не указан / null**: `(int)($payload['mileage'] ?? -1)` →
  `ValidationException` → HTTP 422, решение не считается
  (ApplicationValidator.php:43-46); валидатор в задаче не менялся.
- **Механика**: порог в `rules.php`, понижение в `AssessmentService` после
  `decide()`; `DecisionEngine`, фронтенд и БД не тронуты — совпадает с
  non-goals плана (plan_MILEAGE.md:56).

## Вопрос 1 — сочетание с LTV-reject

Вопрос: заявка с LTV > 85% (reject по LTV) и пробегом 400 001+ — какое итоговое
решение должно быть закреплено в intent?

Варианты:

- «Оставить reject (Recommended)» — высокий пробег понижает только
  approve → review, reject не смягчается. Так в коде (AssessmentService.php:39)
  и тесте `testHighMileageDoesNotSoftenReject`; рекомендация плана
  (plan_MILEAGE.md:60).
- «Переводить в review» — высокий пробег перекрывает любой исход LTV,
  включая reject; потребовалась бы правка AssessmentService.php:39 и замена
  теста.

Ответ заказчика: **Оставить reject**.

## Вопрос 2 — пустая строка mileage

Вопрос: пустая строка `mileage: ''` нормализуется в 0 и проходит валидацию —
существующее поведение валидатора, одинаковое для всех числовых полей
(ApplicationValidator.php:43-46). Вносить ли починку в scope этой задачи?

Варианты:

- «Вне scope (Recommended)» — оставить как есть (позиция плана,
  plan_MILEAGE.md:54,62); при необходимости — отдельная задача по валидатору.
  Задача остаётся только про понижение approve → review.
- «Чинить в этой задаче» — проверка пустой строки в `ApplicationValidator`
  и тест; расширение scope за рамки правила пробега.

Ответ заказчика: **Вне scope**.

## Итог

- Подтверждено: reject при пробеге 400 001+ остаётся reject; пустая строка
  `mileage: ''` — вне scope.
- Остальные темы интервью закрыты сверкой: граница 400 000 включительно,
  неуказанный/null пробег → HTTP 422, механика понижения в `AssessmentService`,
  non-goals плана.
- Текст intent согласован в чате по итогам интервью; на момент этого лога файл
  `docs/intent/intent_MILEAGE.md` не создан.
