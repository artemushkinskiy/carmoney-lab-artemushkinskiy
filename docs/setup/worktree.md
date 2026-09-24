artemushkinskiy@MacBook-Pro-Artem carmoney-lab-artemushkinskiy % git worktree list
/Users/artemushkinskiy/Work/carmoney-lab-artemushkinskiy                                    d72dc02 [d1/1.2.1-1.2.3-artemushkinskiy]
/Users/artemushkinskiy/Work/carmoney-lab-artemushkinskiy/.kilo/worktrees/far-hope           d72dc02 [far-hope]
/Users/artemushkinskiy/Work/carmoney-lab-artemushkinskiy/.kilo/worktrees/highfalutin-cocoa  d72dc02 (detached HEAD)
artemushkinskiy@MacBook-Pro-Artem carmoney-lab-artemushkinskiy %

LtvCalculatorTest.php — проверяет расчёт LTV в процентах на наборах данных и выброс InvalidArgumentException при нулевой стоимости или неположительной сумме.
AssessmentServiceTest.php — проверяет сквозную оценку заявки: низкий LTV → approve с лимитом равным запрошенной сумме, средний → review с нулевым лимитом, высокий → reject.
DecisionEngineTest.php — проверяет выбор решения (approve/review/reject) по порогам LTV, включая граничные значения серой зоны.
VinValidatorTest.php — проверяет формат VIN: 17 символов, регистр, запрещённые буквы I/O/Q, спецсимволы, пустая строка.
ApplicationValidatorTest.php — проверяет валидацию заявки: нормализация VIN, отклонение года из будущего и суммы ниже минимума, сбор всех ошибок разом.
Работаю в папке /Users/artemushkinskiy/Work/carmoney-lab-artemushkinskiy/.kilo/worktrees/far-hope, ветка far-hope.
