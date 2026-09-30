# Как считается решение approve / review / reject

Сборка компонентов и загрузка `rules.php` — в `backend/src/AppFactory.php:27-40`; сам расчёт целиком в `backend/src/Domain/`. Точка входа — `AssessmentService::assess()` (backend/src/Domain/AssessmentService.php:29), порядок внутри:

1. **`ApplicationValidator::validate($payload)`** (ApplicationValidator.php:24) — нормализация и валидация по `rules.php`: VIN через `VinValidator::isValid()` (ключ `vin`), год (`vehicle.min_year`, `vehicle.max_age_years`, возраст через `VehicleAge::inYears()`), пробег — обязателен и в диапазоне `0..vehicle.max_mileage_km`, стоимость (> 0), сумма (`amount.min/max`), срок (`term.min_months/max_months`). Любая ошибка → `ValidationException`, до решения заявка не доходит. Возвращает нормализованный массив полей.
2. **`LtvCalculator::calculate(requested_amount, market_value)`** (LtvCalculator.php:15) — LTV в процентах, округление до 2 знаков.
3. **`DecisionEngine::decide($ltv)`** (DecisionEngine.php:30) — пороги `ltv.approve_max = 60.0` и `ltv.review_max = 85.0` из `rules.php` переданы в конструктор (AppFactory.php:38): `$ltv < 60` → `approve`; `60 <= $ltv <= 85` → `review`; `> 85` → `reject`. Нюанс: комментарии (DecisionEngine.php:10, rules.php:39) пишут `LTV <= approve_max`, но код проверяет строгое `<` — при LTV ровно 60.0 будет `review`.
4. **Правило пробега в `assess()`** (AssessmentService.php:37-39) — если решение `approve` и пробег больше порога `vehicle.review_mileage_km` (rules.php:24, по умолчанию 400 000), решение понижается до `review`. Переопределение стоит до расчёта лимита, поэтому `approved_limit` становится 0 (baseline по задаче MILEAGE; открытые вопросы OQ-1/OQ-2 переданы аналитику).
5. **Лимит в `assess()`** (AssessmentService.php:44): `approved_limit` = запрошенная сумма при `approve`, иначе 0. Справочник `ltv_by_age` в решении и лимите не участвует — по коду это несделанная задача LOAN-12 (AssessmentService.php:11-12).

## Где встало правило «пробег > 400 000 → review»

Правило применяется в `AssessmentService::assess()` сразу после `DecisionEngine::decide($ltv)` (AssessmentService.php:37-39) — `$input['mileage']` к этому моменту уже нормализован валидатором. Условие: `$decision === APPROVE && $input['mileage'] > $this->reviewMileageKm` → `$decision = REVIEW`. На `review` и `reject` правило не действует (`review` уже `review`; `reject` не смягчается — baseline по OQ-1). Сам порог `review_mileage_km` живёт в `backend/config/rules.php:24` и передаётся в `AssessmentService` через конструктор (AppFactory.php:40), чтобы число не хардкодилось (AGENTS.md).

**Зона действия:** заявки с пробегом > `vehicle.max_mileage_km` (500 000) отбрасываются валидацией и до правила не доходят — правило фактически сработает в диапазоне 400 000 < пробег ≤ 500 000 (это значение не менялось; см. `docs/spec/spec_MILEAGE.md` §1 «Не входит»).

## Что сейчас проверяется про пробег

Две проверки:

- **`ApplicationValidator::validate()`** (ApplicationValidator.php:43-53): пробег обязателен — ключ `mileage` отсутствует, `null` или `''` → ошибка `mileage`, `ValidationException`; иначе приведение `(int)` и проверка диапазона `0..vehicle.max_mileage_km` (rules.php:23).
- **`AssessmentService::assess()`** (AssessmentService.php:37-39): при решении `approve` и пробеге > `vehicle.review_mileage_km` (rules.php:24) решение понижается до `review`.

В LTV пробег не входит (LtvCalculator.php:15 — только сумма и стоимость). Других проверок пробега в `backend/src/Domain/` и `rules.php` нет.
