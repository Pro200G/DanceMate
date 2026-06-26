# PROJECT_STRUCTURE.md

> **Проект:** DanceMate
> **Тип проекта:** Кроссплатформенное мобильное приложение для поиска танцевальных партнеров
> **Архитектура:** Clean Architecture + Domain-Driven Design (DDD) + SOLID + Repository Pattern + Unit of Work + CQRS (частично) + Event-Driven Components
> **Backend:** Python 3.13 + FastAPI + SQLAlchemy 2.x + PostgreSQL 17 + Redis + Celery + WebSocket
> **Frontend:** Flutter (Android / iOS)
> **Версия документа:** 1.0

---

# 1. Назначение документа

Документ описывает архитектуру проекта DanceMate.

Он определяет:

* структуру каталогов проекта;
* ответственность каждого слоя;
* зависимости между слоями;
* правила разработки новых модулей;
* принципы организации кода;
* правила масштабирования проекта.

Настоящий документ является основным архитектурным документом проекта.

Все остальные документы являются его дополнением.

---

# 2. Архитектурные принципы

Проект строится на следующих принципах.

## SOLID

Каждый компонент имеет единственную ответственность.

Модули открыты для расширения и закрыты для изменения.

Все зависимости строятся через абстракции.

Интерфейсы разделяются по ответственности.

Наследники полностью взаимозаменяемы.

---

## Clean Architecture

Зависимости всегда направлены внутрь системы.

```text
Presentation

↓

Application (Use Cases)

↓

Domain

↓

Infrastructure
```

Ни один слой не знает о реализации внешнего слоя.

Например:

Domain не знает о PostgreSQL.

Domain не знает о FastAPI.

Domain не знает о Redis.

---

## Domain Driven Design (DDD)

Каждый бизнес-модуль выделяется в отдельный домен.

Основные домены проекта:

* Пользователи
* Профили
* Танцевальные стили
* Поиск партнеров
* Приглашения
* События
* Групповые сборы
* Чаты
* Рейтинги
* Благодарности
* Подписки
* Платежи
* Уведомления
* Модерация

Каждый домен развивается независимо.

---

## Repository Pattern

Работа с базой данных производится исключительно через Repository.

Use Case никогда не работает напрямую с SQLAlchemy.

---

## Unit Of Work

Любая операция записи производится внутри Unit Of Work.

Например:

Регистрация пользователя

↓

Создание пользователя

↓

Создание профиля

↓

Создание настроек

↓

Commit

---

## CQRS (частично)

Команды отделяются от запросов.

Пример:

SearchPartnerQuery

не изменяет БД.

CreateInvitationCommand

изменяет БД.

---

# 3. Общая структура проекта

```text
DanceMate/

├── backend/
├── frontend/
├── docs/
├── infrastructure/
├── .github/
├── docker-compose.yml
├── LICENSE
├── README.md
└── CHANGELOG.md
```

---

# 4. Назначение каталогов

## backend/

Исходный код серверной части.

Содержит всю бизнес-логику приложения.

Подробное описание:

→ 02_BACKEND_STRUCTURE.md

---

## frontend/

Исходный код Flutter-приложения.

Подробное описание:

→ 08_FRONTEND_STRUCTURE.md

---

## docs/

Полная архитектурная документация проекта.

---

## infrastructure/

Инфраструктурные файлы.

Например:

* Docker
* Kubernetes
* Nginx
* CI/CD
* Terraform (при необходимости)

---

## .github/

Автоматизация GitHub Actions.

Содержит:

* сборку;
* тестирование;
* публикацию Docker-образов;
* деплой.

---

# 5. Архитектура зависимостей

Разрешенные зависимости:

```text
API

↓

Use Cases

↓

Domain

↓

Interfaces

↓

Infrastructure

↓

Database
```

Обратные зависимости запрещены.

Например:

Infrastructure

НЕ может обращаться

к Use Cases.

---

# 6. Основные слои системы

## Presentation Layer

Отвечает только за взаимодействие с клиентом.

Содержит:

* REST API;
* WebSocket;
* DTO;
* сериализацию;
* валидацию.

---

## Application Layer

Содержит сценарии использования системы.

Каждый Use Case решает только одну задачу.

Например:

RegisterUser

SearchPartner

CreateInvitation

AcceptInvitation

LeaveRating

CreateSquad

SendGratitude

---

## Domain Layer

Самая важная часть проекта.

Не зависит ни от одного фреймворка.

Содержит:

* Entities;
* Value Objects;
* Domain Events;
* Domain Services;
* Domain Exceptions;
* бизнес-правила.

---

## Infrastructure Layer

Содержит реализации интерфейсов.

Например:

PostgreSQL Repository

Redis Cache

SMS Provider

S3 Storage

Push Notifications

---

## Persistence Layer

Хранение данных.

Используются:

* PostgreSQL;
* Alembic;
* SQL Schema;
* SQL Functions;
* Triggers;
* Views.

---

# 7. Основные домены проекта

Проект разделен на независимые бизнес-домены.

* Authentication
* User
* Profile
* Dance
* Availability
* Matching
* Invitation
* Meeting
* Event
* Squad
* Chat
* Rating
* Gratitude
* Moderation
* Notification
* Media
* Payment
* Subscription
* Analytics
* Administration

Каждый домен имеет собственные:

* Entities;
* Repository;
* Use Cases;
* Services;
* API;
* Tests.

---

# 8. Правила разработки

При добавлении новой функции разработчик обязан:

1. Создать Entity (при необходимости).
2. Создать Repository Interface.
3. Создать Repository Implementation.
4. Создать DTO.
5. Создать Use Case.
6. Создать API Endpoint.
7. Добавить Unit Tests.
8. Добавить Integration Tests.
9. Обновить документацию.

Новая бизнес-логика не должна размещаться в API или ORM-моделях.

---

# 9. Стандарты качества

Проект должен соответствовать:

* SOLID
* Clean Architecture
* Domain Driven Design
* PEP 8
* Black
* Ruff
* MyPy
* Conventional Commits
* Semantic Versioning

---

# 10. Структура архитектурной документации

Документация проекта разделена на специализированные документы:

01_ROOT_STRUCTURE.md

Описание корневой структуры проекта.

02_BACKEND_STRUCTURE.md

Полная структура Backend.

03_DATABASE_STRUCTURE.md

Архитектура PostgreSQL и SQL-проекта.

04_DOMAIN_LAYER.md

Описание Domain Layer.

05_USE_CASES.md

Архитектура Use Cases.

06_INFRASTRUCTURE.md

Описание Infrastructure Layer.

07_API.md

REST API и WebSocket.

08_FRONTEND_STRUCTURE.md

Архитектура Flutter-приложения.

09_DEPENDENCIES.md

Диаграммы зависимостей между слоями.

10_SOLID_RULES.md

Правила разработки согласно SOLID.

11_NAMING.md

Правила именования файлов, классов и модулей.

12_TESTING.md

Архитектура тестирования.

13_DEPLOYMENT.md

Docker, CI/CD, Production Deployment.