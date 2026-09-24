# Как считается решение approve / review / reject

Сборка компонентов и загрузка `rules.php` — в `backend/src/AppFactory.php:27-39`; сам расчёт целиком в `backend/src/Domain/`. Точка входа — `AssessmentService::assess()` (backend/src/Domain/AssessmentService.php:28), порядок внутри:

1. **`ApplicationValidator::validate($payload)`** (ApplicationValidator.php:24) — нормализация и валидация по `rules.php`: VIN через `VinValidator::isValid()` (ключ `vin`), год (`vehicle.min_year`, `vehicle.max_age_years`, возраст через `VehicleAge::inYears()`), пробег (`vehicle.max_mileage_km`), стоимость (> 0), сумма (`amount.min/max`), срок (`term.min_months/max_months`). Любая ошибка → `ValidationException`, до решения заявка не доходит. Возвращает нормализованный массив полей.
2. **`LtvCalculator::calculate(requested_amount, market_value)`** (LtvCalculator.php:15) — LTV в процентах, округление до 2 знаков.
3. **`DecisionEngine::decide($ltv)`** (DecisionEngine.php:30) — единственное место, где рождается решение. Пороги `ltv.approve_max = 60.0` и `ltv.review_max = 85.0` из `rules.php` переданы в конструктор (AppFactory.php:37): `$ltv < 60` → `approve`; `60 <= $ltv <= 85` → `review`; `> 85` → `reject`. Нюанс: комментарии (DecisionEngine.php:10, rules.php:39) пишут `LTV <= approve_max`, но код проверяет строгое `<` — при LTV ровно 60.0 будет `review`.
4. **Назад в `assess()`**: `approved_limit` = запрошенная сумма при `approve`, иначе 0 (AssessmentService.php:39). Справочник `ltv_by_age` в решении и лимите не участвует — по коду это несделанная задача LOAN-12 (AssessmentService.php:11-12).

## Куда встанет правило «пробег > 400 000 → review»

Решение принимает `DecisionEngine::decide()`, но она получает только `float $ltv` — пробега у неё нет. Поэтому два возможных места:

- **`DecisionEngine::decide()`** — тогда нужно передавать mileage внутрь (менять конструктор/сигнатуру, AppFactory.php:37) и добавить порог в `rules.php`;
- либо **`AssessmentService::assess()` сразу после строки 33** (`$decision = $this->decisionEngine->decide($ltv);`) — там `$input['mileage']` уже доступен, и решение переопределяется на `review` при превышении порога.

По конвенции проекта само число 400 000 должно попасть в `backend/config/rules.php` (AGENTS.md: пороги не хардкодим).

**Что уже есть:** значение пробега — оно валидируется и возвращается нормализованным (ApplicationValidator.php:43-46, 78), т.е. в `assess()` доступно как `$input['mileage']`.

**Чего не хватает:**

- порога 400 000 в `rules.php` — нет (есть только `max_mileage_km = 500000`, это лимит валидации, а не порог решения);
- передачи пробега в `DecisionEngine` — нет;
- механизма комбинирования правил решения (LTV + пробег) — нет; при конфликте (LTV-`reject` + пробег-`review`) приоритет в коде не определён — нет.

Учтите: заявки с пробегом > 500 000 отбрасываются валидацией, до решения не доходят — новое правило фактически сработает в диапазоне 400 000 < пробег <= 500 000.

## Что сейчас проверяется про пробег

Единственная проверка — `ApplicationValidator::validate()`, ApplicationValidator.php:43-46: `0 <= mileage <= 500 000` (из `vehicle.max_mileage_km`, rules.php:23); нарушение → ошибка `mileage`, `ValidationException`. Далее: в LTV пробег не входит (LtvCalculator.php:15 — только сумма и стоимость), в решении не участвует (DecisionEngine.php:30 — только LTV). Других проверок пробега в `backend/src/Domain/` и `rules.php` — нет.
