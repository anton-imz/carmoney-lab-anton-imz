# Тесты в tests/Unit/ и текущий worktree

## Тесты в `tests/Unit/` (5 файлов)

- **VinValidatorTest.php** — формат VIN через dataProvider: корректный 17-символьный VIN (в т.ч. в нижнем регистре) валиден; короче/длиннее 17 символов, запрещённые `I`/`O`, спецсимвол, пустая строка — нет.
- **LtvCalculatorTest.php** — расчёт LTV в процентах с округлением до 2 знаков (50.0, 33.33, 120.0) и выброс `InvalidArgumentException` при нулевой стоимости или неположительной сумме.
- **DecisionEngineTest.php** — решение по LTV при порогах 60/85: `<60` → approve, серая зона включая границу 85.0 → review, 85.01 и выше → reject.
- **AssessmentServiceTest.php** — сквозной `assess()` на реальном `rules.php`: низкий LTV → approve с `approved_limit` = запрошенной сумме и корректным `vehicle_age`; средний → review с лимитом 0; высокий → reject с лимитом 0.
- **ApplicationValidatorTest.php** — валидация заявки: валидная заявка проходит с нормализацией VIN в верхний регистр; год из будущего → `ValidationException`; сумма ниже минимума даёт ошибку по полю; при нескольких проблемах собираются все ошибки сразу (`vin`, `market_value`, `term_months`).

## Текущий worktree

- Папка: `C:\c\Users\A38B1~1.KAJ\AppData\Local\Temp\kilo\carmoney-lab-anton-imz`
- Ветка: `d1/1.2.1-1.2.3-anton-imz`

C:/c/Users/A38B1~1.KAJ/AppData/Local/Temp/kilo/carmoney-lab-anton-imz                                    fe57dfa [d1/1.2.1-1.2.3-anton-imz]
C:/c/Users/A38B1~1.KAJ/AppData/Local/Temp/kilo/carmoney-lab-anton-imz/.kilo/worktrees/relic-aspen        fe57dfa (detached HEAD)
C:/c/Users/A38B1~1.KAJ/AppData/Local/Temp/kilo/carmoney-lab-anton-imz/.kilo/worktrees/somber-earthquake  fe57dfa [summarize-unit-tests]
PS C:\c\Users\A38B1~1.KAJ\AppData\Local\Temp\kilo\carmoney-lab-anton-imz> 