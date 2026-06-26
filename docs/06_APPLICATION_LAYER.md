# APPLICATION_LAYER.md

# Application Layer

**Проект:** DanceMate
**Версия:** 1.0

---

# 1. Назначение

Application Layer реализует прикладную логику системы.

Основные задачи:

* выполнение бизнес-сценариев (Use Cases);
* координация работы Domain Layer;
* управление транзакциями;
* преобразование DTO;
* взаимодействие с репозиториями через интерфейсы;
* публикация Domain Events;
* интеграция с внешними сервисами через абстракции.

Application Layer **не содержит HTTP-код, SQL-запросы и ORM-модели**.

---

# 2. Место Application Layer в архитектуре

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

Application Layer зависит только от Domain Layer и объявленных интерфейсов.

---

# 3. Структура каталога

```text
app/
└── application/
    ├── commands/
    ├── command_handlers/
    ├── queries/
    ├── query_handlers/
    ├── dto/
    ├── use_cases/
    ├── interfaces/
    ├── services/
    ├── validators/
    ├── mappers/
    ├── transactions/
    ├── event_handlers/
    └── exceptions/
```

---

# 4. Use Cases

Use Case описывает один завершённый пользовательский сценарий.

Каждый Use Case:

* решает одну бизнес-задачу;
* имеет один вход;
* имеет один результат;
* не знает о FastAPI;
* использует интерфейсы репозиториев.

Структура:

```text
use_cases/
├── auth/
├── profiles/
├── invitations/
├── meetings/
├── events/
├── chats/
├── ratings/
├── notifications/
├── subscriptions/
├── payments/
└── moderation/
```

Примеры:

```text
CreateProfileUseCase
UpdateProfileUseCase
SendInvitationUseCase
AcceptInvitationUseCase
CreateMeetingUseCase
JoinEventUseCase
SendMessageUseCase
PurchaseSubscriptionUseCase
```

---

# 5. Commands (CQRS)

Команды изменяют состояние системы.

```text
commands/
├── create_profile.py
├── update_profile.py
├── delete_profile.py
├── send_invitation.py
├── accept_invitation.py
├── create_meeting.py
├── send_message.py
└── purchase_subscription.py
```

Команда содержит только входные данные.

---

# 6. Command Handlers

Каждая команда имеет отдельный обработчик.

```text
command_handlers/
├── create_profile_handler.py
├── update_profile_handler.py
├── send_invitation_handler.py
├── create_meeting_handler.py
└── purchase_subscription_handler.py
```

Handler:

* получает Command;
* вызывает Domain Layer;
* сохраняет изменения;
* публикует события.

---

# 7. Queries (CQRS)

Запросы не изменяют состояние системы.

```text
queries/
├── get_profile.py
├── search_profiles.py
├── get_events.py
├── get_messages.py
├── get_notifications.py
└── get_subscription.py
```

---

# 8. Query Handlers

```text
query_handlers/
├── get_profile_handler.py
├── search_profiles_handler.py
├── get_events_handler.py
├── get_messages_handler.py
└── get_notifications_handler.py
```

Для чтения допускается использование оптимизированных представлений или Materialized Views.

---

# 9. DTO

DTO используются для обмена данными между слоями.

```text
dto/
├── auth/
├── profile/
├── invitation/
├── meeting/
├── event/
├── chat/
├── payment/
└── notification/
```

Типы DTO:

* Create
* Update
* Response
* Filter
* Search
* Pagination

---

# 10. Интерфейсы

Application Layer использует только абстракции.

```text
interfaces/
├── sms_service.py
├── email_service.py
├── payment_gateway.py
├── storage_service.py
├── notification_service.py
├── event_bus.py
├── cache_service.py
└── unit_of_work.py
```

Реализация располагается в Infrastructure Layer.

---

# 11. Сервисы приложения

Application Services координируют выполнение нескольких Use Cases.

```text
services/
├── matching_service.py
├── recommendation_service.py
├── onboarding_service.py
├── moderation_service.py
└── analytics_service.py
```

---

# 12. Validators

Application Validators проверяют входные данные перед передачей в Domain Layer.

```text
validators/
├── profile_validator.py
├── invitation_validator.py
├── payment_validator.py
└── meeting_validator.py
```

Бизнес-инварианты остаются в Domain Layer.

---

# 13. Mappers

Преобразуют DTO в Domain Entity и обратно.

```text
mappers/
├── profile_mapper.py
├── invitation_mapper.py
├── meeting_mapper.py
└── payment_mapper.py
```

---

# 14. Transactions

Все операции изменения данных выполняются через Unit of Work.

```text
transactions/
└── unit_of_work.py
```

Пример одной транзакции:

* создание встречи;
* создание группового чата;
* добавление участников;
* создание уведомлений.

При ошибке откатывается вся операция.

---

# 15. Event Handlers

Обрабатывают события Domain Layer.

```text
event_handlers/
├── account_created_handler.py
├── invitation_sent_handler.py
├── meeting_created_handler.py
├── payment_completed_handler.py
└── subscription_activated_handler.py
```

---

# 16. Исключения

```text
exceptions/
├── application_exception.py
├── validation_failed.py
├── access_denied.py
├── conflict_exception.py
└── operation_failed.py
```

---

# 17. Правила разработки

Application Layer запрещено:

* выполнять SQL-запросы;
* использовать SQLAlchemy напрямую;
* обращаться к FastAPI Request/Response;
* работать с HTTP-клиентами;
* содержать бизнес-правила Domain Layer.

Application Layer только организует выполнение сценариев.

---

# 18. Взаимодействие со слоями

```text
API
 │
 ▼
DTO
 │
 ▼
Use Case
 │
 ▼
Repository Interface
 │
 ▼
Domain Entity
 │
 ▼
Domain Event
 │
 ▼
Infrastructure
```

---

# 19. Покрытие тестами

Для каждого Use Case должны быть тесты:

* успешное выполнение;
* ошибки валидации;
* ошибки доступа;
* откат транзакции;
* публикация Domain Events.

---
