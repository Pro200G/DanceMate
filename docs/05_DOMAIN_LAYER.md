# 04_DOMAIN_LAYER.md

# Domain Layer

**Проект:** DanceMate
**Версия:** 1.0

---

# 1. Назначение

Domain Layer является центральной частью системы и реализует бизнес-модель приложения.

Этот слой:

* не зависит от FastAPI;
* не зависит от PostgreSQL;
* не зависит от SQLAlchemy;
* не зависит от Redis;
* не зависит от внешних API;
* не содержит HTTP-запросов и SQL-запросов.

В Domain Layer размещаются только бизнес-сущности, правила и интерфейсы.

---

# 2. Место Domain Layer в архитектуре

```text
                   +----------------------+
                   |      API Layer       |
                   +----------+-----------+
                              |
                              v
                   +----------------------+
                   |  Application Layer   |
                   +----------+-----------+
                              |
                              v
                   +----------------------+
                   |    Domain Layer      |
                   +----------+-----------+
                              |
                              v
                   +----------------------+
                   | Infrastructure Layer |
                   +----------------------+
```

Domain Layer не зависит ни от одного другого слоя.

Все остальные слои зависят от Domain Layer.

---

# 3. Принципы Domain Layer

Domain Layer реализует:

* Domain-Driven Design (DDD);
* SOLID;
* Clean Architecture;
* Dependency Inversion Principle.

Основные требования:

* отсутствие зависимостей от инфраструктуры;
* неизменяемость бизнес-правил;
* повторное использование бизнес-логики;
* возможность тестирования без базы данных.

---

# 4. Структура каталога

```text
app/
└── domain/
    ├── entities/
    ├── value_objects/
    ├── repositories/
    ├── services/
    ├── events/
    ├── specifications/
    ├── exceptions/
    ├── policies/
    ├── enums/
    └── shared/
```

---

# 5. Entities

Каталог содержит основные бизнес-сущности.

```text
entities/
├── account.py
├── user.py
├── profile.py
├── invitation.py
├── meeting.py
├── event.py
├── squad.py
├── conversation.py
├── message.py
├── rating.py
├── gratitude.py
├── notification.py
├── payment.py
├── subscription.py
└── media.py
```

Каждая Entity:

* имеет собственный идентификатор;
* содержит бизнес-методы;
* контролирует своё состояние;
* не знает о способе хранения данных.

---

# 6. Value Objects

Value Object представляет неизменяемый объект без собственной идентичности.

```text
value_objects/
├── phone.py
├── email.py
├── full_name.py
├── geo_location.py
├── dance_level.py
├── age_range.py
├── money.py
└── schedule.py
```

Основные свойства:

* неизменяемость (immutable);
* сравнение по значению;
* отсутствие собственного идентификатора.

---

# 7. Repositories

Domain Layer содержит только интерфейсы репозиториев.

```text
repositories/
├── account_repository.py
├── user_repository.py
├── profile_repository.py
├── event_repository.py
├── invitation_repository.py
├── meeting_repository.py
├── chat_repository.py
├── payment_repository.py
└── notification_repository.py
```

Реализация находится в Infrastructure Layer.

---

# 8. Domain Services

Domain Service используется, если бизнес-операция не принадлежит одной сущности.

```text
services/
├── matching_service.py
├── recommendation_service.py
├── invitation_service.py
├── meeting_service.py
├── subscription_service.py
├── payment_service.py
└── moderation_service.py
```

---

# 9. Domain Events

Domain Events фиксируют произошедшие бизнес-события.

```text
events/
├── account_created.py
├── profile_created.py
├── invitation_sent.py
├── invitation_accepted.py
├── meeting_created.py
├── event_created.py
├── payment_completed.py
└── subscription_activated.py
```

---

# 10. Specifications

Specification описывает сложные бизнес-условия.

```text
specifications/
├── can_create_event.py
├── can_send_invitation.py
├── can_join_meeting.py
├── profile_is_complete.py
└── subscription_is_active.py
```

Specifications позволяют инкапсулировать правила проверки без дублирования кода.

---

# 11. Policies

Policies описывают бизнес-политики приложения.

```text
policies/
├── privacy_policy.py
├── moderation_policy.py
├── payment_policy.py
├── recommendation_policy.py
└── notification_policy.py
```

---

# 12. Exceptions

Domain Layer использует только собственные исключения.

```text
exceptions/
├── domain_exception.py
├── validation_exception.py
├── profile_not_found.py
├── invitation_expired.py
├── payment_failed.py
└── subscription_required.py
```

---

# 13. Enums

Все перечисления располагаются централизованно.

```text
enums/
├── gender.py
├── dance_style.py
├── invitation_status.py
├── meeting_status.py
├── payment_status.py
├── notification_type.py
└── subscription_type.py
```

---

# 14. Shared

Общие базовые классы Domain Layer.

```text
shared/
├── aggregate_root.py
├── entity.py
├── value_object.py
├── domain_event.py
└── repository.py
```

---

# 15. Правила разработки

В Domain Layer запрещено:

* использовать SQLAlchemy;
* использовать FastAPI;
* выполнять SQL-запросы;
* использовать HTTP-клиенты;
* импортировать инфраструктурные зависимости.

Допускается использование только стандартной библиотеки Python и внутренних компонентов Domain Layer.

---

# 16. Взаимодействие со слоями

```text
API Layer
        │
        ▼
Application Layer
        │
        ▼
Domain Layer
        │
        ▼
Infrastructure Layer
```

Domain Layer определяет интерфейсы, которые реализуются в Infrastructure Layer.

---

# 17. Связь с базой данных

Domain Layer не знает о структуре PostgreSQL.

Преобразование между Domain Entity и ORM Model выполняется через Mapper в Infrastructure Layer.

---

# 18. Покрытие тестами

Все сущности Domain Layer должны иметь модульные тесты.

Минимальное покрытие:

* создание сущностей;
* бизнес-методы;
* проверки инвариантов;
* генерация Domain Events;
* обработка исключений.