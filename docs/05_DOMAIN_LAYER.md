# 05_DOMAIN_LAYER.md

# Domain Layer (DDD Core)

**Проект:** DanceMate
**Версия:** 1.0
**Стиль:** Domain-Driven Design (Strict)
**Язык спецификации:** Python-like pseudo-code + архитектурные правила

---

# 1. Назначение

Domain Layer — это **ядро системы**, полностью независимое от:

* FastAPI
* SQLAlchemy
* Redis
* PostgreSQL
* внешних API

Он содержит только:

* бизнес-логику;
* правила предметной области;
* инварианты;
* доменные события;
* агрегаты.

---

# 2. Главный принцип

```text id="domain_rule"
DOMAIN НЕ ЗНАЕТ НИЧЕГО О ВНЕШНЕМ МИРЕ
```

---

# 3. Структура Domain Layer

```text id="domain_tree"
domain/
│
├── entities/
├── value_objects/
├── aggregates/
├── services/
├── events/
├── exceptions/
├── repositories/   # ONLY INTERFACES
└── rules/
```

---

# 4. Base Domain Model

## 4.1 Entity

```python id="entity_base"
from dataclasses import dataclass
from typing import Any
from uuid import UUID, uuid4

@dataclass
class Entity:
    id: UUID = uuid4()

    def __eq__(self, other: Any) -> bool:
        return isinstance(other, Entity) and self.id == other.id
```

---

## 4.2 Aggregate Root

```python id="aggregate_base"
class AggregateRoot(Entity):

    def __init__(self):
        self._events: list = []

    def add_event(self, event):
        self._events.append(event)

    def pull_events(self):
        events = self._events[:]
        self._events.clear()
        return events
```

---

## 4.3 Value Object

```python id="vo_base"
from dataclasses import dataclass

@dataclass(frozen=True)
class ValueObject:
    pass
```

---

# 5. Domain Entities

---

## 5.1 User

```python id="user_entity"
class User(Entity):

    def __init__(self, email: str, phone: str):
        self.email = email
        self.phone = phone
        self.is_active = True

    def deactivate(self):
        self.is_active = False
```

---

## 5.2 Profile

```python id="profile_entity"
class Profile(Entity):

    def __init__(self, user_id, name, age):
        self.user_id = user_id
        self.name = name
        self.age = age
        self.rating = 0.0

    def update_rating(self, value: float):
        self.rating = value
```

---

## 5.3 Event

```python id="event_entity"
class Event(AggregateRoot):

    def __init__(self, creator_id, title):
        super().__init__()
        self.creator_id = creator_id
        self.title = title
        self.members: list = []

    def add_member(self, user_id):
        if user_id in self.members:
            return

        self.members.append(user_id)
```

---

## 5.4 Chat

```python id="chat_entity"
class Chat(AggregateRoot):

    def __init__(self, participants: list):
        super().__init__()
        self.participants = participants
        self.messages = []

    def send_message(self, sender_id, text: str):
        message = {
            "sender_id": sender_id,
            "text": text
        }
        self.messages.append(message)
```

---

## 5.5 Rating

```python id="rating_entity"
class Rating(Entity):

    def __init__(self, user_id, value: int):
        self.user_id = user_id
        self.value = value

    def validate(self):
        if not (1 <= self.value <= 5):
            raise ValueError("Rating must be 1..5")
```

---

# 6. Value Objects

---

## 6.1 Email

```python id="email_vo"
import re

class Email(ValueObject):

    def __init__(self, value: str):
        if not re.match(r"[^@]+@[^@]+\.[^@]+", value):
            raise ValueError("Invalid email")

        self.value = value
```

---

## 6.2 Phone

```python id="phone_vo"
class Phone(ValueObject):

    def __init__(self, value: str):
        if len(value) < 10:
            raise ValueError("Invalid phone")

        self.value = value
```

---

## 6.3 Location

```python id="location_vo"
class Location(ValueObject):

    def __init__(self, city: str, country: str):
        self.city = city
        self.country = country
```

---

# 7. Domain Services

---

## 7.1 RatingService

```python id="rating_service"
class RatingService:

    def calculate(self, ratings: list[int]) -> float:
        if not ratings:
            return 0.0

        return sum(ratings) / len(ratings)
```

---

## 7.2 MatchingService

```python id="matching_service"
class MatchingService:

    def match(self, user_a, user_b) -> float:
        score = 0

        if user_a.city == user_b.city:
            score += 50

        if user_a.dance_style == user_b.dance_style:
            score += 50

        return score
```

---

# 8. Domain Events

```python id="domain_event"
class DomainEvent:
    def __init__(self):
        self.timestamp = None
```

---

## 8.1 UserCreatedEvent

```python id="user_created_event"
class UserCreatedEvent(DomainEvent):

    def __init__(self, user_id):
        super().__init__()
        self.user_id = user_id
```

---

## 8.2 EventCreatedEvent

```python id="event_created_event"
class EventCreatedEvent(DomainEvent):

    def __init__(self, event_id):
        super().__init__()
        self.event_id = event_id
```

---

# 9. Repository Interfaces (IMPORTANT)

```python id="repo_interface"
from abc import ABC, abstractmethod

class UserRepository(ABC):

    @abstractmethod
    def get_by_id(self, user_id): pass

    @abstractmethod
    def save(self, user): pass
```

---

## ❗ КРИТИЧНО

Domain:

* НЕ знает SQLAlchemy
* НЕ знает PostgreSQL
* НЕ знает Redis

---

# 10. Domain Rules

---

## 10.1 Business Rules

```text id="rules"
- Пользователь не может оценить себя
- Рейтинг всегда 1..5
- Участник не может быть добавлен дважды
- Чат должен иметь минимум 2 участников
```

---

## 10.2 Validation Rules

* Value Objects валидируют данные
* Entities защищают инварианты
* Services не хранят состояние

---

# 11. Domain Exceptions

```python id="exceptions"
class DomainException(Exception):
    pass


class ValidationException(DomainException):
    pass
```

---

# 12. Domain Event Flow

```text id="event_flow"
Entity → Domain Event → Application Layer → Infrastructure (Queue/Bus)
```

---

# 13. Связь с другими слоями

Domain Layer:

```text id="deps"
НЕ ЗАВИСИТ НИ ОТ ЧЕГО
```

Но используется:

* Application Layer
* Infrastructure Layer (через интерфейсы)

---

# 14. Анти-паттерны (ЗАПРЕЩЕНО)

❌ нельзя:

* SQL в домене
* ORM модели в домене
* FastAPI зависимости
* Redis вызовы
* HTTP запросы
* бизнес-логика в API

---

# 15. Расширение системы

Domain легко расширяется:

* новые Entities
* новые Value Objects
* новые Domain Services
* новые Events

---

# 16. Связь с архитектурой

* `02_SYSTEM_ARCHITECTURE.md`
* `03_DATABASE_STRUCTURE.md`
* `04_PHYSICAL_DATA_MODEL.md`
* `06_APPLICATION_LAYER.md`

---

# 17. Итог

Domain Layer — это:

> ❗ единственный источник бизнес-истины в системе

Он должен быть:

* чистым
* изолированным
* тестируемым
* независимым от технологий
