# 01_PROJECT_STRUCTURE.md

# Project Structure Overview

**Проект:** DanceMate
**Тип:** Кроссплатформенная социальная платформа для танцевального сообщества
**Архитектура:** Clean Architecture + DDD + SOLID + Repository + Unit of Work + CQRS (частично) + Event-driven ready
**Backend:** Python 3.13 + FastAPI + SQLAlchemy 2.x + PostgreSQL 17 + Redis + Celery + WebSocket
**Frontend:** Flutter (iOS / Android)
**Документация:** 18 архитектурных документов

---

# 1. Назначение документа

Этот документ является **точкой входа в архитектуру проекта**.

Он определяет:

* общую структуру репозитория;
* назначение всех каталогов;
* связи между частями системы;
* навигацию по архитектурной документации;
* правила масштабирования проекта.

---

# 2. Общая структура репозитория

```text
DanceMate/
│
├── backend/
├── frontend/
├── database/
├── infrastructure/
├── monitoring/
├── tests/
├── scripts/
├── docker/
│
├── docs/
│   ├── 01_PROJECT_STRUCTURE.md
│   ├── 02_SYSTEM_ARCHITECTURE.md
│   ├── 03_DATABASE_STRUCTURE.md
│   ├── 04_PHYSICAL_DATA_MODEL.md
│   ├── 05_DOMAIN_LAYER.md
│   ├── 06_APPLICATION_LAYER.md
│   ├── 07_INFRASTRUCTURE_LAYER.md
│   ├── 08_API_LAYER.md
│   ├── 09_SECURITY_ARCHITECTURE.md
│   ├── 10_DEPLOYMENT_ARCHITECTURE.md
│   ├── 11_TESTING_STRATEGY.md
│   ├── 12_CODING_STANDARDS.md
│   ├── 13_DEPENDENCIES.md
│   ├── 14_GIT_WORKFLOW.md
│   ├── 15_CI_CD_PIPELINE.md
│   ├── 16_MONITORING.md
│   ├── 17_PERFORMANCE.md
│   ├── 18_PROJECT_ROADMAP.md
│
├── docker-compose.yml
├── pyproject.toml
├── requirements.txt
├── .env.example
├── README.md
└── CHANGELOG.md
```

---

# 3. Назначение каталогов

## backend/

Backend-приложение (FastAPI).

Содержит:

* API Layer
* Application Layer
* Domain Layer
* Infrastructure Layer

---

## frontend/

Flutter приложение:

* UI
* state management
* API client

---

## database/

Полный SQL-проект PostgreSQL:

* schema/
* migrations (Alembic)
* seed/
* diagrams/
* docs/

---

## infrastructure/

Инфраструктура:

* Nginx
* Docker configs
* Kubernetes manifests (future)
* Terraform (future)

---

## monitoring/

Observability stack:

* Prometheus
* Grafana
* Loki
* Alertmanager
* OpenTelemetry

---

## tests/

Все уровни тестирования:

* unit
* integration
* e2e
* load tests

---

## scripts/

DevOps и utility скрипты:

* database init
* backup
* deployment helpers
* migration tools

---

## docker/

Docker конфигурации:

* backend container
* worker container
* database
* redis
* minio

---

# 4. Архитектурная модель

Проект строго следует:

* Clean Architecture
* Domain Driven Design (DDD)
* SOLID
* Repository Pattern
* Unit of Work
* CQRS (частично)

---

# 5. Основные слои backend

```text
API Layer
    ↓
Application Layer
    ↓
Domain Layer
    ↓
Infrastructure Layer
```

---

## API Layer

* REST API
* WebSocket
* DTO
* authentication middleware

---

## Application Layer

* Use Cases
* orchestration
* transactions
* workflow logic

---

## Domain Layer

* Entities
* Value Objects
* Domain Services
* Domain Events
* business rules

---

## Infrastructure Layer

* PostgreSQL (SQLAlchemy)
* Redis
* MinIO
* external APIs
* message brokers

---

# 6. Доменные области

Проект разделён на bounded contexts:

* User
* Profile
* Dance
* Matching
* Invitation
* Event
* Meeting
* Squad
* Chat
* Rating
* Gratitude
* Notification
* Media
* Payment
* Subscription
* Moderation
* Analytics
* Administration

---

# 7. Архитектурные ограничения

## Запрещено:

* бизнес-логика в API Layer
* SQL в Use Cases
* зависимости Domain → Infrastructure
* прямой доступ к ORM из Domain

---

# 8. Принципы проектирования

* Single Responsibility
* Open/Closed
* Liskov Substitution
* Interface Segregation
* Dependency Inversion

---

# 9. Работа с данными

* PostgreSQL — основная БД
* Redis — кеш и pub/sub
* MinIO — медиа-хранилище
* Alembic — миграции

---

# 10. Асинхронная архитектура

Используется:

* async/await
* WebSocket
* Celery workers
* event-driven patterns (future-ready)

---

# 11. Масштабирование

Поддерживается:

* горизонтальное масштабирование backend
* stateless API
* distributed workers
* read replicas PostgreSQL
* Redis cluster (future)

---

# 12. Наблюдаемость

Интеграция:

* logs (structured JSON)
* metrics (Prometheus)
* tracing (OpenTelemetry)
* dashboards (Grafana)

---

# 13. CI/CD

Полностью автоматизирован:

* GitHub Actions
* Docker build
* testing pipeline
* deployment pipeline

---

# 14. Безопасность

* JWT auth
* RBAC
* rate limiting
* input validation
* audit logs

---

# 15. Документация архитектуры

Полная документация:

* 01_PROJECT_STRUCTURE.md
* 02_SYSTEM_ARCHITECTURE.md
* 03_DATABASE_STRUCTURE.md
* 04_PHYSICAL_DATA_MODEL.md
* 05_DOMAIN_LAYER.md
* 06_APPLICATION_LAYER.md
* 07_INFRASTRUCTURE_LAYER.md
* 08_API_LAYER.md
* 09_SECURITY_ARCHITECTURE.md
* 10_DEPLOYMENT_ARCHITECTURE.md
* 11_TESTING_STRATEGY.md
* 12_CODING_STANDARDS.md
* 13_DEPENDENCIES.md
* 14_GIT_WORKFLOW.md
* 15_CI_CD_PIPELINE.md
* 16_MONITORING.md
* 17_PERFORMANCE.md
* 18_PROJECT_ROADMAP.md

---

# 16. Принцип обновления документации

Любое изменение в коде требует:

* обновления соответствующего документа
* проверки согласованности слоёв
* обновления диаграмм при необходимости

---

# 17. Итог

Этот документ является **входной точкой всей архитектуры DanceMate**.

Он связывает:

* код
* базу данных
* инфраструктуру
* CI/CD
* мониторинг
* продуктовую стратегию
