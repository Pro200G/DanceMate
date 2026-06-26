# 07_API.md

# API

**Проект:** DanceMate
**Версия:** 1.0

---

# 1. Назначение

API Layer является внешним интерфейсом системы.

Он отвечает за взаимодействие с клиентскими приложениями:

* Web;
* Android;
* iOS;
* Desktop;
* Admin Panel;
* внешними сервисами.

API Layer принимает HTTP и WebSocket запросы, преобразует их в DTO, передаёт их в Application Layer и возвращает результат клиенту.

API Layer **не содержит бизнес-логики**.

---

# 2. Архитектура слоя

```text
Client
    │
    ▼
FastAPI Router
    │
    ▼
Dependency Injection
    │
    ▼
DTO Validation
    │
    ▼
Use Case
    │
    ▼
Response DTO
    │
    ▼
JSON
```

---

# 3. Структура каталога

```text
app/
└── api/
    │
    ├── v1/
    │   ├── auth.py
    │   ├── users.py
    │   ├── profiles.py
    │   ├── dance_styles.py
    │   ├── invitations.py
    │   ├── meetings.py
    │   ├── events.py
    │   ├── squads.py
    │   ├── chats.py
    │   ├── messages.py
    │   ├── ratings.py
    │   ├── gratitude.py
    │   ├── notifications.py
    │   ├── media.py
    │   ├── subscriptions.py
    │   ├── payments.py
    │   ├── moderation.py
    │   ├── admin.py
    │   └── health.py
    │
    ├── websocket/
    │   ├── chat.py
    │   ├── notifications.py
    │   ├── events.py
    │   └── presence.py
    │
    ├── schemas/
    │
    ├── dependencies/
    │
    ├── middleware/
    │
    ├── security/
    │
    ├── errors/
    │
    ├── responses/
    │
    └── versioning/
```

---

# 4. Версионирование API

Все REST API размещаются внутри версии.

```
/api/v1/
```

При несовместимых изменениях создаётся новая версия.

```
/api/v2/
```

Старые версии поддерживаются до окончания периода совместимости.

---

# 5. REST API

Все ресурсы используют REST.

Пример:

```
GET     /profiles

GET     /profiles/{id}

POST    /profiles

PATCH   /profiles/{id}

DELETE  /profiles/{id}
```

---

# 6. Основные группы маршрутов

## Authentication

```
/auth
```

* регистрация;
* вход;
* выход;
* refresh token;
* OTP;
* смена пароля.

---

## Users

```
/users
```

* управление пользователями;
* настройки;
* роли.

---

## Profiles

```
/profiles
```

* анкеты;
* поиск;
* фильтрация;
* фотографии.

---

## Dance Styles

```
/dance-styles
```

Справочник танцевальных направлений.

---

## Invitations

```
/invitations
```

Приглашения на танцы.

---

## Meetings

```
/meetings
```

Индивидуальные встречи.

---

## Events

```
/events
```

Фестивали, вечеринки, мастер-классы.

---

## Squads

```
/squads
```

Групповые поездки.

---

## Chats

```
/chats
```

Диалоги.

---

## Messages

```
/messages
```

Сообщения.

---

## Ratings

```
/ratings
```

Оценки пользователей.

---

## Gratitude

```
/gratitude
```

Благодарности.

---

## Notifications

```
/notifications
```

Получение уведомлений.

---

## Media

```
/media
```

Фотографии и видео.

---

## Payments

```
/payments
```

Оплата подписок.

---

## Subscriptions

```
/subscriptions
```

Тарифы.

---

## Moderation

```
/moderation
```

Жалобы.

---

## Admin

```
/admin
```

Административный API.

---

## Health

```
/health
```

Проверка состояния сервиса.

---

# 7. WebSocket API

Используется для:

* чатов;
* уведомлений;
* статуса пользователей;
* обновления событий в реальном времени.

Маршруты:

```
/ws/chat

/ws/notifications

/ws/events

/ws/presence
```

---

# 8. DTO

API использует только DTO.

Типы:

* Create
* Update
* Response
* Search
* Filter
* Pagination

Передача ORM-моделей напрямую запрещена.

---

# 9. Валидация

Все входные данные проверяются средствами Pydantic.

Проверяются:

* типы;
* диапазоны;
* обязательные поля;
* форматы;
* ограничения длины.

---

# 10. Dependency Injection

Все зависимости внедряются через FastAPI Depends.

Примеры:

* текущий пользователь;
* Unit of Work;
* сервис авторизации;
* проверка разрешений.

---

# 11. Middleware

Используются следующие middleware:

* CORS;
* Request ID;
* Correlation ID;
* Logging;
* Rate Limiting;
* Authentication;
* Compression (GZip/Brotli);
* Trusted Hosts;
* HTTPS Redirect.

---

# 12. Безопасность

API поддерживает:

* JWT Access Token;
* Refresh Token;
* OTP;
* RBAC;
* OAuth2 (при необходимости).

Все защищённые маршруты требуют авторизации.

---

# 13. Формат ответа

Все успешные ответы используют единый формат.

```json
{
  "success": true,
  "data": {},
  "meta": {}
}
```

---

# 14. Формат ошибки

Все ошибки возвращаются в едином формате.

```json
{
  "success": false,
  "error": {
    "code": "PROFILE_NOT_FOUND",
    "message": "Профиль не найден"
  }
}
```

---

# 15. Документация API

Автоматически генерируются:

* OpenAPI 3.1;
* Swagger UI;
* ReDoc.

---

# 16. Ограничение запросов

Используется Rate Limiting.

Примеры:

* авторизация;
* регистрация;
* отправка OTP;
* создание сообщений.

---

# 17. Логирование

Логируются:

* HTTP-запросы;
* ошибки;
* время выполнения;
* идентификатор пользователя;
* Correlation ID.

---

# 18. Тестирование

API покрывается:

* Unit Tests;
* Integration Tests;
* Contract Tests;
* Security Tests;
* Load Tests.

---

# 19. Правила разработки

Запрещается:

* использовать SQLAlchemy;
* выполнять SQL-запросы;
* размещать бизнес-логику;
* обращаться напрямую к PostgreSQL;
* работать с Redis без Application Layer.

API Layer выполняет только роль адаптера между клиентом и прикладной логикой.

