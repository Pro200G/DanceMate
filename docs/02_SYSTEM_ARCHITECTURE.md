# 02_SYSTEM_ARCHITECTURE.md

# System Architecture

**Проект:** DanceMate
**Версия:** 1.0

---

# 1. Назначение

Документ описывает общую системную архитектуру DanceMate.

Он является центральным архитектурным ядром проекта и определяет:

* структуру системы;
* взаимодействие компонентов;
* границы доменов;
* потоки данных;
* интеграции;
* принципы масштабирования.

---

# 2. Общая архитектура системы

```text id="arch1"
                    ┌──────────────┐
                    │   Clients     │
                    │ Web / Mobile │
                    └──────┬───────┘
                           │ HTTPS / WebSocket
                           ▼
                    ┌──────────────┐
                    │   API Layer   │
                    │ FastAPI App   │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Application   │  │   Domain      │  │ Infrastructure│
│ Layer         │  │   Layer       │  │ Layer         │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                  │
       ▼                 ▼                  ▼
  Use Cases        Entities &        DB / Redis /
                   Business Logic     External APIs
                           │
                           ▼
                    ┌──────────────┐
                    │ PostgreSQL   │
                    │ Redis        │
                    │ MinIO        │
                    └──────────────┘
```

---

# 3. Архитектурный стиль

Система построена на:

* Clean Architecture
* Domain-Driven Design (DDD)
* Modular Monolith (на первом этапе)
* Event-driven ready architecture

---

# 4. Основные слои системы

## 4.1 API Layer

Отвечает за:

* HTTP REST API
* WebSocket
* сериализацию/десериализацию
* авторизацию
* валидацию входных данных

**Запрещено:**

* бизнес-логика
* SQL
* прямой доступ к инфраструктуре

---

## 4.2 Application Layer

Отвечает за:

* сценарии использования (Use Cases)
* оркестрацию доменных операций
* транзакции (Unit of Work)
* координацию репозиториев

**Не содержит:**

* бизнес-правил (они в Domain)

---

## 4.3 Domain Layer

Ядро системы.

Содержит:

* Entities
* Value Objects
* Domain Services
* Domain Events
* Aggregates

**Не зависит ни от одного слоя**

---

## 4.4 Infrastructure Layer

Отвечает за:

* PostgreSQL (SQLAlchemy)
* Redis
* MinIO
* внешние API
* email/SMS
* очереди задач

---

# 5. Доменные модули (Bounded Contexts)

Система разделена на домены:

## 5.1 User Context

* пользователи
* регистрация
* авторизация
* роли

---

## 5.2 Profile Context

* профили
* навыки
* танцевальные стили
* медиа

---

## 5.3 Event Context

* события
* встречи
* расписания
* участники

---

## 5.4 Social Context

* чаты
* сообщения
* WebSocket
* приглашения

---

## 5.5 Rating Context

* рейтинги
* благодарности
* репутация

---

## 5.6 Squad Context

* групповые активности
* объединения пользователей

---

## 5.7 Payment Context

* подписки
* платежи
* тарифы

---

# 6. Потоки данных

## 6.1 HTTP Request Flow

```text id="flow1"
Client
  ↓
API Layer
  ↓
Application Layer
  ↓
Domain Layer
  ↓
Infrastructure Layer
  ↓
Database / Cache / External APIs
```

---

## 6.2 WebSocket Flow

```text id="flow2"
Client
  ↓
WebSocket Gateway
  ↓
Message Handler (Application Layer)
  ↓
Domain Events
  ↓
Redis Pub/Sub (optional)
  ↓
Other Clients
```

---

## 6.3 Event Flow (future-ready)

```text id="flow3"
Domain Event
   ↓
Event Bus
   ↓
Workers
   ↓
Side Effects (Email, Notifications, Analytics)
```

---

# 7. Data Architecture

## 7.1 PostgreSQL

Основное хранилище:

* пользователи
* события
* чаты
* рейтинги

---

## 7.2 Redis

Используется для:

* кеша
* сессий
* rate limiting
* pub/sub

---

## 7.3 MinIO

Используется для:

* изображений
* видео
* документов

---

# 8. Принципы взаимодействия слоёв

## Dependency Rule

```text
Domain ← Application ← API
        ↑
Infrastructure (depends on Domain abstractions)
```

---

## Запрещено:

* Domain зависит от Infrastructure
* Application знает SQL
* API содержит бизнес-логику

---

# 9. Коммуникация между модулями

Используются:

* Domain Events
* Use Case calls
* Repository Interfaces
* Event Bus (future)

---

# 10. Масштабирование архитектуры

## Stage 1 — Modular Monolith

* один backend
* единая БД
* разделение по доменам

---

## Stage 2 — Service Extraction Ready

* выделение Chat service
* выделение Event service
* выделение Payment service

---

## Stage 3 — Microservices

* API Gateway
* независимые сервисы
* event-driven communication

---

# 11. Безопасность архитектуры

Встроенные уровни:

* JWT Authentication
* Role-Based Access Control (RBAC)
* Rate Limiting
* Input Validation
* Audit Logging

---

# 12. Производительность архитектуры

Обеспечивается через:

* кеширование (Redis)
* асинхронность (async/await)
* индексы PostgreSQL
* background workers
* минимизацию API payload

---

# 13. Наблюдаемость

Интеграция:

* logging (structured JSON)
* metrics (Prometheus)
* tracing (OpenTelemetry)
* monitoring (Grafana)

---

# 14. Надёжность системы

Принципы:

* stateless API
* горизонтальное масштабирование
* retry policies
* circuit breakers (future)
* graceful degradation

---

# 15. Основные риски архитектуры

| Риск               | Решение               |
| ------------------ | --------------------- |
| рост нагрузки      | масштабирование       |
| сложные запросы    | индексы + кеш         |
| WebSocket нагрузка | отдельные workers     |
| монолит            | modular decomposition |

---

# 16. Связь с другими документами

Этот документ является центральным и связан с:

* `05_DOMAIN_LAYER.md`
* `06_APPLICATION_LAYER.md`
* `07_INFRASTRUCTURE_LAYER.md`
* `08_API.md`
* `16_OBSERVABILITY.md`
* `17_PERFORMANCE.md`
