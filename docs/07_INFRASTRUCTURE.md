# 06_INFRASTRUCTURE.md

# Infrastructure

**Проект:** DanceMate
**Версия:** 1.0

---

# 1. Назначение

Infrastructure Layer реализует все технические механизмы системы.

Этот слой отвечает за:

* работу с PostgreSQL;
* реализацию репозиториев;
* SQLAlchemy ORM;
* Alembic Migration;
* Redis;
* MinIO (или S3);
* WebSocket;
* Email;
* SMS;
* Push-уведомления;
* платёжные шлюзы;
* очереди сообщений;
* кеширование;
* интеграцию с внешними API.

Infrastructure Layer **реализует интерфейсы**, объявленные в Domain и Application слоях.

---

# 2. Место слоя в архитектуре

```text
                API Layer
                     │
                     ▼
            Application Layer
                     │
                     ▼
              Domain Layer
                     ▲
                     │
          Infrastructure Layer
```

Infrastructure зависит от всех внутренних слоёв, но ни один из них не зависит от Infrastructure.

---

# 3. Структура каталога

```text
app/
└── infrastructure/
    ├── database/
    ├── repositories/
    ├── mappers/
    ├── cache/
    ├── storage/
    ├── websocket/
    ├── notifications/
    ├── sms/
    ├── email/
    ├── payments/
    ├── queue/
    ├── security/
    ├── integrations/
    ├── logging/
    ├── monitoring/
    ├── background/
    └── config/
```

---

# 4. Database

Каталог содержит всё, что связано с PostgreSQL.

```text
database/
├── database.py
├── session.py
├── base.py
├── models/
├── repositories/
├── mappers/
├── converters/
├── unit_of_work.py
└── transaction_manager.py
```

Функции:

* создание соединений;
* управление транзакциями;
* ORM-модели;
* преобразование Domain Entity ↔ ORM.

---

# 5. ORM Models

```text
models/
├── account_model.py
├── user_model.py
├── profile_model.py
├── invitation_model.py
├── meeting_model.py
├── event_model.py
├── chat_model.py
├── message_model.py
├── payment_model.py
└── notification_model.py
```

Каждая ORM-модель соответствует одной таблице PostgreSQL.

---

# 6. Repository Implementations

Реализация интерфейсов репозиториев.

```text
repositories/
├── account_repository.py
├── user_repository.py
├── profile_repository.py
├── invitation_repository.py
├── meeting_repository.py
├── event_repository.py
├── chat_repository.py
├── payment_repository.py
└── notification_repository.py
```

Используют SQLAlchemy и не содержат бизнес-логики.

---

# 7. Database Mappers

Преобразуют ORM-модели в Domain Entity и обратно.

```text
mappers/
├── account_mapper.py
├── user_mapper.py
├── profile_mapper.py
├── meeting_mapper.py
└── payment_mapper.py
```

---

# 8. Cache

Подсистема Redis.

```text
cache/
├── redis.py
├── cache_keys.py
├── cache_service.py
├── profile_cache.py
├── event_cache.py
└── recommendation_cache.py
```

Кэшируются:

* публичные профили;
* результаты поиска;
* рекомендации;
* справочники.

---

# 9. Storage

Работа с файлами.

```text
storage/
├── storage_service.py
├── minio_storage.py
├── s3_storage.py
├── image_processor.py
└── video_processor.py
```

Файлы хранятся вне PostgreSQL.

---

# 10. WebSocket

Реализация обмена сообщениями в реальном времени.

```text
websocket/
├── manager.py
├── connection.py
├── chat_gateway.py
├── event_gateway.py
└── notification_gateway.py
```

---

# 11. Notifications

Подсистема уведомлений.

```text
notifications/
├── notification_service.py
├── push_service.py
├── email_notification.py
├── sms_notification.py
└── websocket_notification.py
```

Поддерживаются:

* Push;
* Email;
* SMS;
* WebSocket.

---

# 12. SMS

```text
sms/
├── sms_service.py
├── twilio_provider.py
├── sms_ru_provider.py
└── mock_provider.py
```

Реализация выбирается через конфигурацию.

---

# 13. Email

```text
email/
├── email_service.py
├── smtp_provider.py
├── sendgrid_provider.py
├── templates/
└── renderer.py
```

---

# 14. Payments

Интеграция с платёжными системами.

```text
payments/
├── payment_service.py
├── stripe_provider.py
├── yookassa_provider.py
├── webhook_handler.py
└── payment_mapper.py
```

Провайдеры взаимозаменяемы благодаря интерфейсам Application Layer.

---

# 15. Queue

Асинхронная обработка задач.

```text
queue/
├── broker.py
├── task_dispatcher.py
├── notification_tasks.py
├── media_tasks.py
└── statistics_tasks.py
```

Очереди используются для тяжёлых операций.

---

# 16. Security

```text
security/
├── jwt_service.py
├── password_hasher.py
├── otp_service.py
├── token_service.py
└── permission_checker.py
```

Реализуются:

* JWT;
* Refresh Token;
* OTP;
* хеширование паролей;
* проверка разрешений.

---

# 17. Integrations

Интеграции со сторонними сервисами.

```text
integrations/
├── maps/
├── analytics/
├── geocoding/
├── social_auth/
└── ai/
```

Все интеграции изолированы от бизнес-логики.

---

# 18. Logging

```text
logging/
├── logger.py
├── audit_logger.py
├── request_logger.py
└── sql_logger.py
```

---

# 19. Monitoring

```text
monitoring/
├── metrics.py
├── healthcheck.py
├── prometheus.py
└── tracing.py
```

Поддерживаются:

* Prometheus;
* OpenTelemetry;
* Health Check.

---

# 20. Background Jobs

Фоновые процессы.

```text
background/
├── cleanup.py
├── refresh_statistics.py
├── expire_invitations.py
└── refresh_materialized_views.py
```

---

# 21. Конфигурация

```text
config/
├── database.py
├── redis.py
├── storage.py
├── email.py
├── sms.py
├── payments.py
└── monitoring.py
```

---

# 22. Правила разработки

Infrastructure Layer разрешается:

* SQLAlchemy;
* Redis;
* HTTP-клиенты;
* SMTP;
* MinIO/S3;
* WebSocket;
* внешние SDK.

Запрещается:

* размещать бизнес-правила;
* изменять Domain Entity;
* нарушать интерфейсы Domain и Application Layer.

---

# 23. Взаимодействие со слоями

```text
Application Layer
        │
        ▼
Repository Interface
        │
        ▼
Infrastructure Repository
        │
        ▼
SQLAlchemy ORM
        │
        ▼
PostgreSQL
```

Все зависимости направлены внутрь архитектуры.

---

# 24. Покрытие тестами

Infrastructure Layer покрывается:

* интеграционными тестами;
* тестами репозиториев;
* тестами внешних интеграций;
* тестами кеширования;
* тестами платёжных шлюзов;
* тестами очередей.