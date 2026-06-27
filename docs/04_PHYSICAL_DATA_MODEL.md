# 04_PHYSICAL_DATA_MODEL.md

# Physical Data Model

**Проект:** DanceMate
**Версия:** 1.0

---

# 1. Назначение

Документ описывает физическую реализацию данных в PostgreSQL для системы DanceMate.

Он определяет:

* реальные таблицы и их структуру;
* индексы;
* ограничения;
* связи на уровне БД;
* типы данных;
* стратегии хранения;
* производительность хранения данных.

---

# 2. Общие принципы проектирования БД

База данных построена на принципах:

* нормализация до 3NF (с контролируемыми исключениями);
* использование UUID как PK;
* обязательные внешние ключи;
* явные ограничения целостности;
* индексация всех критичных запросов;
* минимизация дублирования данных;
* разделение read/write нагрузок (future-ready).

---

# 3. Физическая модель vs логическая модель

| Уровень        | Описание                                 |
| -------------- | ---------------------------------------- |
| Domain Model   | Entities и бизнес-логика                 |
| Logical Model  | ER-модель и связи                        |
| Physical Model | PostgreSQL таблицы, индексы, constraints |

---

# 4. Общие стандарты PostgreSQL

## 4.1 Идентификаторы

Все таблицы используют:

```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
```

---

## 4.2 Временные поля

```sql
created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
deleted_at TIMESTAMP WITH TIME ZONE NULL
```

Soft Delete используется в большинстве таблиц.

---

## 4.3 JSONB

Используется для:

* метаданных;
* гибких настроек;
* расширяемых профилей.

---

## 4.4 ENUM

Используется для:

* ролей;
* статусов событий;
* типов уведомлений;
* типов сообщений.

---

# 5. Основные физические сущности

---

## 5.1 users

Хранит учетные записи пользователей.

### Индексы:

* email (unique)
* phone (unique)
* created_at

---

## 5.2 profiles

Расширенные данные пользователя.

### Связи:

* users (1:1)

### Индексы:

* user_id (unique)
* city
* dance_level

---

## 5.3 dance_styles

Справочник танцевальных стилей.

---

## 5.4 user_dance_styles

M:N связь пользователей и стилей.

### Индексы:

* user_id
* dance_style_id

---

## 5.5 events

События и мероприятия.

### Индексы:

* creator_id
* date
* location
* status

---

## 5.6 event_members

Участники событий.

### Индексы:

* event_id
* user_id

---

## 5.7 chats

Диалоги пользователей.

### Индексы:

* type (private/group)
* created_at

---

## 5.8 messages

Сообщения чатов.

### Индексы:

* chat_id
* sender_id
* created_at

### Оптимизация:

* партиционирование по chat_id (future)

---

## 5.9 invitations

Приглашения пользователей.

### Индексы:

* from_user_id
* to_user_id
* status

---

## 5.10 ratings

Рейтинги пользователей.

### Индексы:

* user_id
* from_user_id

---

## 5.11 gratitude

Благодарности пользователей.

---

## 5.12 squads

Групповые объединения.

### Индексы:

* owner_id

---

## 5.13 squad_members

M:N пользователи ↔ squads

---

## 5.14 notifications

Система уведомлений.

### Индексы:

* user_id
* is_read

---

## 5.15 media

Медиафайлы.

### Хранение:

* MinIO (физически)
* PostgreSQL (метаданные)

---

## 5.16 payments

Платежи и подписки.

### Индексы:

* user_id
* status

---

# 6. Связи между таблицами

## Основные связи

```text id="rel1"
users 1 ─── 1 profiles

users 1 ─── N events

users N ─── N dance_styles

users 1 ─── N messages

events 1 ─── N event_members

users 1 ─── N chats

chats 1 ─── N messages

users 1 ─── N invitations
```

---

# 7. Индексация

## 7.1 Основные индексы

* B-tree (по умолчанию)
* Composite indexes (user_id + created_at)
* Partial indexes (is_deleted = false)
* GIN indexes (JSONB)

---

## 7.2 Индексы для поиска

Используются для:

* поиска пользователей;
* поиска мероприятий;
* фильтрации чатов;
* сортировки сообщений.

---

# 8. Производительность хранения

## 8.1 Партиционирование

Планируется для:

* messages (по chat_id или date)
* logs (по дате)
* notifications (по дате)

---

## 8.2 Архивация

Старые данные:

* события
* сообщения
* логи

переносятся в cold storage.

---

# 9. Ограничения целостности

## Foreign Keys

Все связи строго контролируются:

* ON DELETE CASCADE (для зависимых сущностей)
* ON DELETE SET NULL (для логических связей)

---

## Check Constraints

Примеры:

* rating BETWEEN 1 AND 5
* status IN (...)
* amount >= 0

---

# 10. Триггеры

Используются для:

* обновления updated_at;
* логирования изменений;
* синхронизации агрегатов;
* обновления рейтингов.

---

# 11. Представления (Views)

Используются для:

* агрегированных рейтингов;
* статистики пользователей;
* активных событий;
* аналитики.

---

# 12. Функции PostgreSQL

Используются для:

* сложных вычислений;
* оптимизации запросов;
* бизнес-логики на уровне БД (ограниченно).

---

# 13. Безопасность данных

* Row Level Security (future-ready)
* шифрование чувствительных данных
* ограничение доступа через роли PostgreSQL

---

# 14. Работа с миграциями

Все изменения БД выполняются через:

* Alembic migrations
* versioned SQL scripts (schema/)

Запрещено:

* ручное изменение production БД
* изменение старых миграций

---

# 15. Связь с SQL-проектом

Физическая модель соответствует:

```text
database/
├── schema/
├── seed/
├── diagrams/
├── docs/
└── backup/
```

---

# 16. Связь с архитектурой

Документ напрямую связан с:

* `02_SYSTEM_ARCHITECTURE.md`
* `03_DATABASE_STRUCTURE.md`
* `05_DOMAIN_LAYER.md`
* `06_APPLICATION_LAYER.md`

---

# 17. Эволюция модели

## Stage 1

* монолитная БД

## Stage 2

* read replicas
* кеширование

## Stage 3

* partitioning
* event sourcing (частично)

## Stage 4

* распределённые БД (future)