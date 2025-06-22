Домашняя работа N3
==================

## 1. JSON-schema

### 1.1 JSON-schema для функционального события `domain.candidate_testing.task_passed`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Schema события успешного прохождения теста кандидатом",
  "description": "Schema события успешного прохождения теста кандидатом",
  "type": "object",
  "required": [
    "event_id",
    "event_name",
    "event_version",
    "produced_at",
    "payload"
  ],
  "properties": {
    "event_id": {
      "type": "string",
      "format": "uuid",
      "description": "Уникальный идентификатор события"
    },
    "event_name": {
      "type": "string",
      "enum": ["domain.candidate_testing.task_passed"],
      "description": "Название события"
    },
    "event_version": {
      "type": "integer",
      "minimum": 1,
      "description": "Версия события"
    },
    "produced_at": {
      "type": "string",
      "format": "date-time",
      "description": "Время создания события"
    },
    "payload": {
      "type": "object",
      "required": [
        "task_id",
        "candidate_id",
        "passed_at"
      ],
      "properties": {
        "task_id": {
          "type": "string",
          "format": "uuid",
          "description": "Идентификатор теста"
        },
        "candidate_id": {
          "type": "string",
          "format": "uuid",
          "description": "Идентификатор кандидата"
        },
        "passed_at": {
          "type": "string",
          "format": "date-time",
          "description": "Время завершения тестирования"
        }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```

### 1.2 JSON-формального события `data_replication.candidate.created`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Schema события добавления нового кандидата",
  "description": "Schema события добавления нового кандидата",
  "type": "object",
  "required": [
    "event_id",
    "event_name",
    "event_version",
    "produced_at",
    "payload"
  ],
  "properties": {
    "event_id": {
      "type": "string",
      "format": "uuid",
      "description": "Уникальный идентификатор события"
    },
    "event_name": {
      "type": "string",
      "enum": ["domain.candidate_testing.task_passed"],
      "description": "Название события"
    },
    "event_version": {
      "type": "integer",
      "minimum": 1,
      "description": "Версия события"
    },
    "produced_at": {
      "type": "string",
      "format": "date-time",
      "description": "Время создания события"
    },
    "payload": {
      "type": "object",
      "required": [
        "replication_id",
        "login",
        "name",
        "registered_at"
      ],
      "properties": {
        "replication_id": {
          "type": "string",
          "format": "uuid",
          "description": "Уникальный идентификатор кандидата"
        },
        "login": {
          "type": "string",
          "minLength": 1,
          "description": "Логин / username кандидата"
        },
        "name": {
          "type": "string",
          "minLength": 1,
          "description": "ФИО кандидата"
        },
        "registered_at": {
          "type": "string",
          "format": "date-time",
          "description": "Время регистрации"
        }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```


## 2. Процесс миграции связей.

### 2.1 Переход формальной синхронной на асинхронную event-driven на примере перехода связи COMM-090.

1. Добавляем новое событие `data_replication.candidate.created` в schema registry под версией 1.
2. Добавляем консьюмер, который будет сохранять данные о новых учениках в системе управления заданиями.
3. Пишем продюсер в системе тестирования кандидатов, который будет отправлять событие при регистрации кандидатов.
4. Выключаем синхронную коммуникацию
5. Чистим код от старой коммуникации

### 2.2 Переход формальной асинхронной event-driven на синхронную на примере связи COMM-060

Требованиями обусловлена миграция асинхронной связи на синхронную pull связь: при расчёте бонусов нужен максимально
актуальный рейтинг задания.

1. Добавляем новый эндпоинт в систему тестирования кандидатов, который будет отдавать рейтинг задания по его идентификатору:
`/api/v1/task/4ffc48fb-cf86-4b03-9191-9e8379891b6c/rating`
2. Меняем бизнес-логику расчета бонусов на синхронные вызовы в сервис тестирования кандидатов.
3. Когда заработает новая логика, отключаем продюсер со стороны сервиса тестирования.
4. Ждём, пока консьюмеры обработают все данные из брокера.
5. Удаляем код консьюмера из сервиса бонусов.
6. Удаляем таблицу с рейтингом из сервиса бонусов.
7. Удаляем топик из брокера.
8. Чистим оставшееся: метрики, код, таблицы в базе.

### 2.3 Переход функциональной синхронной на асинхронную event-driven на примере связи COMM-040.

Начисление бонусов будет происходить в ответ на событие `domain.candidate_testing.task_passed`
(был синхронный вызов логики начисления бонусов из сервиса тестирования в сервис бонусов).

1. Добавляем новое событие `domain.candidate_testing.task_passed` в schema registry под версией 1.
2. Добавляем в сервис бонусов «пустой» консьюмер без бизнес-логики.
3. Добавляем продьюсер события в сервис тестиования кандидатов.
4. Проверяем, что продюсер продьюсит событие, а косьюмер его обрабатывает.
5. Переносим новую бизнес-логику начисления бонусов в консьюмер, а старую выключаем.
6. Удаляем эндпойнт синхронного начисления бонусов (если его больше никто не использует).


Дальше я не успел доделать.
