# Refactoring

Рефакторинг часто только переставляет код, а следующая задача проще не становится. Refactoring сохраняет поведение и меняет код, только если следующая задача станет проще. Новый агент без истории чата сначала оспаривает план, а готовый код проверяет [Loop Code Review](https://github.com/di-sukharev/loop-code-review-skill).

## Установка

Отправьте агенту это сообщение.

```text
Установи скиллы глобально
https://github.com/di-sukharev/refactoring-skill
https://github.com/di-sukharev/loop-code-review-skill
```

## Запуск

Напишите `/refactoring` в чате, где агент писал код. В Codex напишите `$refactoring`. Чтобы улучшить другой код, допишите файл, модуль или сценарий. Для интерфейса можно попросить перенести стили внутрь компонентов.

## Другие скиллы

- [Code Scout](https://github.com/di-sukharev/code-scout-skill) поручает поиск кода дешёвой модели.
- [Orchestration](https://github.com/di-sukharev/orchestration-skill) поручает чтение и написание кода дешёвой модели.
- [Loop Tasks](https://github.com/di-sukharev/loop-tasks-skill) запускает для каждой задачи нового агента с чистым контекстом.
- [Loop Code Review](https://github.com/di-sukharev/loop-code-review-skill) отдаёт код новому ревьюеру без истории чата.

[Инструкция для агента](refactoring/SKILL.md) · [Лицензия MIT](LICENSE)
