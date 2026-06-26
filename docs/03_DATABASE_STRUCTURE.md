├── Часть 1. Общая архитектура PostgreSQL
├── Часть 2. Доменная модель (DDD)
├── Часть 3. Логическая модель данных
|       ├── Часть 3.1. Master Data Model (полный перечень сущностей)
|       ├── Часть 3.2. Связи между сущностями
|       ├── Часть 3.3. Data Catalog (Каталог таблиц PostgreSQL)
|       ├── Часть 3.4. Enterprise Data Dictionary
├── Часть 4. Физическая модель PostgreSQL
├── Часть 5. Стратегия индексации
├── Часть 6. Ограничения и целостность данных
├── Часть 7. Функции, процедуры и триггеры
├── Часть 8. Представления и Materialized Views
├── Часть 9. Производительность и масштабирование
├── Часть 10. Миграции, версионирование и правила разработки

# 03_DATABASE_STRUCTURE.md

# Часть 1. Общая архитектура PostgreSQL

> Проект: **DanceMate**
> Версия: **1.0**
> СУБД: **PostgreSQL 17**

---

# 1. Назначение документа

Настоящий документ определяет архитектуру базы данных проекта **DanceMate**.

Документ является единственным источником информации о структуре хранения данных.

На его основе разрабатываются:

* SQL Schema;
* SQLAlchemy Models;
* Alembic Migration;
* Repository Layer;
* документация API;
* ER-диаграммы;
* Unit Tests;
* Integration Tests.

---

# 2. Цели архитектуры

Архитектура PostgreSQL должна обеспечивать:

* высокую производительность;
* горизонтальное масштабирование приложения;
* безопасное хранение данных;
* простое сопровождение;
* расширяемость;
* минимизацию дублирования данных;
* соответствие принципам DDD;
* совместимость с Clean Architecture.

---

# 3. Основные принципы проектирования

## 3.1 PostgreSQL — источник истины

Источник истины проекта —

PostgreSQL.

Не SQLAlchemy.

Не FastAPI.

Не ORM.

SQLAlchemy полностью повторяет структуру PostgreSQL.

Изменения всегда начинаются со структуры базы данных.

---

## 3.2 Нормализация

Все таблицы проектируются минимум до **третьей нормальной формы (3NF)**.

Исключения допускаются только при наличии документированного обоснования производительности.

Разрешается контролируемая денормализация только для:

* аналитики;
* поиска;
* Materialized Views;
* агрегированной статистики.

---

## 3.3 UUID

Во всех основных таблицах используется

```sql
UUID PRIMARY KEY
```

Причины:

* отсутствие конфликтов при масштабировании;
* безопасность идентификаторов;
* возможность работы нескольких сервисов;
* удобство репликации.

---

## 3.4 UTC

Все даты сохраняются исключительно в UTC.

Тип:

```sql
TIMESTAMPTZ
```

Преобразование во временную зону пользователя выполняется приложением.

---

## 3.5 Soft Delete

Физическое удаление данных запрещено для большинства бизнес-сущностей.

Используется:

```text
deleted_at

deleted_by
```

или

```text
is_deleted
```

Физически удаляются только:

* временные файлы;
* кеш;
* служебные записи;
* устаревшие логи (по политике хранения).

---

## 3.6 Аудит

Все критические изменения должны журналироваться.

Используется таблица

```text
audit_logs
```

Фиксируются:

* пользователь;
* действие;
* время;
* IP;
* устройство;
* изменённые поля.

---

# 4. Архитектурные уровни базы данных

База данных разделяется на несколько уровней.

```text
Extensions

↓

Types

↓

Core Tables

↓

Reference Tables

↓

Business Tables

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

Materialized Views

↓

Triggers

↓

Seed
```

Каждый уровень зависит только от предыдущих.

---

# 5. Архитектура схем PostgreSQL

Используется одна основная схема.

```text
public
```

При дальнейшем масштабировании допускается разделение:

```text
public

auth

analytics

audit

system
```

Однако на первом этапе проекта используется единая схема.

---

# 6. Основные группы таблиц

Все таблицы делятся на логические группы.

## Core

Базовые таблицы системы.

Например:

* users
* roles
* permissions
* system_settings

---

## Reference

Справочники.

Например:

* dance_styles
* countries
* languages
* regions

---

## Business

Основные данные пользователей.

Например:

* profiles
* invitations
* meetings
* events
* chats
* ratings

---

## Service

Служебные таблицы.

Например:

* notifications
* audit_logs
* background_jobs

---

## Analytics

Таблицы аналитики.

Например:

* statistics_daily
* statistics_monthly
* popularity_cache

---

# 7. Основные агрегаты DDD

База проектируется вокруг агрегатов предметной области.

Каждый агрегат имеет собственного владельца данных.

## User

Корневой агрегат.

Содержит:

* учетную запись;
* телефон;
* настройки;
* роли;
* авторизацию.

---

## Profile

Профиль танцора.

Содержит:

* описание;
* уровень;
* фотографии;
* видео;
* город;
* танцевальные направления.

---

## Invitation

Приглашение на танец.

Включает:

* отправителя;
* получателя;
* статус;
* комментарий;
* время.

---

## Meeting

Факт совместной тренировки.

---

## Chat

Переписка пользователей.

---

## Event

Танцевальное мероприятие.

---

## Squad

Совместная группа.

---

## Rating

Оценки после встречи.

---

## Gratitude

Благодарности.

---

## Subscription

Платные функции.

---

## Payment

Финансовые операции.

---

# 8. Правила хранения данных

Каждая таблица должна содержать стандартный набор технических полей.

```text
id

created_at

updated_at

created_by

updated_by

deleted_at
```

При необходимости

```text
version
```

используется для оптимистичной блокировки.

---

# 9. Правила именования

Все объекты используют

snake_case.

Пример

Таблицы

```text
dance_styles

event_members

meeting_participants
```

Столбцы

```text
created_at

phone_number

profile_photo
```

Индексы

```text
idx_users_phone

idx_profiles_city
```

Внешние ключи

```text
fk_profiles_users
```

Уникальные ограничения

```text
uq_users_phone
```

Проверки

```text
chk_rating_value
```

---

# 10. Правила хранения файлов

В PostgreSQL не сохраняются бинарные файлы.

Хранятся только ссылки.

Например

```text
avatar_url

video_url

thumbnail_url
```

Физические файлы располагаются:

* MinIO;
* Amazon S3.

---

# 11. Общая стратегия индексации

По умолчанию используется

B-tree.

GIN применяется для:

* полнотекстового поиска;
* JSONB;
* массивов.

GiST применяется для:

* географических координат;
* расстояний.

BRIN применяется для:

* журналов;
* больших временных таблиц.

Hash используется только в исключительных случаях.

---

# 12. Политика транзакций

Все изменения выполняются внутри транзакций.

Запрещается частичная запись данных.

Пример:

Создание пользователя включает:

1. запись пользователя;
2. создание профиля;
3. создание настроек;
4. выдачу роли.

Если любой шаг завершился ошибкой, транзакция полностью откатывается.

---

# 13. Конкурентный доступ

Используется стратегия:

* MVCC PostgreSQL;
* оптимистичная блокировка (`version`);
* выборочное применение `SELECT ... FOR UPDATE` для критичных сценариев (например, платежей и бронирований).

---

# 14. Масштабирование

Архитектура должна поддерживать:

* репликацию;
* read replicas;
* партиционирование;
* резервное копирование;
* горизонтальное масштабирование Backend.

---

# 15. Безопасность

Запрещается хранить:

* пароли;
* SMS-коды;
* JWT-токены;
* секретные ключи;
* персональные файлы.

Пароли заменяются криптографическими хешами.

Временные коды имеют ограниченный срок хранения.

---

# 16. Производительность

При проектировании учитываются:

* минимизация количества JOIN;
* минимизация блокировок;
* отсутствие `SELECT *`;
* использование пакетных операций;
* обязательная индексация часто используемых полей;
* анализ запросов через `EXPLAIN ANALYZE`.

---

# 17. Версионирование схемы

Изменение структуры БД выполняется только через Alembic.

Каждое изменение сопровождается:

* обновлением SQL-проекта;
* обновлением ORM-моделей;
* новой миграцией;
* обновлением документации.

Редактирование уже применённых миграций запрещено.

---

# 18. Принципы разработки

Каждая новая таблица должна:

* относиться к одному агрегату;
* иметь понятное назначение;
* быть документирована;
* иметь первичный ключ;
* иметь внешние ключи;
* иметь индексы для основных сценариев поиска;
* иметь ограничения целостности.

---

# Часть 2. Доменная модель (DDD) и карта агрегатов

> Версия 1.0

---

# 1. Назначение

Настоящий документ определяет предметную область (Domain Model) проекта DanceMate.

Все остальные уровни системы строятся на основании данного документа.

На основе Domain Model разрабатываются:

- PostgreSQL Schema
- SQLAlchemy Models
- Repository
- Use Cases
- REST API
- WebSocket
- Flutter Models
- Unit Tests

Domain является главным источником бизнес-логики.

---

# 2. Общая карта предметной области

Проект состоит из следующих Bounded Context.

```text
DanceMate
│
├── Identity
├── User
├── Profile
├── Dance
├── Availability
├── Matching
├── Invitation
├── Meeting
├── Event
├── Squad
├── Chat
├── Rating
├── Gratitude
├── Notification
├── Media
├── Subscription
├── Payment
├── Moderation
├── Analytics
└── Administration
```

Каждый Context развивается независимо.

---

# 3. Карта зависимостей Context

```text
Identity
      │
      ▼
User
      │
      ▼
Profile
      │
      ├───────────────┐
      ▼               ▼
Dance          Availability
      │               │
      └───────┬───────┘
              ▼
         Matching
              │
      ┌───────┴────────┐
      ▼                ▼
Invitation         Event
      │                │
      ▼                ▼
Meeting          Squad
      │                │
      └───────┬────────┘
              ▼
             Chat
              │
      ┌───────┴────────┐
      ▼                ▼
Rating        Gratitude
              │
              ▼
Notification
```

Зависимости всегда направлены вниз.

Обратные зависимости запрещены.

---

# 4. Aggregate Root

Каждый Context содержит собственный Aggregate Root.

## Identity

Aggregate Root

```text
Account
```

Отвечает за

- регистрацию
- вход
- JWT
- SMS
- устройство

---

## User

Aggregate Root

```text
User
```

Отвечает

- телефон

- состояние

- настройки

- роли

---

## Profile

Aggregate Root

```text
Profile
```

Отвечает

- анкета

- фотографии

- видео

- описание

- параметры танцора

---

## Dance

Aggregate Root

```text
DanceStyle
```

---

## Availability

Aggregate Root

```text
Availability
```

Отвечает

когда пользователь может танцевать.

---

## Matching

Aggregate Root

```text
MatchingSession
```

Отвечает

за алгоритм поиска партнера.

---

## Invitation

Aggregate Root

```text
Invitation
```

---

## Meeting

Aggregate Root

```text
Meeting
```

---

## Event

Aggregate Root

```text
Event
```

---

## Squad

Aggregate Root

```text
Squad
```

---

## Chat

Aggregate Root

```text
Conversation
```

---

## Rating

Aggregate Root

```text
Rating
```

---

## Gratitude

Aggregate Root

```text
Gratitude
```

---

## Subscription

Aggregate Root

```text
Subscription
```

---

## Payment

Aggregate Root

```text
Payment
```

---

## Notification

Aggregate Root

```text
Notification
```

---

## Moderation

Aggregate Root

```text
ModerationCase
```

---

## Analytics

Aggregate Root

```text
Statistics
```

---

# 5. Внутреннее устройство Aggregate

Каждый Aggregate состоит из

```text
Aggregate Root

↓

Entities

↓

Value Objects

↓

Domain Events

↓

Policies

↓

Specifications

↓

Factories

↓

Repository Interface
```

---

# 6. Пример Aggregate

Profile

```text
Profile
│
├── Photos
├── Videos
├── DanceLevels
├── Preferences
├── Visibility
└── Verification
```

Изменить Profile можно только через Aggregate Root.

Нельзя изменять внутренние Entity напрямую.

---

# 7. Entity

Entity имеет собственный жизненный цикл.

Например

```text
Profile

Invitation

Meeting

Message

Notification
```

Entity всегда имеет

```text
UUID
```

---

# 8. Value Objects

Не имеют собственной идентичности.

Например

```text
PhoneNumber

Email

Coordinates

GeoPoint

Money

DanceLevel

AgeRange

TimeInterval

SkillLevel
```

Value Object неизменяем.

---

# 9. Domain Services

Используются только тогда, когда бизнес-логика не принадлежит Entity.

Например

```text
PartnerMatchingService

DistanceCalculationService

SubscriptionService

RecommendationService

RatingCalculationService
```

---

# 10. Domain Events

Каждое важное изменение генерирует событие.

Например

```text
UserRegistered

ProfileCompleted

InvitationCreated

InvitationAccepted

MeetingStarted

MeetingFinished

RatingAdded

SubscriptionActivated

PaymentSucceeded
```

Domain Event не содержит бизнес-логику.

Он сообщает системе о произошедшем событии.

---

# 11. Specifications

Содержат бизнес-предикаты.

Например

```text
CanInvitePartner

CanCreateEvent

CanSendMessage

PremiumRequired

CanRateMeeting
```

Specification возвращает

```text
true

или

false
```

---

# 12. Policies

Policy определяет правила предметной области.

Например

```text
InvitationPolicy

PaymentPolicy

SubscriptionPolicy

ModerationPolicy

RatingPolicy
```

---

# 13. Factory

Используется для создания сложных Aggregate.

Например

```text
ProfileFactory

EventFactory

InvitationFactory

PaymentFactory
```

---

# 14. Repository

Каждый Aggregate имеет собственный Repository.

Например

```text
UserRepository

ProfileRepository

InvitationRepository

MeetingRepository

ChatRepository
```

Repository работает только с Aggregate Root.

---

# 15. Границы Aggregate

Пример

Invitation

может изменять

```text
Invitation
```

но

не может

напрямую изменять

```text
Meeting
```

После принятия приглашения создается

Domain Event

↓

MeetingCreated

↓

Meeting Aggregate

создается самостоятельно.

---

# 16. Владение данными

Каждый Aggregate является единственным владельцем своих данных.

Например

Profile

владеет

```text
Фото

Видео

Описание

Настройки приватности
```

Никакой другой Aggregate не имеет права изменять их напрямую.

---

# 17. Связи Aggregate

Используются только ссылки.

Например

```text
Invitation

↓

UserId

↓

ProfileId
```

Не допускаются циклические ссылки между Aggregate.

---

# 18. Жизненный цикл Aggregate

Пример

Invitation

```text
Created

↓

Pending

↓

Accepted

↓

MeetingCreated

↓

Completed

↓

Archived
```

Каждый переход проверяется бизнес-правилами.

---

# 19. Правила Domain

Domain

никогда

не знает

о

FastAPI

PostgreSQL

Redis

Celery

Docker

SQLAlchemy

HTTP

REST

WebSocket

JSON

JWT

---

# 20. Правила изменения Domain

Любое изменение Aggregate проходит через Use Case.

Пример

```text
AcceptInvitation

↓

Load Invitation Aggregate

↓

Validate

↓

Create Domain Event

↓

Save Aggregate

↓

Publish Event
```

---

# 21. Стратегия масштабирования

Каждый Context может быть выделен в отдельный сервис.

Например

```text
User Service

Profile Service

Chat Service

Payment Service

Notification Service
```

Архитектура не требует изменения Domain при переходе к микросервисам.

---

# 22. Карта будущих таблиц PostgreSQL

Каждый Aggregate соответствует основной таблице:

| Aggregate | Основная таблица |
|------------|------------------|
| Account | accounts |
| User | users |
| Profile | profiles |
| DanceStyle | dance_styles |
| Availability | availability |
| MatchingSession | matching_sessions |
| Invitation | invitations |
| Meeting | meetings |
| Event | events |
| Squad | squads |
| Conversation | conversations |
| Rating | ratings |
| Gratitude | gratitude |
| Notification | notifications |
| Subscription | subscriptions |
| Payment | payments |
| ModerationCase | moderation_cases |
| Statistics | statistics |

---
# Часть 3. Логическая модель данных

Часть 3.1 Master Data Model
Версия 1.0
1. Назначение

Master Data Model (MDM) является главным документом предметной области.

Он определяет:

все сущности проекта;
владельцев данных;
жизненный цикл данных;
связи между сущностями;
границы агрегатов;
будущие таблицы PostgreSQL.

Все остальные документы строятся исключительно на основании MDM.

2. Карта предметной области
DanceMate
│
├── Identity
│
├── User
│
├── Profile
│
├── Matching
│
├── Invitation
│
├── Meeting
│
├── Event
│
├── Squad
│
├── Chat
│
├── Rating
│
├── Gratitude
│
├── Notification
│
├── Subscription
│
├── Payment
│
├── Moderation
│
├── Analytics
│
└── Administration
3. Identity Context

Назначение

Авторизация.

Регистрация.

JWT.

SMS.

Устройства.

Aggregate
Account

Entity

Account

RefreshToken

Device

SmsCode

Session

Будущие таблицы

accounts

account_sessions

account_devices

sms_codes

refresh_tokens
4. User Context

Aggregate

User

Entity

User

Role

Permission

UserSetting

UserBlock

FavoriteCity

Таблицы

users

roles

permissions

role_permissions

user_roles

user_settings

user_blocks

favorite_cities
5. Profile Context

Aggregate

Profile

Entity

Profile

ProfilePhoto

ProfileVideo

DanceSkill

ProfilePreference

SearchPreference

Verification

Achievement

Таблицы

profiles

profile_photos

profile_videos

profile_dance_styles

profile_preferences

search_preferences

verifications

achievements
6. Reference Context

Справочники.

Aggregate отсутствует.

Таблицы

dance_styles

dance_levels

countries

regions

cities

languages

subscription_plans

notification_types

media_types
7. Matching Context

Aggregate

MatchingSession

Entity

MatchingSession

SearchFilter

Recommendation

SearchHistory

Таблицы

matching_sessions

matching_filters

recommendations

search_history
8. Invitation Context

Aggregate

Invitation

Entity

Invitation

InvitationHistory

Таблицы

invitations

invitation_history
9. Meeting Context

Aggregate

Meeting

Entity

Meeting

MeetingParticipant

MeetingResult

Таблицы

meetings

meeting_participants

meeting_results
10. Event Context

Aggregate

Event

Entity

Event

EventMember

EventCategory

EventLocation

EventMedia

Таблицы

events

event_members

event_categories

event_locations

event_media
11. Squad Context

Aggregate

Squad

Entity

Squad

SquadMember

SquadRole

Таблицы

squads

squad_members

squad_roles
12. Chat Context

Aggregate

Conversation

Entity

Conversation

Message

MessageAttachment

MessageReaction

MessageRead

Таблицы

conversations

conversation_members

messages

message_attachments

message_reactions

message_reads
13. Rating Context

Aggregate

Rating

Entity

Rating

RatingCategory

RatingHistory

Таблицы

ratings

rating_categories

rating_history
14. Gratitude Context

Aggregate

Gratitude

Entity

Gratitude

GratitudeTemplate

Таблицы

gratitude

gratitude_templates
15. Notification Context

Aggregate

Notification

Entity

Notification

NotificationQueue

PushDevice

EmailQueue

Таблицы

notifications

notification_queue

push_devices

email_queue
16. Subscription Context

Aggregate

Subscription

Entity

Subscription

SubscriptionHistory

SubscriptionFeature

Таблицы

subscriptions

subscription_history

subscription_features
17. Payment Context

Aggregate

Payment

Entity

Payment

Invoice

Refund

PaymentProvider

Таблицы

payments

invoices

refunds

payment_providers
18. Moderation Context

Aggregate

ModerationCase

Entity

Report

ModerationCase

Evidence

Ban

Таблицы

reports

moderation_cases

evidence

bans
19. Analytics Context

Aggregate

Statistics

Entity

DailyStatistics

MonthlyStatistics

ProfileStatistics

SearchStatistics

Таблицы

daily_statistics

monthly_statistics

profile_statistics

search_statistics
20. Administration Context

Aggregate

Administration

Entity

SystemSetting

AuditLog

BackgroundJob

FeatureFlag

Таблицы

system_settings

audit_logs

background_jobs

feature_flags
21. Итоговая карта таблиц
Контекст	Количество таблиц
Identity	    5
User	        8
Profile 	    8
Reference	    9
Matching	    4
Invitation	    2
Meeting	        3
Event	        5
Squad	        3
Chat	        5
Rating	        3
Gratitude	    2
Notification	4
Subscription	3
Payment	        4
Moderation	    4
Analytics	    4
Administration	4

Итого: около 80–90 таблиц (после добавления связующих таблиц многие связи N:M потребуют отдельных сущностей).

22. Карта зависимостей контекстов
Identity
    │
    ▼
User
    │
    ▼
Profile
    │
 ┌──┴───────────────┐
 ▼                  ▼
Matching         Event
 │                  │
 ▼                  ▼
Invitation      Squad
 │                  │
 └──────┬───────────┘
        ▼
     Meeting
        │
        ▼
       Chat
        │
 ┌──────┴────────────┐
 ▼                   ▼
Rating          Gratitude
        │
        ▼
Notification

Subscription
      │
      ▼
Payment

Moderation

Analytics

Administration

---

# Часть 3.2. Relationship Matrix (Матрица связей между сущностями)

# 1. Назначение

Relationship Matrix определяет:

- все связи между агрегатами;
- владельцев внешних ключей;
- кардинальность;
- обязательность связи;
- каскадное поведение;
- правила удаления;
- правила обновления.

Данный документ является основанием для построения ER-диаграммы PostgreSQL.

---

# 2. Типы связей

Используются четыре типа связей.

| Тип | Обозначение | Пример |
|------|-------------|---------|
| Один к одному | 1:1 | User → Profile |
| Один ко многим | 1:N | Profile → Photos |
| Многие ко многим | N:M | Profile ↔ DanceStyle |
| Агрегатная ссылка | Reference | Invitation → User |

---

# 3. Правила построения

1. Aggregate никогда не содержит другой Aggregate.
2. Связь между Aggregate осуществляется только по UUID.
3. Внешний ключ принадлежит стороне-владельцу.
4. Каскадное удаление используется только для внутренних Entity агрегата.
5. Между агрегатами запрещено использовать `ON DELETE CASCADE`.

---

# 4. Матрица связей агрегатов

| От | К | Тип | FK хранится в | Обязательна |
|----|---|-----|---------------|-------------|
| Account | User | 1:1 | users.account_id | Да |
| User | Profile | 1:1 | profiles.user_id | Да |
| User | Subscription | 1:N | subscriptions.user_id | Нет |
| User | Notification | 1:N | notifications.user_id | Нет |
| User | Device | 1:N | account_devices.account_id | Нет |
| Profile | Invitation | 1:N | invitations.sender_profile_id / receiver_profile_id | Нет |
| Profile | Meeting | N:M | meeting_participants | Нет |
| Profile | Event | N:M | event_members | Нет |
| Profile | Squad | N:M | squad_members | Нет |
| Profile | Conversation | N:M | conversation_members | Нет |
| Profile | Rating | 1:N | ratings.author_profile_id / target_profile_id | Нет |
| Profile | Gratitude | 1:N | gratitude.author_profile_id / target_profile_id | Нет |
| Profile | DanceStyle | N:M | profile_dance_styles | Нет |
| Profile | Photo | 1:N | profile_photos.profile_id | Нет |
| Profile | Video | 1:N | profile_videos.profile_id | Нет |
| Event | EventMember | 1:N | event_members.event_id | Да |
| Event | Media | 1:N | event_media.event_id | Нет |
| Squad | SquadMember | 1:N | squad_members.squad_id | Да |
| Conversation | Message | 1:N | messages.conversation_id | Да |
| Message | Attachment | 1:N | message_attachments.message_id | Нет |
| Message | Reaction | 1:N | message_reactions.message_id | Нет |
| Meeting | Rating | 1:N | ratings.meeting_id | Нет |
| Meeting | Gratitude | 1:N | gratitude.meeting_id | Нет |
| Subscription | Payment | 1:N | payments.subscription_id | Нет |

---

# 5. Внутренние связи агрегатов

## User

```text
User
│
├── Settings
├── Roles
├── Blocks
└── FavoriteCities
```

Все внутренние Entity принадлежат агрегату User.

Удаление User приводит к удалению внутренних сущностей только после процедуры анонимизации или архивирования, если это допускается политикой хранения данных.

---

## Profile

```text
Profile
│
├── Photos
├── Videos
├── Preferences
├── Verification
└── Achievements
```

Все сущности Profile существуют только вместе с Profile.

---

## Event

```text
Event
│
├── Members
└── Media
```

---

## Chat

```text
Conversation
│
├── Members
├── Messages
│
├── Attachments
└── Reactions
```

---

# 6. Связи N:M

Используются исключительно через промежуточные таблицы.

| Сущность A | Сущность B | Таблица связи |
|------------|------------|---------------|
| Profile | DanceStyle | profile_dance_styles |
| Profile | Event | event_members |
| Profile | Squad | squad_members |
| Profile | Meeting | meeting_participants |
| Profile | Conversation | conversation_members |
| Role | Permission | role_permissions |

Прямые связи `N:M` запрещены.

---

# 7. Владельцы данных

| Данные | Владелец |
|----------|----------|
| Телефон | Account |
| Настройки пользователя | User |
| Анкета | Profile |
| Фото | Profile |
| Видео | Profile |
| Приглашение | Invitation |
| Встреча | Meeting |
| Мероприятие | Event |
| Группа | Squad |
| Сообщение | Conversation |
| Оценка | Rating |
| Благодарность | Gratitude |
| Подписка | Subscription |
| Платеж | Payment |
| Жалоба | ModerationCase |
| Уведомление | Notification |

Изменять данные имеет право только их владелец (Aggregate Root).

---

# 8. Правила внешних ключей

Все внешние ключи именуются по шаблону:

```text
fk_<таблица>_<родительская_таблица>
```

Например:

```text
fk_profiles_users

fk_messages_conversations

fk_ratings_meetings
```

---

# 9. Правила удаления

Используются четыре стратегии.

| Тип данных | Стратегия |
|-------------|-----------|
| Справочники | RESTRICT |
| Внутренние Entity агрегата | CASCADE |
| Связи между агрегатами | RESTRICT |
| История и аудит | Запрет удаления (архивирование/Soft Delete) |

---

# 10. Правила обновления

Для всех внешних ключей применяется:

```sql
ON UPDATE CASCADE
```

Это гарантирует целостность при изменении ключей (хотя UUID на практике не изменяются).

---

# 11. Правила обязательности

Связи делятся на:

### Обязательные

```text
User → Profile

Account → User

Conversation → Message

Event → EventMember
```

### Необязательные

```text
Profile → Video

Profile → Achievement

Subscription → Payment

Meeting → Rating
```

---

# 12. Циклические зависимости

Циклические ссылки запрещены.

Правильная схема:

```text
Profile

↓

Invitation

↓

Meeting

↓

Rating
```

Неправильно:

```text
Profile

↓

Invitation

↓

Meeting

↓

Profile
```

Связь обратно к `Profile` осуществляется только через внешний ключ без формирования циклического агрегата.

---

# 13. Подготовка к ER-диаграмме

После построения Relationship Matrix каждая связь будет отражена в ER-диаграмме с указанием:

- кардинальности;
- обязательности;
- владельца внешнего ключа;
- поведения при удалении;
- поведения при обновлении.

---

# Часть 3.3. Data Catalog (Каталог таблиц PostgreSQL)

---

# 1. Назначение

Data Catalog представляет собой полный реестр всех таблиц базы данных проекта.

Документ определяет:

* назначение каждой таблицы;
* принадлежность к Domain Context;
* Aggregate Owner;
* назначение данных;
* примерный объём данных;
* требования к индексам;
* необходимость Soft Delete;
* необходимость аудита;
* необходимость партиционирования.

Данный документ является основой для дальнейшего описания структуры всех таблиц.

---

# 2. Классификация таблиц

Все таблицы делятся на пять категорий.

| Категория | Назначение                |
| --------- | ------------------------- |
| Core      | Основные системные данные |
| Business  | Основные бизнес-сущности  |
| Reference | Справочники               |
| Service   | Служебные таблицы         |
| Analytics | Аналитика и статистика    |

---

# 3. Identity Context

| Таблица          | Тип     | Aggregate | Soft Delete | Audit | Partition |
| ---------------- | ------- | --------- | ----------- | ----- | --------- |
| accounts         | Core    | Account   | Нет         | Да    | Нет       |
| account_sessions | Service | Account   | Нет         | Нет   | Да        |
| account_devices  | Service | Account   | Нет         | Нет   | Нет       |
| refresh_tokens   | Service | Account   | Нет         | Нет   | Да        |
| sms_codes        | Service | Account   | Нет         | Нет   | Да        |

---

# 4. User Context

| Таблица          | Тип       | Aggregate | Soft Delete | Audit | Partition |
| ---------------- | --------- | --------- | ----------- | ----- | --------- |
| users            | Core      | User      | Да          | Да    | Нет       |
| user_settings    | Business  | User      | Нет         | Да    | Нет       |
| user_roles       | Business  | User      | Нет         | Да    | Нет       |
| roles            | Reference | —         | Нет         | Нет   | Нет       |
| permissions      | Reference | —         | Нет         | Нет   | Нет       |
| role_permissions | Reference | —         | Нет         | Нет   | Нет       |
| user_blocks      | Business  | User      | Да          | Да    | Нет       |
| favorite_cities  | Business  | User      | Нет         | Нет   | Нет       |

---

# 5. Profile Context

| Таблица              | Тип      | Aggregate | Soft Delete | Audit | Partition |
| -------------------- | -------- | --------- | ----------- | ----- | --------- |
| profiles             | Business | Profile   | Да          | Да    | Нет       |
| profile_photos       | Business | Profile   | Да          | Нет   | Нет       |
| profile_videos       | Business | Profile   | Да          | Нет   | Нет       |
| profile_dance_styles | Business | Profile   | Нет         | Нет   | Нет       |
| profile_preferences  | Business | Profile   | Нет         | Да    | Нет       |
| search_preferences   | Business | Profile   | Нет         | Да    | Нет       |
| verifications        | Business | Profile   | Да          | Да    | Нет       |
| achievements         | Business | Profile   | Нет         | Да    | Нет       |

---

# 6. Reference Context

| Таблица            | Тип       |
| ------------------ | --------- |
| dance_styles       | Reference |
| dance_levels       | Reference |
| countries          | Reference |
| regions            | Reference |
| cities             | Reference |
| languages          | Reference |
| subscription_plans | Reference |
| notification_types | Reference |
| media_types        | Reference |

Все справочники являются неизменяемыми или редко изменяемыми.

Soft Delete не используется.

---

# 7. Matching Context

| Таблица           | Тип       | Aggregate       |
| ----------------- | --------- | --------------- |
| matching_sessions | Business  | MatchingSession |
| matching_filters  | Business  | MatchingSession |
| recommendations   | Business  | MatchingSession |
| search_history    | Analytics | MatchingSession |

---

# 8. Invitation Context

| Таблица            | Тип       | Aggregate  |
| ------------------ | --------- | ---------- |
| invitations        | Business  | Invitation |
| invitation_history | Analytics | Invitation |

---

# 9. Meeting Context

| Таблица              | Тип      | Aggregate |
| -------------------- | -------- | --------- |
| meetings             | Business | Meeting   |
| meeting_participants | Business | Meeting   |
| meeting_results      | Business | Meeting   |

---

# 10. Event Context

| Таблица          | Тип       | Aggregate |
| ---------------- | --------- | --------- |
| events           | Business  | Event     |
| event_members    | Business  | Event     |
| event_media      | Business  | Event     |
| event_categories | Reference | —         |
| event_locations  | Business  | Event     |

---

# 11. Squad Context

| Таблица       | Тип       | Aggregate |
| ------------- | --------- | --------- |
| squads        | Business  | Squad     |
| squad_members | Business  | Squad     |
| squad_roles   | Reference | —         |

---

# 12. Chat Context

| Таблица              | Тип      | Aggregate    |
| -------------------- | -------- | ------------ |
| conversations        | Business | Conversation |
| conversation_members | Business | Conversation |
| messages             | Business | Conversation |
| message_attachments  | Business | Conversation |
| message_reactions    | Business | Conversation |
| message_reads        | Business | Conversation |

---

# 13. Rating Context

| Таблица           | Тип       | Aggregate |
| ----------------- | --------- | --------- |
| ratings           | Business  | Rating    |
| rating_categories | Reference | —         |
| rating_history    | Analytics | Rating    |

---

# 14. Gratitude Context

| Таблица             | Тип       | Aggregate |
| ------------------- | --------- | --------- |
| gratitude           | Business  | Gratitude |
| gratitude_templates | Reference | —         |

---

# 15. Notification Context

| Таблица            | Тип      | Aggregate    |
| ------------------ | -------- | ------------ |
| notifications      | Business | Notification |
| notification_queue | Service  | Notification |
| push_devices       | Service  | Notification |
| email_queue        | Service  | Notification |

---

# 16. Subscription Context

| Таблица               | Тип       | Aggregate    |
| --------------------- | --------- | ------------ |
| subscriptions         | Business  | Subscription |
| subscription_history  | Analytics | Subscription |
| subscription_features | Reference | —            |

---

# 17. Payment Context

| Таблица           | Тип       | Aggregate |
| ----------------- | --------- | --------- |
| payments          | Business  | Payment   |
| invoices          | Business  | Payment   |
| refunds           | Business  | Payment   |
| payment_providers | Reference | —         |

---

# 18. Moderation Context

| Таблица          | Тип      | Aggregate      |
| ---------------- | -------- | -------------- |
| reports          | Business | ModerationCase |
| moderation_cases | Business | ModerationCase |
| evidence         | Business | ModerationCase |
| bans             | Business | ModerationCase |

---

# 19. Analytics Context

| Таблица            | Тип       |
| ------------------ | --------- |
| daily_statistics   | Analytics |
| monthly_statistics | Analytics |
| profile_statistics | Analytics |
| search_statistics  | Analytics |

---

# 20. Administration Context

| Таблица         | Тип     |
| --------------- | ------- |
| system_settings | Core    |
| audit_logs      | Service |
| background_jobs | Service |
| feature_flags   | Core    |

---

# 21. Итоговая статистика

| Категория | Количество |
| --------- | ---------: |
| Core      |          4 |
| Business  |        30+ |
| Reference |        15+ |
| Service   |        10+ |
| Analytics |        10+ |

Общее количество таблиц после реализации всех промежуточных сущностей составит около **75–90**.

---

# 22. Жизненный цикл данных

Все таблицы делятся на три группы:

### Постоянные

* users
* profiles
* events
* subscriptions

### Временные

* sms_codes
* refresh_tokens
* notification_queue
* background_jobs

### Архивные

* invitation_history
* rating_history
* monthly_statistics

Для архивных таблиц допускается перенос данных в отдельные секции хранения.

---

# 23. Требования к производительности

Каждая таблица должна иметь:

* первичный ключ (`UUID`);
* обязательные внешние ключи;
* индексы для основных сценариев поиска;
* ограничения (`CHECK`, `UNIQUE`, `FOREIGN KEY`);
* комментарии (`COMMENT ON TABLE`, `COMMENT ON COLUMN`) для автоматической генерации документации.

---

# Часть 3.4. Enterprise Data Dictionary

---

# 1. Назначение

Enterprise Data Dictionary определяет полный стандарт описания всех таблиц PostgreSQL.

Каждая таблица описывается одинаковым образом.

Это позволяет автоматически генерировать:

* SQL Schema;
* SQLAlchemy Models;
* Alembic Migration;
* Repository;
* OpenAPI;
* внутреннюю документацию.

---

# 2. Общий шаблон описания таблицы

Для каждой таблицы используется следующая структура.

```text
Название

Назначение

Domain Context

Aggregate

Владелец данных

Описание

Первичный ключ

Список столбцов

Внешние ключи

Ограничения

Индексы

Триггеры

Soft Delete

Audit

Пример записи

Комментарии
```

---

# 3. Корпоративный стандарт столбцов

Практически каждая бизнес-таблица содержит единый набор технических полей.

| Поле       | Тип PostgreSQL | Назначение               |
| ---------- | -------------- | ------------------------ |
| id         | UUID           | Первичный ключ           |
| created_at | TIMESTAMPTZ    | Дата создания            |
| updated_at | TIMESTAMPTZ    | Последнее изменение      |
| created_by | UUID           | Автор создания           |
| updated_by | UUID           | Последний редактор       |
| deleted_at | TIMESTAMPTZ    | Soft Delete              |
| version    | INTEGER        | Оптимистичная блокировка |

Служебные и справочные таблицы могут использовать сокращённый набор полей.

---

# 4. Стандарт именования

Все таблицы используют:

```text
snake_case
```

Все столбцы используют:

```text
snake_case
```

Внешние ключи

```text
user_id

profile_id

meeting_id
```

Булевые поля

```text
is_active

is_verified

is_public

has_subscription
```

Дата

```text
created_at

updated_at

expires_at

deleted_at
```

---

# 5. Стандарт типов PostgreSQL

| Тип данных  | Использование                                      |
| ----------- | -------------------------------------------------- |
| UUID        | Идентификаторы                                     |
| VARCHAR     | Короткий текст                                     |
| TEXT        | Длинный текст                                      |
| BOOLEAN     | Флаги                                              |
| SMALLINT    | Небольшие числовые значения                        |
| INTEGER     | Счётчики                                           |
| BIGINT      | Большие значения                                   |
| NUMERIC     | Денежные суммы                                     |
| TIMESTAMPTZ | Дата и время                                       |
| DATE        | Дата                                               |
| JSONB       | Дополнительные параметры                           |
| INET        | IP-адрес                                           |
| POINT       | Географические координаты                          |
| BYTEA       | Только при отсутствии внешнего файлового хранилища |

---

# 6. Стандарт ограничений

Каждая таблица должна иметь:

* PRIMARY KEY;
* FOREIGN KEY (при наличии связей);
* CHECK;
* UNIQUE (при необходимости);
* NOT NULL для обязательных данных.

---

# 7. Стандарт индексов

Каждая таблица имеет:

* Primary Index;
* индексы поиска;
* индексы сортировки;
* составные индексы для часто используемых запросов.

Например:

```sql
idx_profiles_city

idx_profiles_gender

idx_events_date

idx_messages_created_at
```

---

# 8. Стандарт комментариев

Каждая таблица документируется средствами PostgreSQL.

Пример:

```sql
COMMENT ON TABLE profiles IS 'Профили пользователей';

COMMENT ON COLUMN profiles.birth_date IS 'Дата рождения пользователя';
```

Комментарии используются для автоматической генерации технической документации.

---

# 9. Правила Soft Delete

Используется для:

* пользователей;
* профилей;
* событий;
* приглашений;
* сообщений;
* подписок.

Не используется для:

* справочников;
* журналов аудита;
* промежуточных таблиц связей;
* временных данных.

---

# 10. Правила аудита

Аудит обязателен для:

* регистрации;
* изменения профиля;
* смены ролей;
* оплаты;
* подписок;
* модерации;
* изменения системных настроек.

Остальные таблицы могут использовать выборочный аудит.

---

# 11. Стратегия хранения файлов

База данных хранит только ссылки.

Например:

```text
avatar_url

video_url

thumbnail_url
```

Файлы располагаются в объектном хранилище (MinIO или Amazon S3).

---

# 12. Категории таблиц

Таблицы делятся на следующие группы:

### Core

* accounts
* users

### Business

* profiles
* invitations
* meetings
* events
* messages

### Reference

* dance_styles
* countries
* cities

### Service

* notifications
* refresh_tokens
* background_jobs

### Analytics

* daily_statistics
* search_statistics

---

# 13. Правила доступа

Каждая таблица имеет уровень доступа.

| Уровень       | Описание                           |
| ------------- | ---------------------------------- |
| Public        | Доступно всем                      |
| Authorized    | Только авторизованные пользователи |
| Owner         | Только владелец данных             |
| Moderator     | Модераторы                         |
| Administrator | Администраторы                     |
| System        | Только внутренние сервисы          |

---

# 14. Стратегия кэширования

Кэшируются:

* справочники;
* популярные события;
* танцевальные стили;
* публичные профили;
* результаты поиска.

Не кэшируются:

* платежи;
* роли;
* права;
* модерация.

---

# 15. Правила партиционирования

Партиционирование применяется только для быстрорастущих таблиц:

* messages;
* notifications;
* audit_logs;
* daily_statistics;
* search_history.

Остальные таблицы используют стандартную структуру PostgreSQL.

---

# 16. Стратегия роста

Проект рассчитан на:

* более 1 000 000 пользователей;
* десятки миллионов сообщений;
* миллионы приглашений;
* сотни миллионов записей аналитики.

Структура базы данных должна поддерживать дальнейшее горизонтальное масштабирование без изменения логической модели.

---

