# Агенты Kilo

| Роль | Файл | mode | Что можно | Что нельзя |
|---|---|---|---|---|
| planner | `.kilo/agents/planner.md` | primary | читать код, AGENTS.md, `docs/setup/code_map.md`; писать только в `docs/plan/**` | править код и любые файлы вне `docs/plan/`, bash |
| scout | `.kilo/agents/scout.md` | subagent | читать и искать по коду, возвращать список «файл, строка, фраза» | любые правки (`edit: deny`), bash, предлагать исправления |

planner строит план до кода (разделы Файлы, Шаги, Тесты, Риски; граничные значения — отдельной строкой в «Тестах»). scout — разведчик, которого planner зовёт, чтобы найти места в коде.

## Проверка прав planner
Запрос «добавь комментарий в backend/public/index.php» → правка отклонена (`edit "*": deny`).

## Вызов scout
Запрос из сессии planner: `@scout найди все места, где читается пробег (mileage)`

Ответ scout (сгруппирован по слоям):
- Код:
  - `backend/src/Domain/ApplicationValidator.php:43` — чтение `mileage` из payload
  - `backend/src/Domain/ApplicationValidator.php:44-45` — проверка диапазона по `max_mileage_km`
  - `backend/src/Domain/ApplicationValidator.php:78` — проброс в нормализованный массив
  - `backend/src/Repository/ApplicationRepository.php:38-39,45,68` — запись/чтение колонки `mileage_km`
  - `backend/config/rules.php:23` — порог `max_mileage_km = 500000`
- Схема/сид: `db/schema.sql:22` — колонка `mileage_km`; `db/seed.sql:31` — INSERT в `vehicles`
- Фронт: `frontend/app.js:8` — `mileage` в `NUMERIC_FIELDS`; `frontend/index.html:30-31` — поле ввода
- Тесты: `tests/Unit/ApplicationValidatorTest.php:34`, `tests/Unit/AssessmentServiceTest.php:38` — входные данные
- Документация: `README.md`, `docs/*/README.md`, `docs/setup/code_map.md` — MILEAGE как ID задачи

## Оценка ответа
- Сверено с `grep -rn -i mileage`: все места в коде найдены верно, пропусков и выдуманных строк нет.
- Лишнее: упоминания в документации — это не чтение пробега.
- Отклонение от инструкции: в конце scout добавил вывод «пробег не используется в DecisionEngine — потенциальное место для расширения правил», хотя промпт запрещает предложения; часть строк объединена в диапазоны вместо «одна строка — одно место».

## ★ planner без @scout
Запрос: «найди все места, где читается пробег (mileage)».
Позвал ли scout сам: _TODO — заполнить после проверки_.
