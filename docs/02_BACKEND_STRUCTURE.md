# 02_BACKEND_STRUCTURE.md

> **Проект:** DanceMate
> **Документ:** Архитектура Backend
> **Часть 1. Общая архитектура Backend и дерево каталогов верхнего уровня**
> **Версия:** 1.0

---

# 1. Назначение документа

Настоящий документ описывает архитектуру Backend проекта DanceMate.

Документ определяет:

* структуру серверной части;
* разделение ответственности между слоями;
* зависимости между слоями;
* назначение каждого каталога;
* правила масштабирования Backend.

Backend разрабатывается в соответствии с принципами:

* Clean Architecture;
* Domain-Driven Design (DDD);
* SOLID;
* Repository Pattern;
* Unit of Work;
* Dependency Injection;
* CQRS (частично).

---

# 2. Общая схема Backend

Backend представляет собой многослойную архитектуру.

```text
                    ┌──────────────────────┐
                    │     Mobile App       │
                    │     Flutter          │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      FastAPI API     │
                    │ REST + WebSocket     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Application Layer  │
                    │     Use Cases        │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     Domain Layer     │
                    │ Business Rules       │
                    └──────────┬───────────┘
                               │
                ┌──────────────┴──────────────┐
                ▼                             ▼
      ┌──────────────────┐          ┌──────────────────┐
      │ Infrastructure   │          │ Interface Layer  │
      │ PostgreSQL       │          │ Repository       │
      │ Redis            │          │ Provider         │
      │ Celery           │          │ Contracts        │
      │ MinIO            │          └──────────────────┘
      └──────────┬───────┘
                 │
                 ▼
      ┌─────────────────────────────┐
      │ PostgreSQL / Redis / S3      │
      └─────────────────────────────┘
```

---

# 3. Правило зависимостей

Во всей системе действует одно правило:

> **Зависимости всегда направлены внутрь архитектуры.**

Разрешённые зависимости:

```text
Presentation
        │
        ▼
Application
        │
        ▼
Domain
        ▲
        │
Interfaces
        ▲
        │
Infrastructure
```

Запрещено:

* Domain → FastAPI
* Domain → SQLAlchemy
* Domain → PostgreSQL
* Domain → Redis
* Domain → Celery
* Domain → MinIO

Domain ничего не знает о внешнем мире.

---

# 4. Верхний уровень Backend

```text
backend/
│
├── app/
├── database/
├── alembic/
├── docker/
├── scripts/
├── tests/
├── uploads/
├── docs/
│
├── requirements.txt
├── pyproject.toml
├── alembic.ini
├── Dockerfile
├── docker-compose.yml
├── .env
├── .env.example
├── .gitignore
└── README.md
```

---

# 5. Назначение каталогов

## app/

Главный каталог Backend.

Содержит весь исходный код приложения.

Внутри располагаются:

* Domain Layer;
* Application Layer;
* Infrastructure Layer;
* REST API;
* WebSocket;
* ORM;
* DTO;
* Use Cases;
* Repository.

Это единственный каталог, содержащий бизнес-логику.

---

## database/

Полный SQL-проект.

Содержит:

* SQL Schema;
* SQL Functions;
* Triggers;
* Views;
* Seed;
* ER-диаграммы;
* документацию по базе данных.

В каталоге отсутствует Python-код.

---

## alembic/

Миграции PostgreSQL.

Назначение:

* версия схемы базы;
* автоматическое обновление структуры;
* откат миграций;
* генерация новых миграций.

---

## docker/

Docker-конфигурации.

Содержит:

* PostgreSQL;
* Redis;
* MinIO;
* Nginx;
* Production Dockerfiles.

Каталог не содержит бизнес-логики.

---

## scripts/

Служебные сценарии.

Например:

* создание администратора;
* импорт танцевальных направлений;
* резервное копирование;
* генерация тестовых данных;
* очистка временных файлов.

---

## tests/

Полностью изолированные тесты Backend.

Разделяются на:

* Unit Tests;
* Integration Tests;
* API Tests;
* Performance Tests.

Тесты не смешиваются с исходным кодом.

---

## uploads/

Используется только в режиме локальной разработки.

Production использует MinIO или Amazon S3.

Хранит:

* аватары;
* видео;
* временные файлы.

Каталог не попадает в Git.

---

## docs/

Внутренняя документация Backend.

Например:

* Swagger Extensions;
* схемы API;
* UML;
* Sequence Diagram;
* технические заметки.

---

# 6. Назначение основных файлов

## requirements.txt

Список зависимостей Python.

Используется:

* Docker;
* CI/CD;
* локальной разработкой.

---

## pyproject.toml

Главный файл конфигурации Python.

Содержит настройки:

* Black;
* Ruff;
* MyPy;
* Pytest;
* Build System.

---

## Dockerfile

Сборка контейнера Backend.

Используется для Production.

---

## docker-compose.yml

Локальное окружение разработки.

Поднимает:

* Backend;
* PostgreSQL;
* Redis;
* MinIO;
* Nginx.

---

## alembic.ini

Настройки Alembic.

---

## .env

Локальные настройки.

Не попадает в Git.

---

## .env.example

Шаблон конфигурации.

Используется всеми разработчиками.

---

## README.md

Документация Backend.

Содержит:

* запуск проекта;
* команды;
* структура;
* ссылки на архитектуру.

---

# 7. Ответственность каталогов

| Каталог  | Ответственность          |
| -------- | ------------------------ |
| app      | Исходный код Backend     |
| database | SQL-проект PostgreSQL    |
| alembic  | Миграции БД              |
| docker   | Docker-инфраструктура    |
| scripts  | Служебные сценарии       |
| tests    | Автоматизированные тесты |
| uploads  | Локальное хранение медиа |
| docs     | Внутренняя документация  |

---

# 8. Принципы масштабирования

Backend проектируется модульно.

Каждый новый бизнес-домен добавляется без изменения существующей структуры.

Например:

```text
app/

domain/
    achievements/

application/
    achievements/

api/
    achievements/

tests/
    achievements/
```

Расширение осуществляется добавлением новых модулей, а не изменением существующих.

---

# 9. Основные правила Backend

* Один модуль — одна ответственность.
* Один Use Case — один сценарий.
* API не содержит бизнес-логики.
* ORM-модели не содержат бизнес-логики.
* Все зависимости внедряются через Dependency Injection.
* Взаимодействие с базой выполняется только через Repository.
* Use Cases работают через интерфейсы, а не через конкретные реализации.
* Domain не зависит от инфраструктуры.

---

## Часть 2. Каталог `app/`

---

# 1. Назначение каталога

Каталог `app/` содержит весь исполняемый код Backend.

Внутри отсутствуют:

* SQL-файлы;
* Docker-конфигурация;
* миграции;
* документация;
* резервные копии.

Каталог разделён на независимые архитектурные слои согласно Clean Architecture.

---

# 2. Полная структура каталога

```text
app/
│
├── main.py
├── config.py
├── constants.py
├── dependencies.py
│
├── api/
├── application/
├── domain/
├── interfaces/
├── infrastructure/
│
├── shared/
├── security/
├── websocket/
├── workers/
├── middleware/
├── utils/
└── templates/
```

---

# 3. Назначение каждого каталога

| Каталог        | Ответственность                |
| -------------- | ------------------------------ |
| api            | REST API и WebSocket endpoints |
| application    | Use Cases                      |
| domain         | Бизнес-модель                  |
| interfaces     | Контракты                      |
| infrastructure | Реализации интерфейсов         |
| shared         | Общие классы                   |
| security       | Авторизация                    |
| websocket      | WebSocket                      |
| workers        | Celery                         |
| middleware     | Middleware                     |
| utils          | Вспомогательные функции        |
| templates      | Шаблоны сообщений              |

---

# 4. Domain Layer

Domain является центром всей архитектуры.

Он не знает:

* FastAPI;
* PostgreSQL;
* SQLAlchemy;
* Redis;
* Celery;
* MinIO.

Domain знает только бизнес.

---

## Структура Domain

```text
domain/
│
├── user/
├── profile/
├── dance/
├── availability/
├── matching/
├── invitation/
├── meeting/
├── squad/
├── event/
├── chat/
├── rating/
├── gratitude/
├── moderation/
├── notification/
├── payment/
├── subscription/
├── media/
├── analytics/
│
├── common/
└── shared/
```

Каждая папка представляет отдельный **Bounded Context**.

---

## Пример одного домена

```text
domain/
└── user/
    │
    ├── entities/
    ├── value_objects/
    ├── events/
    ├── exceptions/
    ├── services/
    ├── specifications/
    ├── policies/
    ├── factories/
    └── __init__.py
```

---

### entities/

Содержит бизнес-сущности.

Например:

```text
User
UserSettings
```

Entity не является SQLAlchemy Model.

---

### value_objects/

Неизменяемые объекты.

Например:

```text
Phone

Email

Coordinates
```

---

### events/

Доменные события.

Например

```text
UserRegistered

InvitationAccepted

MeetingFinished
```

---

### services/

Доменные сервисы.

Используются только если бизнес-правило нельзя разместить внутри Entity.

---

### specifications/

Бизнес-предикаты.

Например

```text
CanInvitePartnerSpecification

PremiumRequiredSpecification
```

---

### policies/

Политики предметной области.

Например

```text
InvitationPolicy

RatingPolicy
```

---

### factories/

Создание сложных объектов.

Например

```text
UserFactory
```

---

# 5. Application Layer

Application содержит исключительно сценарии использования.

Структура:

```text
application/
│
├── auth/
├── user/
├── profile/
├── dance/
├── matching/
├── invitation/
├── meeting/
├── squad/
├── chat/
├── rating/
├── gratitude/
├── moderation/
├── notification/
├── payment/
├── analytics/
└── shared/
```

---

## Каждый модуль

Например

```text
application/

invitation/

    commands/

    queries/

    handlers/

    dto/

    validators/
```

---

### commands/

Изменяют систему.

Например

```text
CreateInvitationCommand

AcceptInvitationCommand
```

---

### queries/

Не изменяют данные.

Например

```text
FindPartnerQuery

GetProfileQuery
```

---

### handlers/

Реализация Use Cases.

Например

```text
CreateInvitationHandler

SearchPartnerHandler
```

---

### dto/

DTO уровня Application.

---

### validators/

Проверка входных данных.

---

# 6. Interface Layer

Содержит исключительно абстракции.

```text
interfaces/
│
├── repositories/
├── services/
├── cache/
├── storage/
├── messaging/
├── notifications/
└── unit_of_work/
```

---

Пример

```text
repositories/

UserRepository

ChatRepository

MeetingRepository
```

Здесь нет SQLAlchemy.

Только интерфейсы.

---

# 7. Infrastructure Layer

Реализация всех интерфейсов.

```text
infrastructure/
│
├── database/
├── repositories/
├── cache/
├── storage/
├── sms/
├── websocket/
├── notifications/
├── celery/
├── monitoring/
└── logging/
```

---

## database/

```text
database/

connection.py

session.py

base.py
```

---

## repositories/

Реализация Repository.

Например

```text
SqlAlchemyUserRepository

SqlAlchemyChatRepository
```

---

## cache/

Redis.

---

## storage/

MinIO

Amazon S3

---

## sms/

SMS-шлюзы.

---

## notifications/

Push.

Email.

---

# 8. API Layer

```text
api/
│
├── deps.py
├── middleware.py
│
├── v1/
│
│   ├── auth/
│   ├── users/
│   ├── profiles/
│   ├── dances/
│   ├── search/
│   ├── invitations/
│   ├── meetings/
│   ├── squads/
│   ├── ratings/
│   ├── gratitude/
│   ├── notifications/
│   ├── chat/
│   ├── payment/
│   ├── admin/
│   └── websocket/
```

Каждый модуль содержит

```text
router.py

schemas.py

responses.py
```

API не содержит бизнес-логики.

API вызывает только Use Cases.

---

# 9. Shared Layer

```text
shared/
│
├── exceptions/
├── enums/
├── dto/
├── types/
├── pagination/
├── events/
└── validators/
```

Здесь располагается код, используемый несколькими доменами.

---

# 10. Security

```text
security/

jwt.py

oauth.py

passwords.py

permissions.py

sms.py
```

---

# 11. WebSocket

```text
websocket/

manager.py

chat.py

notifications.py

presence.py
```

---

# 12. Workers

```text
workers/

celery.py

notifications.py

cleanup.py

analytics.py

reminders.py
```

---

# 13. Middleware

```text
middleware/

auth.py

logging.py

cors.py

rate_limit.py

exceptions.py
```

---

# 14. Utils

Только универсальные функции без бизнес-логики.

Например:

* работа с датой и временем;
* генерация UUID;
* форматирование;
* математические утилиты.

---

# 15. Основные зависимости

Разрешено:

```text
API
    ↓
Application
    ↓
Interfaces
    ↓
Domain
    ↑
Infrastructure
```

Запрещено:

* Domain → Infrastructure
* Domain → API
* Domain → SQLAlchemy
* Application → FastAPI
* Repository → API

---

# 16. Правило разработки

Каждый новый бизнес-модуль создаётся сразу во всех слоях.

Пример для нового модуля `Achievement`:

```text
domain/
    achievement/

application/
    achievement/

interfaces/
    repositories/
        achievement_repository.py

infrastructure/
    repositories/
        sqlalchemy_achievement_repository.py

api/
    v1/
        achievements/
```

Таким образом архитектура остаётся симметричной, а каждый новый домен интегрируется без изменения существующих модулей.


## Часть 3. Каталог `database/`

---

# 1. Назначение

Каталог `database/` содержит всю архитектуру базы данных проекта DanceMate.

Каталог не содержит:

- Python-код;
- FastAPI;
- SQLAlchemy;
- Alembic Migration Scripts;
- бизнес-логику.

Его задача — хранить полное описание структуры PostgreSQL.

Этот каталог позволяет восстановить БД полностью без запуска Backend.

---

# 2. Общая структура

```text
database/
│
├── schema/
├── seed/
├── functions/
├── procedures/
├── triggers/
├── views/
├── indexes/
├── types/
├── constraints/
├── docs/
├── diagrams/
├── benchmark/
├── backup/
└── README.md
```

---

# 3. Принцип организации

Структура разделяется по ответственности.

Каждая группа объектов PostgreSQL хранится отдельно.

Например

Таблицы

↓

schema/

Функции

↓

functions/

Представления

↓

views/

Триггеры

↓

triggers/

Это позволяет быстро находить любой объект БД.

---

# 4. Каталог schema/

Содержит все таблицы проекта.

schema/
│
├── core/
│   ├── extensions.sql
│   ├── types.sql
│   ├── roles.sql
│   └── permissions.sql
│
├── user/
│   ├── users.sql
│   ├── profiles.sql
│   └── availability.sql
│
├── dance/
│   ├── dance_styles.sql
│   └── user_styles.sql
│
├── event/
├── chat/
├── rating/
├── payment/
├── notification/
└── analytics/

Каждый файл отвечает только за одну логическую область.

---

# 5. Каталог seed/

Начальные данные.

```text
seed/

roles.sql

permissions.sql

dance_styles.sql

countries.sql

languages.sql

system_settings.sql

subscription_plans.sql

admin.sql
```

Содержит исключительно INSERT.

---

# 6. Каталог functions/

PostgreSQL Functions.

```text
functions/

calculate_rating.sql

calculate_distance.sql

search_partner.sql

recalculate_statistics.sql

cleanup_messages.sql

update_profile_score.sql
```

Каждая функция располагается отдельно.

---

# 7. Каталог procedures/

Stored Procedures.

```text
procedures/

archive_events.sql

merge_profiles.sql

night_statistics.sql

cleanup_database.sql
```

Используются для сложных операций.

---

# 8. Каталог triggers/

```text
triggers/

users/

profiles/

chat/

ratings/

events/

subscriptions/
```

Например

```text
triggers/

users/

before_insert.sql

before_update.sql

after_update.sql
```

Один триггер — один файл.

---

# 9. Каталог views/

SQL Views.

```text
views/

active_users.sql

popular_styles.sql

event_statistics.sql

user_rating.sql

dashboard.sql
```

---

# 10. Каталог indexes/

Все индексы.

```text
indexes/

users.sql

profiles.sql

events.sql

chat.sql

notifications.sql
```

Не смешиваются с созданием таблиц.

Это облегчает оптимизацию.

---

# 11. Каталог constraints/

```text
constraints/

users.sql

profiles.sql

events.sql

ratings.sql
```

CHECK

UNIQUE

FOREIGN KEY

EXCLUDE

---

# 12. Каталог types/

Все ENUM и Composite Types.

```text
types/

user_status.sql

event_status.sql

subscription_status.sql

dance_level.sql

gender.sql
```

Используется единая точка хранения типов.

---

# 13. Документация

```text
docs/

tables.md

indexes.md

constraints.md

functions.md

views.md

performance.md

naming.md
```

Здесь находится техническая документация БД.

---

# 14. ER-диаграммы

```text
diagrams/

er.drawio

er.svg

er.png

domain.drawio

relations.md
```

Используется Draw.io.

PNG экспортируется автоматически.

---

# 15. Benchmark

```text
benchmark/

insert.sql

search.sql

chat.sql

statistics.sql
```

Используется для тестирования производительности PostgreSQL.

---

# 16. Backup

```text
backup/

daily/

weekly/

manual/
```

Каталог не хранится в Git.

---

# 17. README

Внутри database/

README.md

Содержит

структуру

порядок запуска

создание БД

импорт

экспорт

восстановление

---

# 18. Alembic

Alembic располагается отдельно.

```text
backend/

alembic/

versions/

env.py

script.py.mako

README.md
```

Alembic является системой миграций.

Он не является частью SQL-проекта.

---

# 19. Правила разработки SQL

Каждый SQL-файл отвечает только за один объект.

Запрещено

Один файл

↓

20 таблиц.

Правильно

Каждая таблица

↓

свой SQL.

---

# 20. Правила именования

Все файлы используют snake_case.

Например

```text
create_user.sql

calculate_rating.sql

active_users.sql
```

---

# 21. Правила зависимостей

Порядок создания объектов

```text
Extensions

↓

Types

↓

Tables

↓

Indexes

↓

Constraints

↓

Functions

↓

Procedures

↓

Views

↓

Triggers

↓

Seed
```

Такой порядок гарантирует корректную установку базы.

---

# 22. Интеграция с SQLAlchemy

SQLAlchemy Models

НЕ являются

источником истины.

Источник истины —

SQL-проект.

ORM полностью повторяет структуру PostgreSQL.

---

# 23. Интеграция с Alembic

Разработка выполняется по следующему процессу:

1. Изменяется SQL-файл в `database/schema/`.
2. Обновляются связанные объекты (`functions`, `views`, `constraints` и т.д.), если требуется.
3. Обновляются ORM-модели SQLAlchemy.
4. Генерируется миграция Alembic.
5. Миграция проверяется на "чистой" базе данных.
6. Обновляется документация (`docs/` и, при необходимости, `diagrams/`).

Такой процесс гарантирует, что SQL-проект, ORM и миграции всегда синхронизированы.

---

# 02_BACKEND_STRUCTURE.md

## Часть 4. Инфраструктура Backend, тестирование и конфигурация

---

# 1. Назначение

Данный раздел описывает инфраструктурную часть Backend.

В отличие от каталога `app/`, данные каталоги не содержат бизнес-логики.

Они отвечают за:

- сборку проекта;
- тестирование;
- локальную разработку;
- Docker;
- конфигурацию;
- автоматизацию;
- запуск сервисов;
- обслуживание проекта.

---

# 2. Общая структура

```text
backend/
│
├── docker/
├── scripts/
├── alembic/
│
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── requirements.txt
├── alembic.ini
├── .env.example
├── .gitignore
└── README.md
```

Отдельно в корне проекта располагается

```text
tests/
```

---

# 3. Каталог docker/

Назначение:

Хранение инфраструктуры контейнеризации.

Структура

```text
docker/
│
├── backend/
│   └── Dockerfile.dev
│
├── postgres/
│   ├── init.sql
│   ├── postgresql.conf
│   └── pg_hba.conf
│
├── redis/
│   └── redis.conf
│
├── minio/
│   └── init.sh
│
├── nginx/
│   ├── nginx.conf
│   ├── api.conf
│   └── websocket.conf
│
└── monitoring/
    ├── prometheus.yml
    └── grafana/
```

---

## Backend

Содержит Dockerfile для разработки.

Production использует отдельную сборку.

---

## PostgreSQL

Хранит

- настройки PostgreSQL;
- инициализацию;
- оптимизацию.

---

## Redis

Настройки Redis.

---

## MinIO

Создание Bucket.

Создание пользователей.

Политики доступа.

---

## Nginx

Reverse Proxy.

SSL.

WebSocket.

Static.

---

## Monitoring

Prometheus.

Grafana.

Экспортёры.

---

# 4. Каталог scripts/

Содержит служебные сценарии.

```text
scripts/
│
├── database/
├── development/
├── deployment/
├── backup/
├── import/
└── maintenance/
```

---

## database/

```text
create_database.py

drop_database.py

seed_database.py

reset_database.py
```

---

## development/

```text
create_admin.py

generate_demo_data.py

create_fake_users.py

clear_media.py
```

---

## deployment/

```text
deploy_dev.sh

deploy_stage.sh

deploy_prod.sh
```

---

## backup/

```text
backup_database.py

restore_database.py
```

---

## import/

```text
import_styles.py

import_events.py

import_regions.py
```

---

## maintenance/

```text
cleanup_logs.py

cleanup_uploads.py

recalculate_statistics.py
```

---

# 5. Alembic

Структура

```text
alembic/
│
├── versions/
├── env.py
├── script.py.mako
└── README.md
```

---

## versions/

Каждая миграция — отдельный файл.

Пример

```text
20260701_create_users.py

20260703_create_profiles.py

20260710_create_events.py
```

Миграции никогда не редактируются после публикации.

Создаётся новая миграция.

---

# 6. pyproject.toml

Главный конфигурационный файл Python.

Используется для

- Black
- Ruff
- MyPy
- Pytest
- Build System

Все настройки инструментов находятся здесь.

---

# 7. requirements.txt

Содержит зависимости Production.

Например

```text
fastapi

sqlalchemy

alembic

redis

celery

asyncpg

uvicorn
```

---

Дополнительно рекомендуется

```text
requirements/

base.txt

dev.txt

prod.txt

test.txt
```

Тогда

Production

не содержит

pytest

black

ruff

mypy

---

# 8. .env.example

Шаблон конфигурации.

Например

```text
POSTGRES_HOST=

POSTGRES_PORT=

POSTGRES_DB=

POSTGRES_USER=

POSTGRES_PASSWORD=

SECRET_KEY=

JWT_SECRET=

REDIS_URL=

MINIO_ENDPOINT=
```

Настоящий `.env`

никогда

не попадает в Git.

---

# 9. Docker Compose

Используется исключительно

для локальной разработки.

Поднимает

```text
Backend

PostgreSQL

Redis

MinIO

Nginx
```

Production использует Kubernetes либо отдельные compose-файлы.

---

# 10. README

Backend README содержит

структуру

команды

локальный запуск

Docker

миграции

сидирование

тестирование

---

# 11. Каталог tests/

Рекомендуется располагать

в корне проекта.

```text
tests/
│
├── unit/
├── integration/
├── api/
├── websocket/
├── performance/
├── load/
├── e2e/
├── fixtures/
├── factories/
├── mocks/
└── data/
```

---

## unit/

Проверяет

Domain

Application

Use Cases

Value Objects

Полностью без PostgreSQL.

---

## integration/

Проверяет

Repository

Redis

PostgreSQL

Unit Of Work

---

## api/

Проверяет

REST API

FastAPI

Authentication

Permissions

---

## websocket/

Проверяет

WebSocket

чат

онлайн

уведомления

---

## performance/

Проверка скорости.

Например

```text
поиск

создание событий

чат

рейтинги
```

---

## load/

Нагрузочные тесты.

Используются

Locust

k6

JMeter

---

## e2e/

Полный пользовательский сценарий.

Например

```text
Регистрация

↓

Создание профиля

↓

Поиск партнера

↓

Приглашение

↓

Чат

↓

Оценка
```

---

## fixtures/

Общие тестовые данные.

---

## factories/

Генераторы объектов.

Например

```text
UserFactory

EventFactory

InvitationFactory
```

---

## mocks/

Mock сервисов

SMS

Email

Push

Storage

---

## data/

Тестовые JSON

CSV

SQL

---

# 12. Логирование

Рекомендуется

```text
logs/

backend.log

celery.log

nginx.log
```

Каталог

не попадает

в Git.

---

# 13. Кэш

Redis

используется для

JWT Blacklist

Rate Limit

Онлайн пользователи

Кэш поиска

Кэш событий

Кэш популярных профилей

---

# 14. Медиафайлы

Локальная разработка

```text
uploads/

avatars/

videos/

events/

temp/
```

Production

использует

MinIO

или

Amazon S3.

---

# 15. Правила разработки

Любой новый сервис должен сопровождаться

Unit Tests

↓

Integration Tests

↓

API Tests

↓

документацией

↓

миграцией (при необходимости)

---

# 16. Правила зависимостей

Разрешено

```text
tests
      │
      ▼
API
      │
      ▼
Application
      │
      ▼
Domain
      ▲
      │
Infrastructure
```

Запрещено

Application

↓

использовать Docker

Domain

↓

использовать PostgreSQL

Repository

↓

использовать FastAPI

---

# 17. Стандарты качества

Перед слиянием в основную ветку автоматически выполняются:

- форматирование (Black);
- линтинг (Ruff);
- проверка типов (MyPy);
- модульные тесты;
- интеграционные тесты;
- проверка миграций Alembic;
- проверка покрытия кода (целевой минимум — 80%).

Все проверки должны успешно завершиться до публикации изменений.